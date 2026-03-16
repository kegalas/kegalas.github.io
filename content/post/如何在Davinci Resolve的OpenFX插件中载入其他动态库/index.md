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

    (void)outChars;

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

    InterlockedExchange(&g_initialized, (g_real != NULL) ? 1 : 0);
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
        // NOTE: https://learn.microsoft.com/zh-cn/windows/win32/api/libloaderapi/nf-libloaderapi-freelibrary
        // https://learn.microsoft.com/zh-cn/windows/win32/dlls/dllmain
        // 不应该在此调用FreeLibrary(g_real)，否则会陷入死锁。windows在程序关闭时会自动处理这件事。
        g_real = NULL;
        g_real_GetPlugin = NULL;
        g_real_GetNumber = NULL;
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

macOS拥有截然不同的动态库处理规则。简便来说，也有两条

1. 解析二进制文件中保存的“install name”，从中找到例如`/xxx/lib/a.dylib`或`@rpath/b.dylib`这种路径。
2. 回退到系统路径查找，例如`/usr/local/lib`，`/usr/lib`。但是不包括homebrew安装的库，即`/opt/homebrew`。

macOS完全不会像windows一样自动查找可执行文件目录，更别提动态库了。但是这个install name给了我们很大空间，它是可以修改的。我们可以先用`otool`工具看看它长什么样

```bash
$ otool -L SomePlugin.ofx

SomePlugin.ofx:
	@rpath/SomePlugin.ofx (compatibility version 0.0.0, current version 0.0.0)
	@rpath/libAAA.0.1.0.dylib (compatibility version 0.1.0, current version 0.1.0)
	/usr/lib/libc++.1.dylib (compatibility version 1.0.0, current version 2000.67.0)
	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1356.0.0)
```

可以看到，除了自身之外，他还引用了一个库libAAA，并且它的路径被标记为`@rpath`。`@rpath`其实是一个路径列表，macOS需要去这个列表里的目录查询是否存在这样的动态库，例如`libAAA.0.1.0.dylib`。这个列表可以用下列命令查询

```bash
otool -l SomePlugin.ofx
```

注意从原来大写的`-L`改成小写的`-l`，里面会有一些标记为LC_RPATH的项，这些项就组成了我们的`@rpath`需要的列表。我们可以通过`install_name_tool`来修改这个LC_RPATH，例如

```bash
install_name_tool -add_rpath "@loader_path/../Libraries" SomePlugin.ofx
```

我们就往插件里添加了一个rpath项了。同时，上述代码有一个新的概念`@loader_path`。其实，macOS还有一个`@executable_path`，它们是如下的含义

- `@executable_path`指向可执行程序的目录，也就是说，如果可执行文件`a`引用了`b.dylib`，对于`b.dylib`来说，`@executable_path`是`a`所在的目录。
- `@loader_path`指向动态库所在的目录，也就是`b.dylib`所在的目录。这对我们很有用。

就上面这个例子的`@loader_path/../Libraries`，就意味着，我们加载`SomePlugin.ofx`时，需要去其上级目录下的`Libraries`去找库，于是我们就可以把所有库全放在这个文件夹内。

因为每次编译都会重置`rpath`，而我们不太可能每次都去手动调用`install_name_tool`。好在cmake提供了一个便利的方式，之前我们的cmake文件中也写了，回顾一下：

```cmake
set_target_properties(${PLG_IMPL_NAME} PROPERTIES INSTALL_RPATH "@loader_path/../Frameworks;@loader_path/../Libraries")
```

这里，我同时添加了`Frameworks`和`Libraries`，作为经典的macOS目录结构。

到目前为止，我们解决了第一层的依赖问题，即系统知道如何去找`SomePlugin.ofx`的依赖了。但是，加入`SomePlugin.ofx`依赖`a.dylib`，而`a.dylib`又依赖`b.dylib`，那我们也得用`install_name_tool`去修改`a.dylib`，并且递归的修改`b.dylib`、`c.dylib`等等。

macOS的开发者们已经开发了一些工具，例如`macdylibbundler`，然而它非常不智能。首先它处理已存在的文件夹非常暴力，直接删除，导致我不能先处理我的库，再处理第三方库。另外，例如插件引用的是libfmt.dylib（一个符号链接别名），它会直接拆解符号连接，直接把libfmt.12.1.0.dylib复制过来并直接指向它，而非复制一个libfmt.12.1.0.dylib，建立一个libfmt.12.dylib指向它，再建立一个libfmt.dylib指向它。

我们又回到手写cmake上，之前用在Windows上的`GET_RUNTIME_DEPENENCIES`仍然可以用，并且会递归查找所有依赖。只要我们遍历所有依赖，然后用`install_name_tool`一个一个修改不就行了？然后我们会写出如下的**不完美**代码

```cmake
set(TARGET_BINARY "@INSTALL_PATH@/@PLUGIN_NAME@.ofx.bundle/Contents/@OFX_ARCH_NAME@/@PLG_IMPL_NAME@.ofx")
set(LIB_DIR "@INSTALL_PATH@/@PLUGIN_NAME@.ofx.bundle/Contents/Libraries")

file(GET_RUNTIME_DEPENDENCIES
    LIBRARIES "${TARGET_BINARY}"
    RESOLVED_DEPENDENCIES_VAR _resolved_deps
    UNRESOLVED_DEPENDENCIES_VAR _unresolved_deps
    # 排除macOS 系统库
    PRE_EXCLUDE_REGEXES "^/usr/lib/.*" "^/System/Library/.*"
    POST_EXCLUDE_REGEXES "^/usr/lib/.*" "^/System/Library/.*"
)

message(STATUS "Found resolved dependencies: ${_resolved_deps}")

set(_all_binaries_to_fix "${TARGET_BINARY}")

foreach(_lib ${_resolved_deps})
    get_filename_component(_lib_name "${_lib}" NAME)

    set(_new_lib_path "${LIB_DIR}/${_lib_name}")

    file(COPY "${_lib}"
      DESTINATION "${LIB_DIR}/"
      FOLLOW_SYMLINK_CHAIN
    )

    execute_process(COMMAND install_name_tool -id
      "@loader_path/../Libraries/${_lib_name}"
      "${_new_lib_path}"
    )
    list(APPEND _all_binaries_to_fix "${_new_lib_path}")
endforeach()

foreach(_target_file ${_all_binaries_to_fix})
  file(GET_RUNTIME_DEPENDENCIES
      LIBRARIES "${_target_file}"
      RESOLVED_DEPENDENCIES_VAR _sub_resolved_deps
      UNRESOLVED_DEPENDENCIES_VAR _sub_unresolved_deps
      # 排除macOS 系统库
      PRE_EXCLUDE_REGEXES "^/usr/lib/.*" "^/System/Library/.*"
      POST_EXCLUDE_REGEXES "^/usr/lib/.*" "^/System/Library/.*"
  )

  message(STATUS "Found resolved dependencies for ${_target_file}:\n ${_sub_resolved_deps}")

  foreach(_old_path ${_sub_resolved_deps})
    get_filename_component(_lib_name "${_old_path}" NAME)

    # 不管 _target_file 是否真的引用了 _old_path，
    # 如果不匹配，install_name_tool 会静默跳过。
    execute_process(COMMAND install_name_tool -change
      "${_old_path}"
      "@loader_path/../Libraries/${_lib_name}"
      "${_target_file}"
    )
  endforeach()
endforeach()

# 签名（由内而外）
message(STATUS "Signing all bundled binaries...")
foreach(_file ${_all_binaries_to_fix})
    execute_process(COMMAND codesign --force --sign - "${_file}")
endforeach()
execute_process(COMMAND codesign --force --deep --sign -
  "@INSTALL_PATH@/@PLUGIN_NAME@.ofx.bundle")

if(_unresolved_deps)
    message(WARNING "Unresolved dependencies: ${_unresolved_deps}")
endif()

```

这段代码直觉上很容易理解，我们先找到`SomePlugin.ofx`的所有依赖，然后复制过来修改它们自身的id。这一步主要是因为这些依赖通常在homebrew里，所以需要复制到`SomePlugin.ofx`的`@rpath`里。把它们的ID改成基于 `@loader_path` 的，防止它去系统路径乱找。

接着再找到这些库各自的依赖，故技重施使用`GET_RUNTIME_DEPENDENCIES`找。这些库在复制过来之后，通常还用“硬路径”直接指向自己依赖的`/opt/homebrew`库，我们要一一地改成`@loader_path/../Libraries/`中的库。

最后，由于改这些路径会破坏macOS的签名，所以还要重新进行签名。

听起来十分完美，然而有一个严重的问题。cmake用`GET_RUNTIME_DEPENDENCIES`得到的目录，和`otool -L`得到的目录可能不完全一样。例如：

```
cmake: /opt/homebrew/Cellar/ffmpeg/8.0.1_1/lib/libavcodec.62.dylib
otool: /opt/homebrew/opt/ffmpeg/lib/libavcodec.62.dylib
```

懂homebrew的可能都知道，这两个目录就是一个目录，只不过用了不同的符号链接。但是，`install_name_tool`非常蠢，只要两个目录字符串不严格相等，就无法替换目录。不得已，我们只能请出最古老的方法了：正则表达式手动硬解析

```cmake
set(TARGET_BINARY "@INSTALL_PATH@/@PLUGIN_NAME@.ofx.bundle/Contents/@OFX_ARCH_NAME@/@PLG_IMPL_NAME@.ofx")
set(LIB_DIR "@INSTALL_PATH@/@PLUGIN_NAME@.ofx.bundle/Contents/Libraries")

file(GET_RUNTIME_DEPENDENCIES
    LIBRARIES "${TARGET_BINARY}"
    RESOLVED_DEPENDENCIES_VAR _resolved_deps
    UNRESOLVED_DEPENDENCIES_VAR _unresolved_deps
    # 排除macOS 系统库
    PRE_EXCLUDE_REGEXES "^/usr/lib/.*" "^/System/Library/.*"
    POST_EXCLUDE_REGEXES "^/usr/lib/.*" "^/System/Library/.*"
)

message(STATUS "Found resolved dependencies: ${_resolved_deps}")

set(_all_binaries_to_fix "${TARGET_BINARY}")

foreach(_lib ${_resolved_deps})
    get_filename_component(_lib_name "${_lib}" NAME)

    set(_new_lib_path "${LIB_DIR}/${_lib_name}")

    file(COPY "${_lib}"
      DESTINATION "${LIB_DIR}/"
      FOLLOW_SYMLINK_CHAIN
    )

    execute_process(COMMAND install_name_tool -id
      "@loader_path/../Libraries/${_lib_name}"
      "${_new_lib_path}"
    )
    list(APPEND _all_binaries_to_fix "${_new_lib_path}")
endforeach()

foreach(_target_file ${_all_binaries_to_fix})
  execute_process(
    COMMAND otool -L "${_target_file}"
    OUTPUT_VARIABLE _otool_out
  )

  string(REPLACE "\n" ";" _lines "${_otool_out}")
  foreach(_line ${_lines})
    if(_line MATCHES "[ \t](/[^ ]+) \\(" AND NOT _line MATCHES "(/usr/lib/|/System/Library/)")
      string(REGEX REPLACE ".*[ \t](/[^ ]+) \\(.*" "\\1" _old_path "${_line}")

      get_filename_component(_lib_name "${_old_path}" NAME)

      execute_process(COMMAND install_name_tool
        -change "${_old_path}"
        "@loader_path/../Libraries/${_lib_name}"
        "${_target_file}"
      )
    endif()
  endforeach()

endforeach()

# 签名（由内而外）
message(STATUS "Signing all bundled binaries...")
foreach(_file ${_all_binaries_to_fix})
    execute_process(COMMAND codesign --force --sign - "${_file}")
endforeach()
execute_process(COMMAND codesign --force --deep --sign -
  "@INSTALL_PATH@/@PLUGIN_NAME@.ofx.bundle")

if(_unresolved_deps)
    message(WARNING "Unresolved dependencies: ${_unresolved_deps}")
endif()

```

我们直接用正则表达式，解析出任意不在系统目录下的库，然后直接进行替换，保证了字符串的严格相等。不过这也有一个坏处，如果某天苹果进行了一次巨大升级，我们的代码就要改了。

# 附录

## macOS下检测依赖库是否全都可以找到

给出以下代码

```cpp
#include <dlfcn.h>
#include <cstdio>

int main() {
    void* handle = dlopen("/Library/OFX/Plugins/SomePlugin.ofx.bundle/Contents/MacOS/SomePlugin.ofx", RTLD_NOW);
    if (!handle) {
        printf("failed!\n");
        return 1;
    }
    printf("ok!\n");
    dlclose(handle);
    return 0;
}
```

如果所有的`@rpath`、`@loader_path`之类的全部配置正确，那么他一定是可以输出`ok!`的。如果没有成功的话，可以使用otool去一个一个找，是谁缺了东西。

## windows下的依赖库查询

推荐使用`Dependencies`来查找依赖库。不过需要注意的是，这个查找是静态的查找，如果软件是通过`LoadLibrary`来在运行时加载，这个软件就没用了。这时候，我就推荐你使用微软出品的`SysinternalsSuite`中的`Procmon`来检测，对任意软件进行观察，观察它加载dll的行为，如果没找到肯定会报错。
