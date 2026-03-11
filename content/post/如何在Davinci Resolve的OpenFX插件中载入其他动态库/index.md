---
title: "如何在Davinci Resolve的OpenFX插件中载入其他动态库"
date: 2026-03-11T13:58:22+08:00
draft: false
tags:
  - Davinci Resolve
  - OpenFX
  - C++
  - C
  - Windows
  - macOS
categories: 其他计算机科学
mathjax: true
markup: goldmark
---

# 前言

首先要明白，OpenFX的官网是[https://openeffects.org/](https://openeffects.org/)，而非[http://www.openfx.org/](http://www.openfx.org/)。

另外，虽然达芬奇的版本区别文件（[https://documents.blackmagicdesign.com/SupportNotes/DaVinci_Resolve_Studio_Features.pdf](https://documents.blackmagicdesign.com/SupportNotes/DaVinci_Resolve_Studio_Features.pdf)）中指出，免费版不支持OpenFX，但实际上目前达芬奇20免费版支持了OpenFX在Fusion和调色界面的使用，我进行了简单的尝试，功能一切正常，而且可以导出视频。

但也不是完全没有问题，达芬奇自己提供的一些闭源OpenFX插件并不可以使用，会有完全破坏视频内容的水印。这些水印是在OpenFX中实现的，而不是在达芬奇软件内。等价于说，任意第三方开发的OpenFX，只要开发者不弄水印，没人管得了你是在免费版还是studio版中使用。

（题外话，目前我尝试出了一种显示版本限制的情况。自己开发的插件，拖到fusion界面中使用，关掉davinci，把这个插件弄坏，再打开davinci。此时davinci就会同时显示插件损坏，以及要求你购买高级版。）

达芬奇软件自己提供了几个openfx示例，windows版在目录`C:\ProgramData\Blackmagic Design\DaVinci Resolve\Support\Developer\OpenFX`下。当然，也提供了Fusion Fuse等插件的示例。就我的看法，OpenFX比Fuse要完备一些。

这几个OpenFX插件最终的编译产物都是一个单独的.ofx文件。从makefile来看，这个.ofx本质上是一个动态链接库，只是改了后缀名（.dll、.dylib、.so）。当然，这几个插件很小，根本不需要外部依赖。我们自己开发插件时可能就需要第三方库，比如引入深度学习算法需要torch，数值优化需要dlib，传统机器视觉算法需要opencv，等等。有时候，这些三方库会提供静态链接文件，只需要把他链接进.ofx文件即可。但很多时候，因为LGPL的限制，或者根本没有静态库，我们就只能使用动态库了。

# Windows上的处理

在Windows上，动态库的路径查找大概只有两个规则：

- 查找exe文件的相同目录
- 查找环境变量中的所有目录

对应到达芬奇的情况下，除了环境变量，就只会查找"Resolve.exe"这个文件所在的目录，例如`G:\Program Files\Blackmagic Design\DaVinci Resolve\Resolve.exe`。

然而，OpenFX标准要求我们，要放到一个特定的OFX目录中，例如：`C:\Program Files\Common Files\OFX\Plugins\TemporalBlurPlugin.ofx.bundle\Contents\Win64\TemporalBlurPlugin.ofx`，这个目录并不是Resolve.exe所在的目录。达芬奇是如何访问这个无法自动查找的.ofx呢？事实上，达芬奇会遍历这整一个OFX文件夹，然后使用类似于`LoadLibrary`的winapi，进行动态库的延迟加载。

如果我们只有一个.ofx，那么一切就都没有问题。但是，如果我们的a.ofx，依赖某个b.dll，该怎么办？直觉上来说，我们可能会放到`TemporalBlurPlugin.ofx.bundle\Contents\Win64\`这个文件夹下，然而这一目录并不在`Resolve.exe`的同目录下，所以并不会被自动加载。与此同时，达芬奇也并不会进行任何操作，将其他依赖通过`LoadLibrary`加载。

那么，还有两种可能。

- 把dll直接放到`Resolve.exe`目录下
- 把dll目录加入环境变量

这两个方法非常直接，但完全不适合正式使用，只适合调试用。首先来说第一条，他有这么几个问题：

1. 其他openfx宿主无法加载。如果要让vegas、natron等软件都加载，就要复制好几次。
2. 无法处理同名dll的问题。达芬奇的目录下也有几十个dll，如果你依赖了相同名字的dll，就几乎不可能完成这件事。你既不知道达芬奇所用的库版本，也不知道编译器是什么，强行替换只会导致问题。
3. 不利于分发。你很难让用户把你的几个dll准确的放到那个目录，删除也不方便删除。

那么第二条怎么样？他也有几个问题：

1. 环境变量污染。可能别的软件就会因为你设置了这个环境变量，而载入错误的库，从而崩溃。
2. 仍然是分发问题。达芬奇用户都是影视方面的，谁懂你环境变量是什么。

那么，有没有一种好办法，能够让我们把所有dll放在.ofx的相同目录，又能够成功加载？

好消息是，winapi为我们提供了`AddDllDirectory`这个函数，它可以让我们自由的添加dll的搜索目录。虽然达芬奇没有使用这个函数来添加.ofx目录，但我们可以自己来。我们可以弄一个代理的空壳插件，在插件中调用`AddDllDirectory`，并转发所有外部函数调用到真正的插件文件上。

根据官方文档所说（[https://openfx.readthedocs.io/en/main/Guide/ofxExample1_Basics.html#life-cycle-of-a-plugin](https://openfx.readthedocs.io/en/main/Guide/ofxExample1_Basics.html#life-cycle-of-a-plugin)），ofx插件只需要对外提供两个接口即可：

- OfxGetNumberOfPlugins
- OfxGetPlugin

所以我们只用转发这两个函数，C代码如下

```c
#include <windows.h>
#include <shlwapi.h>
#include <wchar.h>

#ifndef MAX_PATH
#define MAX_PATH 260
#endif

typedef void OfxPlugin;

typedef OfxPlugin* (__cdecl *PFN_OfxGetPlugin)(int);
typedef int (__cdecl *PFN_OfxGetNumberOfPlugins)(void);

static HMODULE g_hModule = NULL;    // 代理ofx文件
static HMODULE g_real = NULL;       // 插件的真正实现dll文件
static PFN_OfxGetPlugin g_real_GetPlugin = NULL;
static PFN_OfxGetNumberOfPlugins g_real_GetNumber = NULL;
static CRITICAL_SECTION g_cs;
static volatile LONG g_initialized = 0; /* 0 = not init, 1 = init */

static void build_real_dll_path(WCHAR *out, size_t outChars, const WCHAR *realName) {
    // 从代理的.ofx文件，转到真正的插件实现dll

    WCHAR dir[MAX_PATH];
    dir[0] = L'\0';
    GetModuleFileNameW(g_hModule, dir, MAX_PATH);
    PathRemoveFileSpecW(dir);
    PathCombineW(out, dir, realName);
}

static BOOL ensure_real_loaded(void) {
    // 线程安全地加载插件的实现dll

    if (InterlockedCompareExchange(&g_initialized, 1, 1) == 1 && g_real != NULL) {
        return TRUE;
    }

    EnterCriticalSection(&g_cs);
    if (g_real != NULL) {
        LeaveCriticalSection(&g_cs);
        InterlockedExchange(&g_initialized, 1);
        return TRUE;
    }

    WCHAR dllPath[MAX_PATH];
    dllPath[0] = L'\0';
    build_real_dll_path(dllPath, MAX_PATH, L"libSomePluginImpl.dll");

    g_real = LoadLibraryExW(dllPath, NULL, LOAD_WITH_ALTERED_SEARCH_PATH);

    if (!g_real) {
        // fallback: 插件目录加入到搜索目录，非常老的系统中可能会失败
        WCHAR dir[MAX_PATH];
        dir[0] = L'\0';
        GetModuleFileNameW(g_hModule, dir, MAX_PATH);
        PathRemoveFileSpecW(dir);

        SetDefaultDllDirectories(LOAD_LIBRARY_SEARCH_DEFAULT_DIRS | LOAD_LIBRARY_SEARCH_USER_DIRS);
        AddDllDirectory(dir);

        g_real = LoadLibraryW(dllPath);
    }

    if (g_real) {
        FARPROC p1 = GetProcAddress(g_real, "OfxGetPlugin");
        FARPROC p2 = GetProcAddress(g_real, "OfxGetNumberOfPlugins");
        if (p1) g_real_GetPlugin = (PFN_OfxGetPlugin)p1;
        if (p2) g_real_GetNumber = (PFN_OfxGetNumberOfPlugins)p2;
    }

    LeaveCriticalSection(&g_cs);

    InterlockedExchange(&g_initialized, (g_real != NULL) ? 1 : 2);
    return (g_real != NULL);
}

#ifdef __cplusplus
extern "C" {
#endif

#if defined(WIN32) || defined(WIN64)
#define OfxExport extern __declspec(dllexport)
#else
#define OfxExport extern
#endif

OfxExport OfxPlugin *OfxGetPlugin(int nth) {
    if (!ensure_real_loaded()) return NULL;
    if (!g_real_GetPlugin) return NULL;
    return g_real_GetPlugin(nth);
}

OfxExport int OfxGetNumberOfPlugins(void) {
    if (!ensure_real_loaded()) return 0;
    if (!g_real_GetNumber) return 0;
    return g_real_GetNumber();
}

#ifdef __cplusplus
} // extern "C"
#endif

BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved) {
    (void)lpvReserved;
    if (fdwReason == DLL_PROCESS_ATTACH) {
        g_hModule = (HMODULE)hinstDLL;
        InitializeCriticalSection(&g_cs);
        InterlockedExchange(&g_initialized, 0);
    } else if (fdwReason == DLL_PROCESS_DETACH) {
        if (g_real) {
            FreeLibrary(g_real);
            g_real = NULL;
            g_real_GetPlugin = NULL;
            g_real_GetNumber = NULL;
        }
        DeleteCriticalSection(&g_cs);
    }
    return TRUE;
}
```

除开一些辅助函数和锁相关的逻辑，这个代码很简单。我们重新定义了`OfxGetPlugin`和`OfxGetNumberOfPlugins`两个函数，当外界调用这两个函数时，我们便将其转发到真正的实现。

如何找到真正的实现？在函数`ensure_real_loaded`中，我们调用`LoadLibraryExW`来自动的加载预定义的dll文件，并且同时将目录加入查找范围。如果windows版本较老，就会回退到`AddDllDirectory`和`LoadLibraryW`两个函数来进行目录添加和dll加载。当加载完成后，我们就调用`GetProcAddress`来获取真正的实现函数的地址，然后记录下来。

在转发的时候，我们将调用，转发到这些记录下来的函数地址即可。注意，这个C代码连C标准库都没依赖，只依赖了winapi。主要是考虑到mingw环境下，c标准库需要引用一个非系统dll，有些麻烦。

下一个问题，我们怎么知道依赖哪些dll？在cmake中，我们链接到的是三方库的target，我们并不知道三方库还有哪些间接引用。不过cmake提供的`file(GET_RUNTIME_DEPENDENCIES)`函数非常方便。下面给出一个例子（先只看WIN32的部分，APPLE的部分后面会讲。另外我只给出了mingw的情况，msvc可能还需要其他处理）

```cmake
set(SRC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/src")
set(OFX_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/third_party/OpenFX-1.4")
set(OFX_SUPPORT_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/third_party/OFXSupport")

if(APPLE)
  set(INSTALL_PATH "/Library/OFX/Plugins")
  set(OFX_ARCH "MacOS")
elseif(WIN32)
  set(INSTALL_PATH "C:/Program Files/Common Files/OFX/Plugins")
  set(OFX_ARCH "Win64")
else()
  message(FATAL_ERROR "Platform not supported")
endif()

set(PLUGIN_NAME "SomePlugin")

if(WIN32)
  set(PLG_WRAPPER_NAME "${PLUGIN_NAME}")
  set(PLG_IMPL_NAME "SomePluginImpl")
  add_library(${PLG_WRAPPER_NAME} SHARED ${SRC_DIR}/plugin_wrapper.c) # 即上述的c代码
  target_link_libraries(${PLG_WRAPPER_NAME} PRIVATE shlwapi) # 我们调用了shlwapi.h，需要链接
else()
  set(PLG_IMPL_NAME "${PLUGIN_NAME}")
endif()

set(PLUGIN_SOURCES # 真正的插件实现源码
  ${SRC_DIR}/xxx.h
  ${SRC_DIR}/xxx.cpp
)

file(GLOB_RECURSE OFX_SUPPORT_SOURCES ${OFX_SUPPORT_ROOT}/Library/*.cpp)

get_filename_component(EXCLUDE_FILE "${OFX_SUPPORT_ROOT}/Library/ofxsHWNDInteract.cpp" ABSOLUTE)
list(REMOVE_ITEM OFX_SUPPORT_SOURCES "${EXCLUDE_FILE}") # NOTE: 很扯淡，但是这个文件确实被官方vcxproj里面排除了

list(APPEND PLUGIN_SOURCES ${OFX_SUPPORT_SOURCES})

add_library(${PLG_IMPL_NAME} SHARED ${PLUGIN_SOURCES})

target_include_directories(${PLG_IMPL_NAME} PRIVATE
  ${OFX_ROOT}/include
  ${OFX_SUPPORT_ROOT}/include
)

if(APPLE)
  set_target_properties(${PLG_IMPL_NAME} PROPERTIES LINK_FLAGS "-fvisibility=hidden -Wl,-exported_symbols_list,${CMAKE_CURRENT_SOURCE_DIR}/src/osx_symbols.txt")
  set_target_properties(${PLG_IMPL_NAME} PROPERTIES INSTALL_RPATH "@loader_path/../Frameworks;@loader_path/../Libraries")
  set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
elseif(WIN32)
  set_target_properties(${PLG_IMPL_NAME} PROPERTIES LINK_FLAGS "-shared -fvisibility=hidden -Xlinker --version-script=${CMAKE_CURRENT_SOURCE_DIR}/src/mingw_symbols.txt")
endif()

target_link_libraries(${PLG_IMPL_NAME} PUBLIC xxxlib)
target_link_libraries(${PLG_IMPL_NAME} PUBLIC yyylib)

if(WIN32)
  set_target_properties(${PLG_WRAPPER_NAME} PROPERTIES PREFIX "")
  set_target_properties(${PLG_WRAPPER_NAME} PROPERTIES SUFFIX ".ofx")
else()
  set_target_properties(${PLG_IMPL_NAME} PROPERTIES PREFIX "")
  set_target_properties(${PLG_IMPL_NAME} PROPERTIES SUFFIX ".ofx")
endif()

set(OFX_ARCH_NAME ${OFX_ARCH} CACHE STRING "OpenFX target OS and architecture")

if(WIN32)
  install(TARGETS ${PLG_WRAPPER_NAME}
    RUNTIME DESTINATION ${INSTALL_PATH}/${PLUGIN_NAME}.ofx.bundle/Contents/${OFX_ARCH_NAME}
    COMPONENT davinci
  )
endif()

install(TARGETS ${PLG_IMPL_NAME}
  RUNTIME DESTINATION ${INSTALL_PATH}/${PLUGIN_NAME}.ofx.bundle/Contents/${OFX_ARCH_NAME}
  COMPONENT davinci
)

install(FILES ${SRC_DIR}/Info.plist
  DESTINATION ${INSTALL_PATH}/${PLUGIN_NAME}.ofx.bundle/Contents
  COMPONENT davinci
)

set(LIB_INSTALL_DIR "UNKNOWN")
if(apple)
  set(LIB_INSTALL_DIR ${INSTALL_PATH}/${PLUGIN_NAME}.ofx.bundle/Contents/Libraries)
elseif(WIN32)
  set(LIB_INSTALL_DIR ${INSTALL_PATH}/${PLUGIN_NAME}.ofx.bundle/Contents/${OFX_ARCH_NAME})
endif()

if(APPLE)
  configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/fix_deps_macos.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/fix_deps.cmake"
    @ONLY
  )
elseif(WIN32)
  configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/fix_deps_win.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/fix_deps.cmake"
    @ONLY
  )
endif()

install(SCRIPT "${CMAKE_CURRENT_BINARY_DIR}/fix_deps.cmake"
  COMPONENT davinci
)
```

这里只是定义了一些target、一些目录，一些编译链接选项。下面是`fix_deps_win.cmake.in`，真正执行dll复制逻辑

```cmake
set(TARGET_BINARY "@INSTALL_PATH@/@PLUGIN_NAME@.ofx.bundle/Contents/@OFX_ARCH_NAME@/lib@PLG_IMPL_NAME@.dll")
set(LIB_DIR "@INSTALL_PATH@/@PLUGIN_NAME@.ofx.bundle/Contents/@OFX_ARCH_NAME@")

file(GET_RUNTIME_DEPENDENCIES
    LIBRARIES "${TARGET_BINARY}"
    RESOLVED_DEPENDENCIES_VAR _resolved_deps
    UNRESOLVED_DEPENDENCIES_VAR _unresolved_deps
    # 排除windows 系统库
    PRE_EXCLUDE_REGEXES "C:/WINDOWS/.*" "C:\\\\WINDOWS\\\\.*" "api-ms-win.*" "ext-ms-win.*"
    POST_EXCLUDE_REGEXES "C:/WINDOWS/.*" "C:\\\\WINDOWS\\\\.*" "api-ms-win.*" "ext-ms-win.*"
    DIRECTORIES "G:/Program_Files/msys64/mingw64/bin/"
)

message(STATUS "Found resolved dependencies for ${TARGET_BINARY}:\n ${_resolved_deps}")

set(_all_binaries_to_fix "${TARGET_BINARY}")

foreach(_lib ${_resolved_deps})
    get_filename_component(_lib_name "${_lib}" NAME)

    set(_new_lib_path "${LIB_DIR}/${_lib_name}")

    file(COPY "${_lib}"
      DESTINATION "${LIB_DIR}/"
      FOLLOW_SYMLINK_CHAIN
    )
    list(APPEND _all_binaries_to_fix "${_new_lib_path}")
endforeach()

if(_unresolved_deps)
    message(WARNING "Unresolved dependencies: ${_unresolved_deps}")
endif()
```

# macOS上的处理
