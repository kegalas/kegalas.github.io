---
title: "Atcoder Beginner Conteset 263 E（概率DP）"
date: 2022-08-07T12:57:17+08:00
draft: false
tags: [算法,概率DP]
categories: 算法
mathjax: true
markup: pandoc
---

题意：有$N$个方块，初始时站在$1$方块上。每个方块上有一个数$a_i$，等概率地随机选一个$0\sim a_i$的数，假设在$x$方块上选到了$y$，则跳到$x+y$方块。求跳到$N$上时，选取次数的期望值。

设$dp[i]$是从$i$走到$N$的期望，那么$dp[N]=0$。

记

$$
s = dp[i+1]+dp[i+2]+\cdots+dp[i+a_i]
$$

然后有

$$
dp[i] = \frac{s+a_i}{a_i+1}+\frac{1}{a_i+1}\left(\frac{s+2a_i}{a_i+1}\right)+\cdots
$$

$$
=\sum_{n=1}^\infty\left(\frac{1}{a_i+1}\right)^{n-1}\left(\frac{s+na_i}{a_i+1}\right)
$$

$$
=\sum_{n=1}^\infty\frac{s}{(a_i+1)^n}+\sum_{n=1}^\infty\frac{na_i}{(a_i+1)^{n}}
$$

$$
=\frac{s}{1-\frac{1}{a_i+1}}-s+1+\frac{1}{a_i}
$$


$$
=\frac{s+1}{a_i}+1
$$

最终

$$
dp[i]=\frac{dp[i+1]+dp[i+2]+\cdots+dp[i+a_i]+1+a_i}{a_i}
$$

其中$s$可以用前缀和优化。

$$
\frac{s+a_i}{a_i+1}=\frac{dp[i+1]+1+dp[i+2]+1+\cdots+dp[i+a_i]+1}{a_i+1}
$$

其实就是非常朴素的选择次数除以概率再相加。然后如果某一次选中了0就再往下算。

最终的代码如下

```cpp
#include <iostream>
#include <algorithm>
#include <cmath>
#include <string>
#include <cstring>
#include <map>
#include <set>
#include <queue>
#include <stack>
#include <vector>
#include <cstdint>
#include <cstdio>
#include <bitset>
#include <deque>

#define LL long long
#define ULL unsigned long long
#define i128 __int128
#define debug(a) std::cout<<#a<<"="<<a<<std::endl
#define lth(i,x,y) for(int i=x;i<=y;i++)
#define htl(i,x,y) for(int i=x;i>=y;i--)
#define mms(x) memset(x, 0, sizeof(x))

const int MAXN = 200005;
const int INF = 0x7fffffff;
const double EPS = 1e-8;
const int MOD = 998244353;
const double PI = acos(-1);

LL arr[MAXN];
LL inv[MAXN];
LL sum[MAXN];
LL dp[MAXN];

LL qPowMod(LL a, LL n, LL b){
    LL ans = 1;
    while(n){
        if(n&1){
            ans = ans%b*a%b;
        }
        a = a%b*a%b;
        n>>=1;
    }
    return ans;
}

LL fermat_inv(LL a, LL b){
    return qPowMod(a,b-2,b);
}

void init(int n){
    for(int i=1;i<=n;i++){
        inv[i] = fermat_inv(i,MOD);
    }
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);

    int n;
    std::cin>>n;

    init(n);

    for(int i=1;i<=n-1;i++){
        std::cin>>arr[i];
    }

    dp[n] = 0;
    sum[n] = 0;

    for(int i=n-1;i>=1;i--){
        dp[i] = (((sum[i+1]-sum[i+arr[i]+1]+arr[i]+1+MOD)%MOD)*(inv[arr[i]]%MOD))%MOD;
        sum[i] = (sum[i+1]+dp[i])%MOD;
    }

    std::cout<<dp[1]<<"\n";

    return 0;
}
```

其中第74行的

```cpp
dp[i] = (((sum[i+1]-sum[i+arr[i]+1]+arr[i]+1+MOD)%MOD)*(inv[arr[i]]%MOD))%MOD;
```

不能写为

```cpp
dp[i] = (((sum[i+1]-sum[i+arr[i]+1]+arr[i]+1)%MOD)*(inv[arr[i]]%MOD))%MOD;
```

原因应当是，会出现负数。虽然按照原本的想法，sum[i+1]肯定是大于等于sum[i+arr[i]]。但是sum本身也在进行取余运算，可能会导致小于的情况，导致负数的出现。此时要加一个998244353。
