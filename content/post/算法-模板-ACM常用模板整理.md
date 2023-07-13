---
title: "算法竞赛常用模板整理"
date: 2021-12-18T15:57:37+08:00
draft: false
tags: [算法,模板]
description: 包含ICPC竞赛常用的一些模板
categories: 算法
mathjax: true
markup: pandoc
---

```CPP
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
#include <regex>
#include <unordered_map>
#include <unordered_set>

#define debug(a) std::cout<<#a<<"="<<a<<std::endl
#define lth(i,x,y) for(int i=x;i<=y;i++)
#define htl(i,x,y) for(int i=x;i>=y;i--)
#define mms(x) memset(x, 0, sizeof(x))

using LL = long long;
using ULL = unsigned long long;
using i128 = __int128;
using pii = std::pair<int,int>;
using pll = std::pair<LL,LL>;

int const MAXN = 200005;
int const INF = 0x7fffffff;
double const EPS = 1e-8;
int const MOD = 998244353;
double const PI = acos(-1);

int arr[MAXN];

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);

	

    return 0;
}

```

# 字符串

## KMP

```cpp
//复杂度n
//kmp,luogu3375
std::vector<int> prefixFunc(std::string const & str){
    //输入一个字符串，输出该字符串的前缀函数表
    //如果输入不是字符串而是一个数组，也可以很方便的修改为vector
    int n = str.length();
    std::vector<int> ans(n);

    for(int i=1, j=0;i<n;i++){
        //ans[0]=0，因为只看真前缀和真后缀
        while(j && str[i]!=str[j]) j = ans[j-1];
        if(str[i]==str[j]) j++;
        ans[i] = j;
    }

    return ans;
}

std::vector<int> KMP(std::string const & s, std::string const & p){
    //输入主串和模式串，返回所有匹配的开始下标，下标从0开始
    std::vector<int> vec;
    std::vector<int> pf = prefixFunc(p);
    int ns = s.size(), np = p.size();
    for(int i=0, j=0;i<ns;i++){
        while(j && s[i]!=p[j]) j = pf[j-1];
        if(s[i]==p[j]) j++;
        if(j==np){
            vec.push_back(i-j+2);
            j = pf[j-1];
        }
    }
    
    return vec;
}
```

## 字典树(Trie)

```cpp
//复杂度 插入或查找一次 模板串长度
#include <iostream>
#include <cstdio>
#include <cstring>

#define MAXN 500005

using namespace std;

int nxt[MAXN][26];
int cnt;

void init(){
    memset(nxt,0,sizeof(nxt));
    cnt = 1;
}

void insert(string s){
    int cur = 1;
    for(auto c:s){
        if(!nxt[cur][c-'a']){
            nxt[cur][c-'a']=++cnt;
        }
        cur = nxt[cur][c-'a'];
    }
}

bool find_prefix(string s){
    int cur=1;
    for(auto c:s){
        if (!nxt[cur][c - 'a'])
        {
            return false;
        }            
        cur = nxt[cur][c - 'a'];
    }
    return true;
}

int main(){

    int n;
    cin>>n;
    for(int i=1;i<=n;i++){
        string s1;
        cin>>s1;
        insert(s1);
    }
    while(1){
        string s1;
        cin>>s1;
        cout<<find_prefix(s1)<<endl;
    }


    return 0;
}
```

## AC自动机

```cpp
//复杂度 文本串长度+模板串长度之和
//AC自动机，luogu P3808

class AC{
public:
	int trie[MAXN][26], total;
	int end[MAXN],fail[MAXN];

    void init(int m){
        for(int i=0;i<=m;i++){
            end[i] = 0;
            fail[i] = 0;
            for(int j=0;j<26;j++) trie[i][j] = 0;
        }
        total = 0;
    }

	void insert(std::string const & str){
	    //插入模式串
		int u = 0;
		for(auto c:str){
			if(!trie[u][c-'a']){
				trie[u][c-'a'] = ++total;
			}
			u = trie[u][c-'a'];
		}
		end[u]++;
	}

	void buildFail(){
	    //构建fail指针
	    std::queue<int> qu;
		for(int i=0;i<26;i++){
			if(trie[0][i]) qu.push(trie[0][i]);
		}

		while(!qu.empty()){
			int u = qu.front();
			qu.pop();

			for(int i=0;i<26;i++){
				if(trie[u][i]){
					fail[trie[u][i]] = trie[fail[u]][i];	
					qu.push(trie[u][i]);
				}
				else{
					trie[u][i] = trie[fail[u]][i];
				}
			}
		}
	}

	int query(std::string const & str){
	    //查询主串str中出现了几个模式串
		int u = 0, res = 0;
		for(auto c:str){
			u = trie[u][c-'a'];
			for(int j = u ; j && end[j] != -1 ; j = fail[j]){
				res += end[j];
				end[j] = -1;
			}
		}

		return res;
	}
};
````

## 最小表示法

例如S = bcda, 则S的循环同构有cdab, dabc, abcd，其中字典序最小的是abcd，最小表示法就是求这个字典序最小的。当然题目里面可能是算数组里面字典序最小的。

```cpp
//最小表示法
//luogu P1368
//复杂度 O(n)
int arr[MAXN];

int minStr(int n){//数组下标从0开始，共n个；返回最小表示的开始下标
    int i=0, j=1, k=0;
    while(i<n && j<n && k<n){
        if(arr[(i+k)%n]==arr[(j+k)%n]){
            k++;
        }
        else if(arr[(i+k)%n]>arr[(j+k)%n]){//最大表示法时就把这一段和下一段换一下
            i += k+1;
            k = 0;
        }
        else{
            j += k+1;
            k = 0;
        }
        
        if(i==j) j++;
    }
    
    return std::min(i,j);
}
```

## Manacher

```CPP
//manacher 算法
//luogu P3805
//复杂度O(n)
std::vector<int> getD1(std::string const & str){
    //返回字符串以某一位为中心的，最长的（奇数长度）回文子串的长度半径
    //例如abcba中，d1[2] = 3
    int n = str.size();
    std::vector<int> d(n);
    for(int i=0,l=0,r=-1;i<n;i++){
        int j = l+r-i;
        int dj = j>=0?d[j]:0;
        d[i] = std::max(std::min(dj,j-l+1),0);
        
        if(j-dj<l){
            while(i-d[i]>=0 && i+d[i]<n && str[i-d[i]]==str[i+d[i]])
                d[i]++;
            l = i-d[i]+1, r = i+d[i]-1;
        }
    }
    
    return d;
}

std::vector<int> getD2(std::string const & str){
    //返回字符串以某一位的左边间隙为中心的，最长的（偶数长度）回文子串的长度半径
    //例如abba中，d2[2] = 2
    int n = str.size();
    std::vector<int> d(n);
    for(int i=0,l=0,r=-1;i<n;i++){
        int j = l+r-i;
        int dj = j>=0?d[j]:0;
        d[i] = std::max(std::min(dj,j-l),0);
        
        if(j-dj-1<l){
            while(i-d[i]-1>=0 && i+d[i]<n && str[i-d[i]-1]==str[i+d[i]])
                d[i]++;
            l = i-d[i], r = i+d[i]-1;
        }
    }
    
    return d;
}
```

# 数论

## 扩展欧几里得

```cpp
//复杂度 logn
//求解ax+by=gcd(a,b)的一组解
//扩展欧几里得
#include <iostream>

using namespace std;

int exgcd(int a,int b,int& x,int& y){
    if(b==0){
        x = 1;
        y = 0;//此时ax+by=gcd(a,b)中b=0，任何数与0的最大公约数是他本身，所以ax+0y=a，x=1 y=0
        return a;
    }
    int d = exgcd(b,a%b,y,x);
    y -= (a/b)*x;
    return d;
}

int main(){
    int a,b,x,y,z;
    cin>>a>>b;
    z = exgcd(a,b,x,y);
    cout<<x<<" "<<y<<" "<<z<<endl;
    //x,y的意义见开头，z即是最大公约数
    return 0;
}

```

## 欧几里得算法

```cpp
//复杂度 logn
#include <iostream>

inline int gcd(int a,int b){
    return b==0 ? a : gcd(b, a%b);
}

int main(){
    int a,b;
    std::cin>>a>>b;
    std::cout<<gcd(a,b);
    return 0;
}
```

## 欧拉筛

TODO: 用模板元编程实现编译期算素数

```cpp
//复杂度 n
//欧拉筛
#include <iostream>
#include <cstring>
#define MAXN 50005


using namespace std;

int main(){
    int n, prime[MAXN], cnt=0;
    bool is_not_prime[MAXN];
    cin>>n;
    //2~n中有多少个素数
    memset(is_not_prime, 0, sizeof(prime));

    for (int i = 2;i<=n;i++){
        if(!is_not_prime[i])    prime[++cnt] = i;
        for(int j = 1;j<=cnt&&i*prime[j]<=n;j++){
            is_not_prime[i*prime[j]] = 1;
            if(i%prime[j]==0) break;
        }
    }

    for(int i = 1;i<=cnt;i++){
        cout<<prime[i]<<" ";
        //输出素数
    }
    return 0;
}
```

## Miller-Rabin素数测试

```cpp
//对数 n 进行 k 轮测试的时间复杂度是 klog^3(n)
//miller-rabin
#include <iostream>
#include <ctime>
#include <cstdio>

using namespace std;

const int test_time = 10;

int qPowMod(int a, int m, int n){
    if(m==0) return 1;
    if(m==1) return (a%n);
    long long ans = 1; //不打ll会溢出
    while(m){
        if(m&1){
            ans = ans%n*a%n;
        }
        a = (long long)a%n*a%n;//这里不打ll会溢出导致判断错误
        m>>=1;
    }
    return ans;
}

bool millerRabin(int n) {
    if (n < 3 || n % 2 == 0) return n == 2;
    int a = n - 1, b = 0;
    while (a % 2 == 0) a /= 2, ++b;
    // test_time 为测试次数,建议设为不小于 8
    // 的整数以保证正确率,但也不宜过大,否则会影响效率
    for (int i = 1, j; i <= test_time; ++i) {
        int x = rand() % (n - 2) + 2, v = qPowMod(x, a, n);
        if (v == 1) continue;
        for (j = 0; j < b; ++j) {
            if (v == n - 1) break;
            v = (long long)v * v % n;
        }
        if (j >= b) return 0;
    }
    return 1;
}

int main(){
    srand(time(NULL));
    int n;
    cin>>n;
    if(millerRabin(n)){
        cout<<"Probably a prime"<<endl;
    }
    else{
        cout<<"A composite"<<endl;
    }
    return 0;
}
```

## 乘法逆元

```cpp
//复杂度 扩展欧几里得法和费马小定理法 logn, 线性求逆元，对于1~n这些数，总共n
//乘法逆元
//分为扩展欧几里得法、快速幂法、线性求逆元
//ax≡1(mod b)，x为a在乘法意义上的逆元，记作a^(-1)，或者inv(a)
//用扩展欧几里得法的角度看，就是求ax+by=1的整数解
//快速幂法利用费马小定理，需要b为素数

#include <iostream>
#include <cstdio>

using namespace std;

const int MAXN = 3000005;

int exgcd(int a,int b,int& x,int& y){
    if(b==0){
        x = 1;
        y = 0;
        return a;
    }
    int d = exgcd(b,a%b,y,x);
    y -= (a/b)*x;
    return d;
}

void exgcd_inv(int a, int b){
    int x,y;
    int d = exgcd(a,b,x,y);
    if(d!=1){//显然a，b要互质才会有逆元
        cout<<"None"<<endl;
    }
    else{
        cout<<(x+b)%b<<endl;//实际上是为了防止出现x为负数的情况
    }
}

int qPowMod(int a, int n, int b){
    int ans = 1;
    while(n){
        if(n&1){
            ans = ans%b*a%b;
        }
        a = a%b*a%b;
        n>>=1;
    }
    return ans;
}

void fermat_inv(int a, int b){
    cout<<qPowMod(a,b-2,b)<<endl;
}

long long inv[MAXN];

int main(){
    long long a,b,n;
    cin>>a>>b;
    exgcd_inv(a,b);
    //fermat_inv(a,b);

    /*
    //线性求逆元
    inv[1] = 1;
    for(long long i = 2;i<=n;i++){
        //inv[i] = -(b/i)*inv[b%i]; //这样写会出现负数
        inv[i] = (long long)(b-b/i)*inv[b%i]%b;
    }
    for(long long i=1;i<=n;i++){
        printf("%lld\n",inv[i]);
    }
    */

    return 0;
}
```

## 线性同余方程

```cpp
//复杂度 logn
//ax≡c (mod b)求解x
//和ax+by=c等价
#include <iostream>

using namespace std;

int exgcd(int a,int b,int& x,int& y){
    if(b==0){
        x = 1;
        y = 0;
        return a;
    }
    int d = exgcd(b,a%b,y,x);
    y -= (a/b)*x;
    return d;
}

int linearEquation(int a, int b, int c, int &x, int &y){
    int d = exgcd(a,b,x,y);
    if(c%d!=0) return -1;
    x = x*c/d;
    y = y*c/d;

    return d;
}

int main(){
    int a,b,c,x,y;

    cin>>a>>b;
    c=1;

    int d = linearEquation(a,b,c,x,y);
    //d是a,b的最大公约数

    if(d==-1){
        cout<<"None"<<endl;
    }
    else{
        //cout<<x<<" "<<y<<endl;
        //下面输出的是最小整数解
        int t = b/d;
        x = (x%t+t)%t;
        cout<<x<<endl;
    }

    return 0;
}

```

## 中国剩余定理

求解如下方程中的$x$

$$
\left\{\begin{matrix}
x \equiv a_1(mod\quad r_1) \\
x \equiv a_2(mod\quad r_2 \\
\vdots \\
x \equiv a_k(mod\quad r_k)
\end{matrix}\right.
$$

```cpp
//复杂度 klogn
#include <iostream>

using namespace std;

typedef long long ll;

const int MAXN = 10005;

long long a[MAXN],r[MAXN];

long long exgcd(ll a, ll b, ll &x, ll &y){
    if(!b){
        x=1;
        y=0;
        return a;
    }
    ll d = exgcd(b,a%b,x,y);
    ll tmp = x;
    x = y;
    y = tmp - (a/b)*y;
    return d;
}

int main(){
    int k;
    cin>>k;
    //共有k个方程
    for(int i=1;i<=k;i++){
        cin>>a[i]>>r[i];
		//x≡ai(mod ri)
    }
    
    ll n=1,ans=0;
    for(int i=1;i<=k;i++){
        n = n * r[i];
    }
    for(int i=1;i<=k;i++){
        ll m = n/r[i];
        ll x,y;
        exgcd(m,r[i],x,y);
        ans = (ans+a[i]*m*x%n)%n;
    }
    
    cout<<ans<<endl;
    //输出x的值
    return 0;
}
```

## 用乘法逆元计算组合数

TODO: 用模板元编程实现编译期算阶乘

根据

$$
(a/b)\%p=(a\times b^{-1})\%p=[(a\%p)\times(b^{-1}\%p)]\%p
$$

（如果加载不全，见[取余运算的分配律](https://kegalas.top/p/%E5%8F%96%E4%BD%99%E8%BF%90%E7%AE%97%E7%9A%84%E5%88%86%E9%85%8D%E5%BE%8B/)）

可以不用除法求出组合数。其中$b^{-1}$是$b$在模$p$意义下的逆元。

注意阶乘和其逆元的预处理。

```cpp
//复杂度 初始化为nlogn 后续查询为O(1)
#include <iostream>

#define LL long long

const int MAXN = 200005;
const int MOD = 998244353;

LL fac[MAXN];
LL invFac[MAXN];

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
    fac[0] = 1;
    invFac[0] = 1;

    for(int i=1;i<=n;i++){
        fac[i] = (fac[i-1]*i)%MOD;
        invFac[i] = fermat_inv(fac[i],MOD);
    }
}

LL comb(LL n, LL m){
    if(n<0||m<0||m>n) return 0;
    return (((fac[n]*invFac[m])%MOD)*invFac[n-m])%MOD;
}

int main(){
    int n,m;
    std::cin>>n>>m;
    
    init(n);
    
    std::cout<<comb(n,m)<<"\n";

    return 0;
}
```

# 图论

## 最短路

### Dijkstra

```cpp
//复杂度 优先队列实现为mlogm
#include <iostream>
#include <cstring>
#include <vector>
#include <queue>
#define MAXN 500005
#define MAXINT 0x7fffffff

using namespace std;

struct edge{
    int v,w;//下一点，权
};

struct node {
  int dis, u;
  bool operator>(const node& a) const { return dis > a.dis; }
};

int n,m,s;//点，边，起点

vector<edge> graph[MAXN];

int dis[MAXN];

bool tag[MAXN];

priority_queue<node, vector<node>, greater<node> > pq;

int main(){
    scanf("%d%d%d",&n,&m,&s);

    for(int i=1;i<=m;i++){
        int a,b,c;
        scanf("%d%d%d",&a,&b,&c);
        //起点，终点，边权
        edge t;
        t.v=b;
        t.w=c;
        graph[a].push_back(t);
    }

    for(int i=1;i<=n;i++){
        dis[i] = MAXINT;
        //初始化为无限远
    }

    dis[s]=0;
    node tmp;
    tmp.dis=0;
    tmp.u=s;
    pq.push(tmp);

    while (!pq.empty())
    {
        int u = pq.top().u;
        pq.pop();
        if(tag[u]) continue;
        tag[u]=1;
        for(auto g : graph[u]){
            int v = g.v, w = g.w;
            if(dis[v]>dis[u]+w){
                dis[v] = dis[u]+w;
                tmp.dis = dis[v];
                tmp.u = v;
                pq.push(tmp);
            }
        }
    }

    for(int i=1;i<=n;i++){
        printf("%d ",dis[i]);
    }

    return 0;
}
```

### Bellman-Ford

```cpp
//复杂度 nm
#include <iostream>
#include <vector>
#include <cstring>

#define MAXN 500005
#define INF  0x6fffffff

using namespace std;

struct edge{
    int v;//下一点
    int w;//权
};

int n,m,s;
int dis[MAXN];
vector<edge> graph[MAXN];

int main(){
    
    cin>>n>>m>>s;

    for(int i=1;i<=n;i++) dis[i]=INF; 

    for(int i=1;i<=m;i++){
        edge tmp;
        int a,b,c;
        cin>>a>>b>>c;
        //起点，终点，边权
        tmp.v=b;
        tmp.w=c;
        graph[a].push_back(tmp);
    }

    dis[s] = 0;

    bool flag;
    for (int i=1;i<=n;i++)//松弛n-1轮，若第n轮还能松弛，就说明有负环
    {
        flag = 0;
        for(int u=1;u<=n;u++){//这里看似是两层循环，实际上总数是边数，整个算法的复杂度是mn
            for (auto e : graph[u]){
                int w=e.w,v=e.v;
                if(dis[v]>dis[u]+w){
                    dis[v]=dis[u]+w;
                    flag = 1;
                }
            }
        }
        if(!flag){
            break;
        }
    }

    cout<<flag<<endl;//可以输出是否有负权环

    for(int i=1;i<=n;i++){
        //if(dis[i]!=INF){
            cout<<dis[i]<<" ";
        //}
        //else{
        //    cout<<"2147483647 ";//根据luogu P3371要输出这个数
        //}
        
    }
    
    return 0;
}
```

### SPFA

```cpp
//复杂度 nm
/*
    bellman-ford的优化
    只有上一次被松弛的结点，所连接的边，
    才有可能引起下一次的松弛操作
*/
#include <iostream>
#include <vector>
#include <queue>

#define MAXN 500005
#define INF  0x5fffffff

using namespace std;

struct edge{
    int v;
    int w;
};

int dis[MAXN];//距离
int cnt[MAXN];//算到达本节点所要经过的边数，若cnt>=n，则说明有负权环
bool tag[MAXN];//用于判断是否为上次松弛过的节点的边所连的点

int n,m,s;
queue<int> qu;
vector<edge> graph[MAXN];

int main(){
    cin>>n>>m>>s;

    for(int i=1;i<=n;i++){
        dis[i] = INF;
    }

    dis[s] = 0;
    tag[s] = 1;

    for(int i=1;i<=m;i++){
        int a,b,c;
        cin>>a>>b>>c;
        //起点，终点，边权
        edge tmp;
        tmp.v=b;
        tmp.w=c;
        graph[a].push_back(tmp);
    }

    qu.push(s);

    bool is_negative_circle = 0;

    while(!qu.empty()){
        if(is_negative_circle) break;
        int u = qu.front();
        qu.pop();
        tag[u]=0;
        for(auto e : graph[u]){
            int v = e.v, w = e.w;
            if(dis[v]>dis[u]+w){
                dis[v]=dis[u]+w;
                cnt[v]=cnt[u]+1;
                if(cnt[v]>=n) {
                    is_negative_circle = 1;
                    break;
                }
                if(!tag[v]){
                    qu.push(v);
                    tag[v]=1;
                }
            }
        }
    }

    cout<<is_negative_circle<<endl;//可以输出是否有负权环

    for(int i=1;i<=n;i++){
        //if(dis[i]!=INF){
            cout<<dis[i]<<" ";
        //}
        //else{
        //    cout<<"2147483647 ";//根据luogu P3371要输出这个数
        //}
    }

    return 0;
}
```

### Floyd

```cpp
//复杂度 n^3
//floyd
#include <iostream>
#include <cstring>

#define MAXN 5005
#define MAXINT 0x3fffffff //不能设置为int的最大值，否则后面加法可能导致溢出

using namespace std;

int graph[MAXN][MAXN];

int main(){
    int n,m;//点数，边数
    cin>>n>>m;
    for(int i = 1;i<=n;i++){
        for(int j = 1;j<=n;j++){
            graph[i][j] = MAXINT;
        }
    }
    for(int i = 1;i<=n;i++){
        graph[i][i] = 0;
    }
    for(int i=1;i<=m;i++){
        int a,b,v;
        cin>>a>>b>>v;//起点，终点，边权
        graph[a][b] = v;
    }

    for(int k = 1;k<=n;k++){
        for(int i=1;i<=n;i++){
            for(int j=1;j<=n;j++){
                graph[i][j] = min(graph[i][j],graph[i][k]+graph[k][j]);
            }
        }
    }


    for(int i = 1;i<=n;i++){
        for(int j=1;j<=n;j++){
            cout<<graph[i][j]<<" ";
        }
        cout<<endl;
    }

    return 0;
}
```

## 拓扑排序

```cpp
//复杂度 n
//拓扑排序
#include <iostream>
#include <vector>
#include <queue>

using namespace std;

struct graph{
    int value;
    int to;
};

struct point
{
    int in_num;
    vector<graph> graph1;
}point1[100];

int main(){
    int n,m;
    cin>>n>>m;

    for(int i = 1;i<=n;i++){
        point1[i].in_num=0;
    }
    
    for (int i=1;i<=m;i++){
        int a,b,value;
        cin>>a>>b;//起点，终点；本次是无权图
        graph tmp;
        tmp.to=b;
        tmp.value = 0;
        point1[a].graph1.push_back(tmp);
        point1[b].in_num++;
    }
    
    queue<int> que;
    vector<int> ans;

    for (int i = 1; i <= n; i++)
    {
        if(point1[i].in_num == 0){
            que.push(i);
        }
    }

    while (!que.empty())
    {
        int p = que.front();
        que.pop();
        ans.push_back(p);
        for(int i = 0;i<point1[p].graph1.size();i++){
            int next = point1[p].graph1[i].to;
            point1[next].in_num--;
            if(point1[next].in_num==0){
                que.push(next);
            }
        }

    }

    if(ans.size()==n){
        for(int i = 0;i<ans.size();i++){
            cout<<ans[i]<<" ";
        }
    }
    else{
        cout<<"none";
    }

    return 0;
}
```

## 最小生成树

### Kruskal

```cpp
//复杂度 mlogm
#include <iostream>
#include <algorithm>
#define MAXN 200005

using namespace std;

int u[MAXN],v[MAXN],w[MAXN];
int r[MAXN];//临时边序号，间接排序
int find_sets[MAXN];//并查集

bool cmp(const int i, const int j){return w[i]<w[j];}

int find(int x){return find_sets[x]==x ? x : find_sets[x] = find(find_sets[x]);}

int main(){
    int n,m;//点数和边数

    cin>>n>>m;
    for(int i=1;i<=m;i++){
        cin>>u[i]>>v[i]>>w[i];
        //起点，终点，边权
    }

    for(int i = 1;i<=n;i++){
        find_sets[i]=i;
    }
    for(int i=1;i<=m;i++){
        r[i]=i;
    }

    sort(r+1,r+m+1,cmp);

    int ans = 0;
    //int cnt=0;

    for(int i=1;i<=m;i++){
        int tmp = r[i];
        int x = find(u[tmp]);
        int y = find(v[tmp]);

        if(x!=y){
            ans += w[tmp];
            find_sets[x] = y;
            //cnt++;
        }
    }
    //计数，如果小于n-1则不连通
    /*
    if(cnt<n-1){
        cout<<"orz"<<endl;
        return 0;
        //如果是多个样例注意这个return 0
    }
    */

    cout<<ans<<endl;

    return 0;
}
```

### Prim算法

```cpp
//复杂度 (m+n)logn
#include <iostream>
#include <vector>
#include <queue>

using namespace std;

const int MAXN = 5005;
const int MAXM = 200005;
const int INF = 0x5fffffff;

struct edge{
    int v,w;

    edge()=default;
    edge(int v,int w):v(v),w(w){}

    bool operator>(const edge& x) const {return w>x.w;}
};

vector<edge> graph[MAXN];
bool vis[MAXN];

priority_queue<edge, vector<edge>, greater<edge> > pq;

//以下orz代表不连通

int main(){
    int n,m;//点数，边数
    scanf("%d%d",&n,&m);
    int ans = 0;
    int cnt = 1;

    for(int i=1;i<=m;i++){
        int a,b,c;//起点，终点，边权
        scanf("%d%d%d",&a,&b,&c);
        graph[a].push_back(edge(b,c));
        graph[b].push_back(edge(a,c));
        //无向图
    }

    for(int i=0;i<graph[1].size();i++){
        pq.push(graph[1][i]);
    }
    vis[1]=true;

    while(cnt!=n&&!pq.empty()){
        edge minx=pq.top();
        pq.pop();
        while(vis[minx.v]){
            if(pq.empty()){
                cout<<"orz"<<endl;
                return 0;
            }
            minx=pq.top();
            pq.pop();
        }
        
        vis[minx.v] = true;
        ans+=minx.w;
        cnt++;
        
        for(int i=0;i<graph[minx.v].size();i++){
            if(!vis[graph[minx.v][i].v])
                pq.push(graph[minx.v][i]);
        }
    }

    if(cnt<n){
        cout<<"orz"<<endl;
    }
    else{
        cout<<ans<<endl;
    }

    return 0;
}
```

## 最小树形图

### 朱刘算法

```cpp
//复杂度 nm
//最小树形图，朱刘算法
//从根节点能到达其他所有点
//luogu4716
#include <iostream>

using namespace std;

const int MAXN = 105;
const int MAXM = 10005;
const int INF = 0x7fffffff;

struct Edge{
    int u,v,w;
};

Edge edge[MAXM]; 
int vis[MAXN],id[MAXN];
int in[MAXN],pre[MAXN];
int n,m,root;

int zhuliu(){
    int ans = 0;
    for(;;){
        for(int i=1;i<=n;i++) in[i]=INF;
        
        for(int i=1;i<=m;i++){
            int u = edge[i].u;
            int v = edge[i].v;
            if(u!=v&&edge[i].w<in[v]){//遍历所有边，找到对每个点的最短入边
                in[v] = edge[i].w;
                pre[v] = u;
            }
        }
        
        for(int i=1;i<=n;i++){
            if(i!=root&&in[i]==INF){
                return -1;//无解
            }
        }
        
        int cnt = 0;//记录环数以及下一次循环的点数
        
        for(int i=1;i<=n;i++){
            vis[i] = -1;
            id[i] = -1;
        }
        
        in[root] = 0;
        
        for(int i=1;i<=n;i++){
            if(i==root) continue;
            ans += in[i];
            int v=i;
            while(vis[v]!=i&&id[v]==-1&&v!=root){
                vis[v] = i;
                v = pre[v];
            }
            if(v!=root&&id[v]==-1){
                id[v] = ++cnt;
                for(int u=pre[v];u!=v;u=pre[u]) id[u] = cnt;
            }
        }
        
        if(cnt==0){//无环，得到解
            break;
        }
        
        for(int i=1;i<=n;i++){
            if(id[i]==-1) id[i]=++cnt;
        }
        
        for(int i=1;i<=m;i++){
            int u = edge[i].u;
            int v = edge[i].v;
            edge[i].u = id[u];
            edge[i].v = id[v];
            if(edge[i].u!=edge[i].v) edge[i].w -= in[v];
        }
        
        n = cnt;
        root = id[root];
    }
    return ans;
}

int main(){
    cin>>n>>m>>root;//点数，边数，根节点序号
    for(int i=1;i<=m;i++){
        cin>>edge[i].u>>edge[i].v>>edge[i].w;
        //起点，终点，边权
    }
    cout<<zhuliu()<<endl;
    return 0;
}
```

## 二分图匹配

### 匈牙利算法

```cpp
//复杂度 nm
//luogu 3386
//求二分图最大匹配，根据定理，最大匹配=最小点覆盖，以及最小边覆盖=点数-最大匹配
#include <iostream>
#include <cstring>

const int MAXN = 505;
bool graph[MAXN][MAXN];
bool vis[MAXN];
int toLeft[MAXN];

bool match(int const & i, int const & rightNum){
    for(int j=1;j<=rightNum;j++){
        if(graph[i][j]&&!vis[j]){
            vis[j] = true;
            if(toLeft[j]==0 || match(toLeft[j], rightNum)){
                toLeft[j] = i;
                return true;
            }
        }
    }
    return false;
}

int hungarian(int const & leftNum, int const & rightNum){
    int cnt = 0;

    for(int i=1;i<=leftNum;i++){
        std::memset(vis,0,sizeof(vis));
        if(match(i,rightNum)) cnt++;
    }

    return cnt;
}

int main(){
    int n,m,e;//左边点数，右边点数，边数
    std::cin>>n>>m>>e;
    
    for(int i=1;i<=e;i++){
        int x,y;
        std::cin>>x>>y;
        graph[x][y] = true;
    }

    std::cout<<hungarian(n,m)<<"\n";

    return 0;
}
```

### KM算法

```cpp
//luogu 6577
//二分图的最大权匹配，必须是完备匹配才能正确运行，即左右各n个点，最大匹配有n条边
//随机数据O(n^3)，最坏O(n^4)，所以luogu上会超时一些数据
#include <iostream>
#include <cstring>

typedef long long LL;

const int MAXN = 505;
const LL INF = 1e17;
LL graph[MAXN][MAXN];
LL labelL[MAXN], labelR[MAXN];
bool visL[MAXN],visR[MAXN];
int toLeft[MAXN];
LL upd[MAXN];

bool match(int const & i, int const & pointNum){
    visL[i] = true;
    for(int j=1;j<=pointNum;j++){
        if(!visR[j]){
            if(labelL[i]+labelR[j]-graph[i][j]==0){
                visR[j] = true;
                if(!toLeft[j]||match(toLeft[j],pointNum)){
                    toLeft[j] = i;
                    return true;
                }
            }
            else{
                upd[j] = std::min(upd[j],labelL[i]+labelR[j]-graph[i][j]);
            }
        }
    }
    return false;
}

LL KM(int const & pointNum){
    std::memset(toLeft,0,sizeof(toLeft));
    for(int i=1;i<=pointNum;i++){
        labelL[i] = -INF;
        labelR[i] = 0;
        for(int j=1;j<=pointNum;j++) labelL[i] = std::max(labelL[i], graph[i][j]);
    }

    for(int i=1;i<=pointNum;i++){
        for(int j=1;j<=pointNum;j++) upd[j] = INF;
        while(true){
            std::memset(visL,0,sizeof(visL));
            std::memset(visR,0,sizeof(visR));
            if(match(i,pointNum)) break;
            LL delta = INF;
            for(int j=1;j<=pointNum;j++)
                if(!visR[j]) delta = std::min(delta,upd[j]);
            for(int j=1;j<=pointNum;j++){
                if(visL[j]) labelL[j] -= delta;
                if(visR[j]) labelR[j] += delta;
            }
        }
    }

    LL ans = 0;
    for(int i=1;i<=pointNum;i++) ans += graph[toLeft[i]][i];
    return ans;//输出最大权匹配的权值和
}

int main(){
    int n,e;//一边的点数；边数
    std::cin>>n>>e;
    
    for(int i=1;i<=n;i++){
        for(int j=1;j<=n;j++){
            graph[i][j] = -INF;
        }
    }
    
    for(int i=1;i<=e;i++){
        int x,y;
        std::cin>>x>>y;
        std::cin>>graph[x][y];
    }

    std::cout<<KM(n)<<"\n";
    for(int i=1;i<=n;i++){
        std::cout<<toLeft[i]<<" ";
    }
    std::cout<<"\n";

    return 0;
}
```

## 动态维护二分图判定

只判定一次可以用涂色法。动态加边可以用扩展域并查集（可撤销）来实现。

```cpp
std::stack<pii> stk;

class DSU{
public:
    int fa[MAXN*2], rk[MAXN*2];
    
    void init(int n){
        for(int i=1;i<=n;i++) fa[i] = i, rk[i] = 1;
    }
    
    int find(int x){
        return fa[x]==x ? x : find(fa[x]);
    }
    
    void merge(int x, int y){
        x = find(x), y = find(y);
        if(x==y) return;
        if(rk[x]>rk[y]) std::swap(x,y);
        fa[x] = y;
        stk.push({x,rk[x]==rk[y]});//保存操作记录，也可以用stack以外的数据结构
        if(rk[x]==rk[y]) rk[y]++;
    }
    
    void erase(pii p){
        rk[find(p.first)]-=p.second;
        fa[p.first] = p.first;
    }
};

bool add(int x, int y){
    //设总共n个点，每次添加一条边<x,y>，注意没有边也算二分图
    dsu.merge(x,y+n);
    dsu.merge(y,x+n);
    if(dsu.find(x)==dsu.find(x+n) || dsu.find(y)==dsu.find(y+n)){
        //说明不是二分图
        return false;
    }
    else{
        //说明是二分图
        return true;
    }
}
//删边的时候，需要注意用一个pii删（调用erase函数），first保存了<x,y>这条边的x（y可以用find函数找出来），second保存了秩的数据，在删边时有用。至于删完是不是二分图，我没有找到办法。我做过的题目都是，添加了这条边后不再是二分图，输出某个结果，然后撤销这条边（之后显然是二分图）。要不就是只有加边的。直接删去任意一条边的题目并没有遇到过。

```

## 网络流

### 最大流

#### Ford-Fulkerson

##### DFS实现的Ford-Fulkerson

```cpp
//luogu 3376
//复杂度O(ef)，边数乘以最大流，所以在luogu上这题超时
#include <iostream>
#include <cstring>
#include <vector>

typedef long long LL;

const int MAXN = 205;
const LL INF = 0xffffffff;

struct Edge{
    int v;LL w;//指向的点，容量
    Edge(int v_, LL w_):v(v_),w(w_){}
};

std::vector<Edge> edges;
std::vector<std::vector<int> > graph(MAXN);//vector版的链式前向星
bool vis[MAXN];

LL DFS(int const & p, LL const & flow, int const & s, int const & t){
    if(p==t) return flow;
    vis[p] = true;

    int size = graph[p].size();
    for(int i=0 ; i<size ; i++){
        int eg = graph[p][i];
        int to = edges[eg].v;
        LL vol = edges[eg].w, c;

        if(vol>0 && !vis[to] && (c=DFS(to,std::min(flow,vol),s,t))!=-1){
            edges[eg].w -= c;
            edges[eg^1].w += c;
            return c;
        }
    }

    return -1;
}

LL FF(int const & p, LL const & flow, int const & s, int const & t){
    LL ans = 0, c;
    while((c=DFS(p,flow,s,t))!=-1){
        std::memset(vis,0,sizeof(vis));
        ans += c;
    }
    return ans;
}

int main(){
    int n,m,s,t;//点数，边数，源点，汇点
    std::cin>>n>>m>>s>>t;

    for(int i=1;i<=m;i++){
        int u,v;LL w;
        std::cin>>u>>v>>w;//起点，终点，边容量
        graph[u].push_back(edges.size());
        edges.push_back(Edge(v,w));
        graph[v].push_back(edges.size());
        edges.push_back(Edge(u,0));
    }

    std::cout<<FF(s,INF,s,t)<<"\n";//输出最大流

    return 0;
}
```

##### EdmondsKarp

```cpp
//luogu P3376
//EK算法的时间复杂度为O(nm^2)，这题不会超时
#include <iostream>
#include <cstring>
#include <vector>
#include <queue>

typedef long long LL;

const int MAXN = 205;
const LL INF = 0xffffffff;

struct Edge{
    int v;LL w;//指向的点，容量
    Edge(int v_, LL w_):v(v_),w(w_){}
};

std::vector<Edge> edges;
std::vector<std::vector<int> > graph(MAXN);//vector版的链式前向星
int last[MAXN];
LL flow[MAXN];

bool BFS(int const & s, int const & t){
    std::memset(last,-1,sizeof(last));
    std::queue<int> qu;
    qu.push(s);
    flow[s] = INF;
    while(!qu.empty()){
        int p = qu.front();
        qu.pop();
        if(p == t) break;

        int size = graph[p].size();
        for(int i=0;i<size;i++){
            int eg = graph[p][i];
            int to = edges[eg].v;
            LL vol = edges[eg].w;
            if(vol>0 && last[to] == -1){
                last[to] = eg;
                flow[to] = std::min(flow[p], vol);
                qu.push(to);
            }
        }
    }
    return last[t] != -1;
}

LL EK(int const & s, int const & t){
    LL ans = 0;

    while(BFS(s,t)){
        ans += flow[t];
        for(int i=t;i!=s;i=edges[last[i]^1].v){
            edges[last[i]].w -= flow[t];
            edges[last[i]^1].w += flow[t];
        }
    }

    return ans;
}

int main(){
    int n,m,s,t;//点数，边数，源点，汇点
    std::cin>>n>>m>>s>>t;

    for(int i=1;i<=m;i++){
        int u,v;LL w;
        std::cin>>u>>v>>w;
        graph[u].push_back(edges.size());
        edges.push_back(Edge(v,w));
        graph[v].push_back(edges.size());
        edges.push_back(Edge(u,0));
    }

    std::cout<<EK(s,t)<<"\n";

    return 0;
}
```

##### Dinic

```cpp
//luogu P3376
//Dinic算法的时间复杂度为O(n^2m)，这题不会超时
#include <iostream>
#include <cstring>
#include <vector>
#include <queue>

typedef long long LL;

const int MAXN = 205;
const LL INF = 0xffffffff;

struct Edge{
    int v;LL w;//指向的点，容量
    Edge(int v_, LL w_):v(v_),w(w_){}
};

std::vector<Edge> edges;
std::vector<std::vector<int> > graph(MAXN);//vector版的链式前向星
std::vector<int> cur(MAXN);
int level[MAXN];

bool BFS(int const & s, int const & t){//BFS分层
    std::memset(level, -1, sizeof(level));
    level[s] = 0;
    std::queue<int> qu;
    qu.push(s);

    while(!qu.empty()){
        int p = qu.front();
        qu.pop();
        int size = graph[p].size();
        for(int i=0;i<size;i++){
            int eg = graph[p][i];
            int to = edges[eg].v;
            LL vol = edges[eg].w;
            if(vol>0 && level[to] == -1){
                level[to] = level[p] + 1;
                qu.push(to);
            }
        }
    }

    return level[t] != -1;
}

LL DFS(int const & p, LL const & flow, int const & s, int const & t){
    if(p==t) return flow;

    LL surplus = flow;//剩余流量

    int size = graph[p].size();
    for(int & i=cur[p];i<size && surplus;i++){//这里i是引用，是当前弧优化
        int eg = graph[p][i];
        int to = edges[eg].v;
        LL vol = edges[eg].w;
        if(vol>0 && level[to]==level[p]+1){
            LL c = DFS(to, std::min(vol, surplus), s, t);
            surplus -= c;
            edges[eg].w -= c;
            edges[eg^1].w += c;
        }
    }

    return flow - surplus;
}

LL Dinic(int const & p, LL const & flow, int const & s, int const & t){
    LL ans = 0;
    while(BFS(s,t)){
        cur.assign(MAXN,0);
        ans += DFS(p,flow,s,t);
    }
    return ans;
}

int main(){
    int n,m,s,t;//点数，边数，源点，汇点
    std::cin>>n>>m>>s>>t;

    for(int i=1;i<=m;i++){
        int u,v;LL w;
        std::cin>>u>>v>>w;
        graph[u].push_back(edges.size());
        edges.push_back(Edge(v,w));
        graph[v].push_back(edges.size());
        edges.push_back(Edge(u,0));
    }

    std::cout<<Dinic(s,INF,s,t)<<"\n";

    return 0;
}
```

## 连通性相关

### 强连通分量

#### Tarjan算法

```cpp
//复杂度 n+m
#include <iostream>
#include <vector>
#include <stack>

using namespace std;

const int MAXN = 5005;
const int MAXM = 10005;

int dfn[MAXN], low[MAXN], instk[MAXN], scc[MAXN], cnt=0, cscc=0;
//scc代表每个店所属的强连通分量的编号
vector<int> edges[MAXN];
stack<int> stk;

void tarjan(int u){
    low[u] = dfn[u] = ++cnt;
    instk[u] = 1;
    stk.push(u);
    for(int i=0;i<edges[u].size();i++){
        int v = edges[u][i];
        if(!dfn[v]){
            tarjan(v);
            low[u] = min(low[u],low[v]);
        }
        else if(instk[v]){
            low[u] = min(low[u], dfn[v]);
        }
    }
    if(low[u]==dfn[u]){
        int top;
        cscc++;
        do{
            top = stk.top();
            stk.pop();
            instk[top] = 0;
            scc[top] = cscc;
        }while(top!=u);
    }
}

int main(){
    int n,m;
    cin>>n>>m;
    //点数，边数
    for(int i=1;i<=m;i++){
        int a,b;
        cin>>a>>b;
        //起点，终点
        edges[a].push_back(b);
    }
    for(int i=1;i<=n;i++){
        if(!dfn[i])
            tarjan(i);
    }
    for(int i=1;i<=n;i++){
        cout<<scc[i]<<endl;
    }
    return 0;
}
```

## 割点

### Tarjan算法

```cpp
//复杂度 n+m
//tarjan求割点,luogu P3388
#include <iostream>
#include <vector>
#include <stack>
#include <algorithm>

using namespace std;

const int MAXN = 20005;
const int MAXM = 100005;

int dfn[MAXN], low[MAXN], cnt=0;
vector<int> edges[MAXN];
vector<int> cut;//存储割点

void tarjan(int u, bool root = true){
    int tot = 0;
    low[u] = dfn[u] = ++cnt;
    for(int i=0;i<edges[u].size();i++){
        int v = edges[u][i];
        if(!dfn[v]){
            tarjan(v,false);
            low[u] = min(low[u],low[v]);
            tot += (low[v]>=dfn[u]);//统计满足的点的个数
        }
        else{
            low[u] = min(low[u], dfn[v]);
        }
    }
    if(tot>root){//如果是根节点，则需要有至少两个子树，否则只需要有一个子树
        cut.push_back(u);
    }
}

int main(){
    int n,m;
    cin>>n>>m;
    //点数，边数
    for(int i=1;i<=m;i++){
        int a,b;
        cin>>a>>b;
        //起点，终点
        edges[a].push_back(b);
        edges[b].push_back(a);
        //无向图
    }
    for(int i=1;i<=n;i++){
        if(!dfn[i])
            tarjan(i);
    }
    cout<<cut.size()<<endl;
    sort(cut.begin(),cut.end());
    for(int i=0;i<cut.size();i++){
        cout<<cut[i]<<" ";
        //输出割点的编号
    }
    return 0;
}
```

## 割边

### Tarjan算法

```cpp
//复杂度 n+m
//tarjan求割边
#include <iostream>
#include <vector>
#include <stack>
#include <algorithm>

using namespace std;

const int MAXN = 20005;
const int MAXM = 100005;

int dfn[MAXN], low[MAXN], cnt=0, fa[MAXN];//fa记录父节点
vector<int> edges[MAXN];
vector<pair<int, int>> bridges;//存储割边

void tarjan(int u){
    low[u] = dfn[u] = ++cnt;
    for(int i=0;i<edges[u].size();i++){
        int v = edges[u][i];
        if(!dfn[v]){
            fa[v] = u;
            tarjan(v);
            low[u] = min(low[u],low[v]);            
            if(low[v]>dfn[u])
                bridges.emplace_back(u,v);
        }
        else if(fa[u]!=v){
            low[u] = min(low[u], dfn[v]);
        }
    }
}

int main(){
    int n,m;
    cin>>n>>m;
    //点数，边数
    for(int i=1;i<=m;i++){
        int a,b;
        cin>>a>>b;
        //起点，终点
        edges[a].push_back(b);
        edges[b].push_back(a);
		//无向图
    }
    for(int i=1;i<=n;i++){
        if(!dfn[i])
            tarjan(i);
    }
    cout<<bridges.size()<<endl;
    for(int i=0;i<bridges.size();i++){
        cout<<bridges[i].first<<" "<<bridges[i].second<<endl;
        //输出割边
    }
    return 0;
}
```

# 计算几何

## 二维凸包

### Andrew扫描法

```cpp
//复杂度 nlogn
#include <iostream>
#include <cstdio>
#include <cstring>
#include <vector>
#include <algorithm>

#define MAXN 50005

using namespace std;

struct Point{
    double x,y;
    Point()=default;
    Point(double x, double y):x(x),y(y){}
    Point operator + (Point p){
        return Point(x+p.x, y+p.y);
    }
    Point operator - (Point p){
        return Point(x-p.x, y-p.y);
    }
    Point operator * (double d){
        return Point(x*d, y*d);
    }
    double dot(Point p){//点积
        return x*p.x+y*p.y;
    }
    double det(Point p){//叉积
        return x*(p.y)-(p.x)*y;
    }
};

int n;
Point po[MAXN*2];

bool cmp(Point& a, Point& b){
    if(a.x!=b.x) return a.x<b.x;
    return a.y<b.y;
}

vector<Point> convexHull(){
	//返回凸包上的点
    int k = 0;
    vector<Point> qs;
    for(int i=0;i<n;i++){
        while(k>1&&(qs[k-1]-qs[k-2]).det(po[i]-qs[k-1])<=0){
            qs.erase(qs.end()-1);
            k--;
        }
        qs.push_back(po[i]);
        k++;
    }
    for(int i=n-2,t=k;i>=0;i--){
        while(k>t&&(qs[k-1]-qs[k-2]).det(po[i]-qs[k-1])<=0) {
            qs.erase(qs.end()-1);
            k--;
        }
        qs.push_back(po[i]);
        k++;
    }
    qs.erase(qs.end()-1);
    return qs;
}

int main(){
    cin>>n;
    for(int i=0;i<n;i++){
        cin>>po[i].x>>po[i].y;
        //输入点的横纵坐标
    }

    sort(po,po+n,cmp);

    for(auto p:convexHull()){
        cout<<p.x<<" "<<p.y<<endl;
    }

    return 0;
}
```

## 旋转卡壳求最远点对

```cpp
//复杂度 nlogn，其中求凸包nlogn，旋转卡壳本身为n
//Luogu P1452
//旋转卡壳和凸包
#include <iostream>
#include <cstring>
#include <vector>
#include <algorithm>

#define MAXN 50005

using namespace std;

struct Point{
    int x,y;
    Point()=default;
    Point(int x, int y):x(x),y(y){}
    Point operator - (Point p){
        return Point(x-p.x, y-p.y);
    }
    Point operator + (Point p){
        return Point(x+p.x, y+p.y);
    }
    Point operator * (int d){
        return Point(x*d, y*d);
    }
    int dot(Point p){
        return x*p.x+y*p.y;
    }
    int det(Point p){
        return x*(p.y)-y*(p.x);
    }
};

bool cmp(Point& a, Point& b){
    if(a.x!=b.x) return a.x<b.x;
    return a.y<b.y;
}

int n;
Point po[MAXN];

vector<Point> convexHull(){
	//返回凸包上的点
    vector<Point> ans;
    int k = 0;
    for(int i=0;i<n;i++){
        while(k>1&&(ans[k-1]-ans[k-2]).det(po[i]-ans[k-1])<=0){
            ans.erase(ans.end()-1);
            k--;
        }
        ans.push_back(po[i]);
        k++;
    }
    for(int i=n-2,t=k;i>=0;i--){
        while(k>t&&(ans[k-1]-ans[k-2]).det(po[i]-ans[k-1])<=0){
            ans.erase(ans.end()-1);
            k--;
        }
        ans.push_back(po[i]);
        k++;
    }
    ans.erase(ans.end()-1);
    return ans;
}

inline long long dist(Point a, Point b){//计算距离的平方
    return (a-b).dot(a-b);
}



void rc(vector<Point> ans){
    int tn = ans.size();
    int cnt=0;
    if(tn==2){
        cout<<dist(ans[0],ans[1])<<endl;
        return;
    }
    int i=0,j=0;
    for(int k=0;k<tn;k++){
        if(!cmp(ans[i],ans[k])) i=k;
        if(cmp(ans[j],ans[k])) j=k;
    }
    long long res = 0;
    int si=i,sj=j;
    while(i!=sj||j!=si){
        res = max(res,dist(ans[i],ans[j]));
        if((ans[(i+1)%tn]-ans[i]).det(ans[(j+1)%tn]-ans[j])<0){
            i = (i+1)%tn;
        }else{
            j = (j+1)%tn;
        }
        
        cnt++;
    }
    //返回凸包最远点对的距离的平方
    cout<<res<<endl;
}

int main(){
    cin>>n;
    vector<Point> qs;
    for(int i=0;i<n;i++){
        cin>>po[i].x>>po[i].y;
        //按横纵坐标输入点对
    }
    sort(po,po+n,cmp);
    qs = convexHull();
    rc(qs);
    return 0;
}
```

## 平面最近点对

输入$n$个点的平面坐标，使用分治法计算最近点对，复杂度$O(nlogn)$

```cpp
//复杂度nlogn
//Luogu P1257
#include <iostream>
#include <algorithm>
#include <cmath>

const int MAXN = 200005;

struct Node{
    int x,y;
    int id;

    bool operator<(const Node& a) const {
        if(x!=a.x) return x<a.x;
        return y<a.y;
    }
}nodes[MAXN];

bool cmp(const Node& a, const Node& b){
    return a.y<b.y;
}

double minDist;
int ansA, ansB;

inline void updAns(const Node& a, const Node& b){
    double dist = sqrt((a.x-b.x)*(a.x-b.x)+(a.y-b.y)*(a.y-b.y)+0.0);
    if(dist<minDist){
        minDist = dist;
        //如果要记录节点
        ansA = a.id;
        ansB = b.id;
    }
}

void calcMin(int l, int r){
    if(r-l<=3){
        for(int i=l;i<=r;i++){
            for(int j=i+1;j<=r;j++){
                updAns(nodes[i],nodes[j]);
            }
        }
        std::sort(nodes+l, nodes+r+1, cmp);
        return;
    }

    int m = (l+r)>>1;
    int midx = nodes[m].x;
    calcMin(l,m);
    calcMin(m+1,r);
    
    //归并排序的合并，两个有序数组合并，合并之后仍然有序
    std::inplace_merge(nodes+l, nodes+m+1, nodes+r+1, cmp);

    static Node t[MAXN];
    int tsz = 0;

    for (int i = l; i <= r; ++i){
        if (abs(nodes[i].x - midx) < minDist) {
            for (int j = tsz - 1; j >= 0 && nodes[i].y - t[j].y < minDist; --j)
                updAns(nodes[i], t[j]);
            t[tsz++] = nodes[i];
        }
    }

}

int main(){
    int n;
    std::cin>>n;
    for(int i=1;i<=n;i++){
        std::cin>>nodes[i].x>>nodes[i].y;
        nodes[i].id = i;
    }

    std::sort(nodes+1,nodes+1+n);
    minDist = 1e20;
    calcMin(1,n);

    printf("%.4f\n",minDist);

    return 0;
}
```

## 扫描线算法

```cpp
//Luogu P5490
//复杂度 nlogn
#include <iostream>
#include <algorithm>

using ll = long long;

int const MAXN = 2000005;

struct Line{
    ll l,r,h;
    int tag;
    Line(){}
    Line(ll l, ll r, ll h, int tag):l(l),r(r),h(h),tag(tag){}
    
    bool operator<(Line const & rhs) const{
        return h<rhs.h;
    }  
}line[MAXN*2];

ll st[MAXN*4+2];//对于一颗线段树，n个数所组成的树最多有4n-5个节点，开大了一点
ll posX[MAXN*2];
ll len[MAXN*4+2];

void update(int l, int r, int s, int t, int p, ll c){//c表示加减的数值
    if(posX[t+1]<=l || r<=posX[s]) return;
    if(l<=posX[s] && posX[t+1]<=r){
        st[p] += c;
        if(st[p]){
            len[p] = posX[t+1] - posX[s];
        }
        else{
            len[p] = len[p*2] + len[p*2+1];
        }
        
        return;
    }
    
    int m = s + ((t-s)>>1);
    if(l<=posX[m]) update(l, r, s, m, p*2, c);
    if(r>posX[m])  update(l, r, m+1, t, p*2+1, c);
    if(st[p]){
        len[p] = posX[t+1] - posX[s];
    }
    else{
        len[p] = len[p*2] + len[p*2+1];
    }
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);

	int n;
	std::cin>>n;//矩形个数
	ll x1,x2,y1,y2;
	
	for(int i=1;i<=n;i++){
	    std::cin>>x1>>y1>>x2>>y2;//输入每个矩形的左下角和右上角
	    posX[2*i-1] = x1, posX[2*i] = x2;
	    line[2*i-1] = Line(x1,x2,y1,1), line[2*i] = Line(x1,x2,y2,-1);
	}
	
	n*=2;//方便起见
	
	std::sort(line+1,line+n+1);
	std::sort(posX+1,posX+n+1);
	
	int sumSeg = std::unique(posX+1, posX+1+n) - posX - 1 - 1;//去重求出线段总数
    ll ans = 0;
    
    for(int i=1;i<n;i++){//最后一条边不用管
        update(line[i].l, line[i].r, 1, sumSeg, 1, line[i].tag);
        ans += len[1] * (line[i+1].h - line[i].h);
    }
    
    std::cout<<ans<<"\n";//输出矩形的并集的总面积

    return 0;
}

```

## 二维数点

```cpp
//Luogu P2163
//时间复杂度 nlogn
#include <iostream>
#include <algorithm>

int const MAXN = 500005;

struct Point{
    int x,y;
    int tag;//用于区分实际的点和查询时的虚点  
    
    bool operator<(Point const & p){
        if(x!=p.x) return x<p.x;
        if(y!=p.y) return y<p.y;
        return tag<p.tag;
    }
}pts[MAXN*5];//实点和查询矩阵的点都放在这里面

int b[MAXN*5];
int bit[MAXN];
int ans[MAXN][5];
int tot[MAXN];

inline int lowbit(int n){
    return n&(-n);
}

void update(int p, int k, int n){
    for(;p<=n;p+=lowbit(p)){
        bit[p]+=k;
    }
}

int query(int p){
    int ret=0;
    for(;p;p-=lowbit(p)){
        ret+=bit[p];
    }
    return ret;
}

inline int lsh(int x, int cnt){//离散化函数
    return std::lower_bound(b+1,b+cnt+1,x)-b;
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);

	int n,m;
	std::cin>>n>>m;//点数，查询数

    for(int i=1;i<=n;i++){
        std::cin>>pts[i].x>>pts[i].y;//所有实点的坐标
        pts[i].tag = 0;
    }
    
    for(int i=1;i<=m;i++){
        int x1,x2,y1,y2;
        std::cin>>x1>>y1>>x2>>y2;//查询的长方形的左下角和右上角
        
        pts[++n].x = x1-1, pts[n].y = y1-1, pts[n].tag = i;
        pts[++n].x = x2, pts[n].y = y2, pts[n].tag = i;
        pts[++n].x = x2, pts[n].y = y1-1, pts[n].tag = i;
        pts[++n].x = x1-1, pts[n].y = y2, pts[n].tag = i;
    }
    
    std::sort(pts+1,pts+1+n);
    for(int i=1;i<=n;i++) b[i] = pts[i].y;
    std::sort(b+1,b+1+n);
    int cnt = std::unique(b+1,b+1+n) - b - 1;//把所有y离散化
    
    for(int i=1;i<=n;i++){
        if(pts[i].tag){
            ans[pts[i].tag][++tot[pts[i].tag]] = query(lsh(pts[i].y, cnt));
        }
        else{
            update(lsh(pts[i].y, cnt), 1, cnt);
        }
    }
    
    for(int i=1;i<=m;i++){
        std::cout<<ans[i][4]-ans[i][3]-ans[i][2]+ans[i][1]<<"\n";
    }

    return 0;
}
```

# 组合数学

## 卡特兰数

TODO: 用模板元编程实现编译期算卡特兰数

第$n$个记作$C_n$

$n$对括号形成的字符串，合法的情况数是$C_n$

$n$个节点的二叉树，总共有$C_n$种

$2n+1$个节点组成的满二叉树，有$C_n$种

$n\times n$的格点网中，从左下角格点出发，到达右上角格点，不穿过对角线的单调路径个数有$C_n$个。

其计算公式为

$$
C_n = \frac{1}{n+1}\binom{2n}{n}
$$

$$
C_n=\binom{2n}{n} - \binom{2n}{n-1}
$$

$$
C_0=1,C_{n+1}=\sum^n_{i=0}C_iC_{n-i}
$$

$$
C_0=1,C_{n+1}=\frac{2(2n+1)}{n+2}C_n
$$

```cpp
//复杂度 n
#include <iostream>
//前几项：1（第0项）, 1, 2, 5, 14, 42, 132, 429, 1430, 4862, 16796
using namespace std;

typedef long long ll;
const int MAXN = 3005;

ll h[MAXN];

ll comb(int a,int b){
    ll ans=1;
    for(int i=1;i<=b;i++){
        ans*=a;//数字太大会爆
        a--;
    }
    for(int i=1;i<=b;i++){
        ans/=i;
    }
    return ans;
}

int main(){
    int n;
    cin>>n;
    for(int i=1;i<=n;i++){
        cout<<comb(2*i,i)/(i+1)<<endl;
    }
    cout<<"###"<<endl;
    //下面是递推求法，不容易爆
    h[1]=1;
    cout<<h[1]<<endl;
    for(int i=2;i<=n;i++){
        h[i] = h[i-1]*(4*i-2)/(i+1);
        cout<<h[i]<<endl;
    }

    return 0;
}
```

## 稳定婚姻问题

### Gale-Shapley算法

```cpp
//POJ 3487
#include <iostream>
#include <queue>
#include <algorithm>
#include <cstring>
using namespace std;

const int N   = 30;
const int inf = 1<<29;
const int MOD = 2007;

typedef long long ll;

int couple;
int maleLike[N][N], femaleLike[N][N];
int maleChoice[N],  femaleChoice[N];
int maleName[N],    femaleName[N];
char str[N];
queue<int>freemale;//目前单身的男人

int main(){
    int t;
    scanf("%d",&t);//数据组数
    while(t--){
        scanf("%d",&couple);//男女对数
        while(!freemale.empty()){
            freemale.pop();
        }
        for(int i=0;i<couple;i++){
            scanf("%s",str);
            maleName[i]=str[0]-'a';//题目中是以小写字母给男人名字，转化为数字
            freemale.push(maleName[i]);
        }
        sort(maleName, maleName+couple);//名字排序，便于字典序

        for(int i=0;i<couple;i++){
            scanf("%s",str);
            femaleName[i]=str[0]-'A';//女人名字是大写字母
        }
        
        for(int i=0;i<couple;i++){
            scanf("%s",str);
            for(int j=0;j<couple;j++){
                maleLike[i][j]=str[j+2]-'A';//男人喜好顺序由男人名字:女人名字列表给出;降序排列
            }
        }
        
        //女士对男士的打分，添加虚拟人物，编号couple，为女士的初始对象
        for(int i=0;i<couple;i++){
            scanf("%s",str);
            for(int j=0;j<couple;j++){
                femaleLike[i][str[j+2]-'a']=couple-j;//排名越前打分越高
            }
            femaleLike[i][couple]=0;
        }
        memset(maleChoice,0,sizeof(maleChoice));
        //一开始男士的期望都是最喜欢的女士

        for(int i=0;i<couple;i++){
            femaleChoice[i]=couple;
        }
        
        while(!freemale.empty()){
            int male=freemale.front();
            //找出未配对的男士
            int female=maleLike[male][maleChoice[male]];
            //找出心意的女士
            if(femaleLike[female][male]>femaleLike[female][femaleChoice[female]]){
            //比现男友好
                freemale.pop();
                if(femaleChoice[female]!=couple){
                //前男友再次单身，并且不能将虚拟人物加入队列
                    freemale.push(femaleChoice[female]);
                    maleChoice[femaleChoice[female]]++;
                }
                femaleChoice[female]=male;
				//更换男友
            }
            else
                maleChoice[male]++;
                //如果被拒绝，则选择下一位
        }
        for(int i=0;i<couple;i++){
            printf("%c %c\n",maleName[i]+'a', maleLike[maleName[i]][maleChoice[maleName[i]]]+'A');
        }
        if(t) puts("");

    }

    return 0;
}
```

# 树

## 树状数组

```cpp
//复杂度 单次查询 logn 单次修改 logn
//树状数组，维护的是数组的前缀和，有大量的应用
//luogu P3374

#include <iostream>
#include <cstdio>
#define MAXN 500005

using namespace std;

int arr[MAXN];
int bit[MAXN];

inline int lowbit(int n){
    return n&(-n);
}

void update(int p, int k, int n){
    for(;p<=n;p+=lowbit(p)){
        bit[p]+=k;
    }
}

int query(int p){
    int ans=0;
    for(;p;p-=lowbit(p)){
        ans+=bit[p];
    }
    return ans;
}

int main(){
	int n,m;
    scanf("%d%d",&n,&m);
    //数组长度，查询数
    for(int i=1;i<=n;i++){
        scanf("%d",&arr[i]);
        update(i,arr[i],n);
    }

    for(int i=1;i<=m;i++){
        int op;
        int x,y,k;
        scanf("%d",&op);
        scanf("%d",&x);
        if(op==1){
            scanf("%d",&k);
            //将单点增加k，如果想要改成修改，则可以update(x,-arr[x]+k)
            update(x,k,n);
        }
        else{
            scanf("%d",&y);
            //输出[x,y]的数组和
            cout<<query(y)-query(x-1)<<endl;
        }
    }
    return 0;
}
```

### 树状数组求逆序对

```cpp
//复杂度 nlogn
//Luogu P1908
#include <iostream>
#include <algorithm>

#define LL long long

const int MAXN = 500005;

LL arr[MAXN];

struct Par{
    LL value,id;
}par[MAXN];

bool cmp(const Par& a,const Par& b){
    if(a.value!=b.value) return a.value<b.value;
    return a.id<b.id;
}

LL bit[MAXN];

inline LL lowbit(LL n){
    return n&(-n);
}

void update(LL p, LL k, LL n){
    for(;p<=n;p+=lowbit(p)){
        bit[p]+=k;
    }
}

long long query(LL p){
    LL ans=0;
    for(;p;p-=lowbit(p)){
        ans+=bit[p];
    }
    return ans;
}

int main(){
    LL n;
    std::cin>>n;

    for(int i=1;i<=n;i++){
        std::cin>>par[i].value;
        par[i].id = i;
    }

    std::sort(par+1,par+1+n,cmp);

    for(int i=1;i<=n;i++){
        arr[par[i].id] = i;
    }//这一步其实是离散化

    LL ans = 0;

    for(int i=1;i<=n;i++){
        ans += query(arr[i]);
        update(arr[i],1,n);
    }

    ans = n*(n-1)/2-ans;//本来统计的是等于或顺序对，现在反过来计算逆序对

    std::cout<<ans<<"\n";

    return 0;
}
```

## 并查集

```cpp
//复杂度 很小
//并查集 Luogu3367
const int MAXN = 10005;

class DSU{
public:
    int fa[MAXN], rk[MAXN];
    
    void init(int n){
        for(int i=1;i<=n;i++) fa[i] = i, rk[i] = 1;
    }
    
    int find(int x){
        //没有路径压缩的find，在需要删除操作时，不能使用路径压缩，只能按秩合并保证复杂度
        return fa[x]==x ? x : find(fa[x]);
    }
    
    int findc(int x){
        //带路径压缩的find
        return fa[x]==x ? x : (fa[x] = findc(fa[x]));
    }
    
    void merge(int x, int y){
        //按秩合并，如果不需要则直接 fa[find(x)] = find(y);
        x = find(x), y = find(y);
        if(x==y) return;
        if(rk[x]>rk[y]) std::swap(x,y);
        fa[x] = y;
        if(rk[x]==rk[y]) rk[y]++;
    }
    
    void mergec(int x, int y){
        //按秩合并+路径压缩，如果不需要则直接 fa[findc(x)] = findc(y);
        x = findc(x), y = findc(y);
        if(x==y) return;
        if(rk[x]>rk[y]) std::swap(x,y);
        fa[y] = x;
        if(rk[x]==rk[y]) rk[y]++;
    }
    
    void erase(int x){
        --rk[find(x)];
        fa[x] = x;
    }
};
```

## 线段树

```cpp
//复杂度 单次查询 logn 单次修改 logn
//luogu 3372
//线段树维护的数据要求满足结合律，比如区间和，区间最大区间最小，区间gcd
//区间修改一般支持加、乘、赋值
int const MAXN = 100005;
using LL = long long;

struct Node
{
    int s,t;//该端点的起点和终点下标
    LL tag, v;
};

Node st[MAXN*4+2];
LL arr[MAXN];

void build(int s, int t, int p){
    st[p].s = s;
    st[p].t = t;
    if(s==t) {
        st[p].v = arr[s];
        st[p].tag = 0;
        return;
    }
    int m = s+((t-s)>>1);
    build(s,m,p*2);
    build(m+1,t,p*2+1);
    st[p].v = st[p*2].v + st[p*2+1].v;
    st[p].tag = 0;
}

void spreadTag(int p){
    if(st[p].tag){
        int s = st[p].s, t = st[p].t;
        int m = s+((t-s)>>1);
        st[p*2].v     += (m-s+1)*st[p].tag;
        st[p*2+1].v   += (t-m)*st[p].tag;
        st[p*2].tag   += st[p].tag;
        st[p*2+1].tag += st[p].tag;
        st[p].tag=0;
    }
}

void update(int l, int r, int p, LL k){
    int s = st[p].s, t = st[p].t;
    if(l<=s && t<=r){
        st[p].v   += (t-s+1) * k;
        st[p].tag += k;
        return;
    }
    spreadTag(p);
    
    int m = s+((t-s)>>1);
    if(l<=m) update(l, r, p*2, k);
    if(r>m)  update(l, r, p*2+1, k);
    st[p].v = st[p*2].v + st[p*2+1].v;
}

LL query(int l, int r, int p){
    int s = st[p].s, t = st[p].t;
    if(l<=s && t<=r) return st[p].v;
    
    spreadTag(p);
    int m = s+((t-s)>>1);
    LL ret = 0;
    if(l<=m) ret+=query(l,r,p*2);
    if(r>m)  ret+=query(l,r,p*2+1);
    
    return ret;
}
```

## 珂朵莉树

```CPP
//珂朵莉树，区间推平问题
//方便给某个区间赋值，区间加数，维护区间第k大值，区间和等等
//数据随机的情况下，复杂度为nloglogn
//珂朵莉树的每一个节点都是一个区间，这个区间内的值相同。
struct Node{
    int l,r;
    mutable LL v;//这里修改成自己需要的数据类型，在[l,r]内都等于这个值
    Node(int l, int r, LL v):l(l),r(r),v(v){}
    bool operator<(Node const & x) const {return l<x.l;}
};

class ODT{
public:
    std::set<Node> tree;
    
    auto split(int pos){
        auto it = tree.lower_bound(Node(pos,0,0));
        
        if(it!=tree.end() && it->l==pos)
            return it;
        it--;
        int l = it->l, r = it->r;
        LL v = it->v;
        tree.erase(it);
        tree.insert(Node(l,pos-1,v));
        return tree.insert(Node(pos,r,v)).first;
    }
    
    void assign(int l, int r, int v){
        //给区间赋值
        auto end = split(r+1), begin = split(l);//必须要注意顺序
        tree.erase(begin,end);
        tree.insert(Node(l,r,v));
    }
    
    void perf(int l, int r){//其他操作的模板函数
        auto end = split(r+1), begin = split(l);
        for(auto it=begin;it!=end;it++){
            //这里是操作
            //这些操作都很暴力，例如k大值，就把区间全部枚举排序一遍去找
            //例如区间和，就枚举区间加起来，注意是加it->v * (it->r-it->l+1)
            //例如区间加数，就枚举区间给所有的it->v都加一个数
        }
    }
};

//珂朵莉树的初始化不能用assign，设范围为[1,w]，初值全部为0，则
ODT odt;
odt.tree.insert(Node(1,w,0));
```

## 分块

分块是根号算法，比线段树略差，但是不需要满足结合律，也不需要传递tag。

```cpp
//luogu 3372 和线段树区间加，维护区间和一样
//复杂度n sqrt(n)
//在块内时对块操作，跨块时中间对块操作，两边多余部分暴力处理
LL arr[MAXN];

class BA{
public:
    int st[MAXN],ed[MAXN],size[MAXN],bel[MAXN];//每一段的开始下标、结束下标、段大小；每个元素属于哪个段
    int sq;
    LL sum[MAXN];//保存第i个块的和
    LL tag[MAXN];
    
    void init(int n){
        sq = std::sqrt(n);
        for(int i=1;i<=sq;i++){
            st[i] = n / sq * (i-1) + 1;
            ed[i] = n / sq * i;
            size[i] = ed[i] - st[i] + 1;
        }
        ed[sq] = n;//最后一段可能长度不够n/sq
        size[sq] = ed[sq] - st[sq] + 1;
        
        for(int i=1;i<=sq;i++)
            for(int j=st[i];j<=ed[i];j++)
                bel[j] = i, sum[i] += arr[j];
    }
    
    void update(int l, int r, LL k){
        if(bel[l]==bel[r]){
            for(int i=l;i<=r;i++){
                arr[i]+=k;
                sum[bel[i]] += k;
            }
            return;
        }
        
        for(int i=l;i<=ed[bel[l]];i++){
            arr[i]+=k;
            sum[bel[i]]+=k;
        }
        for(int i=st[bel[r]];i<=r;i++){
            arr[i]+=k;
            sum[bel[i]]+=k;
        }
        for(int i=bel[l]+1;i<bel[r];i++){
            tag[i] += k;
        }
    }
    
    LL query(int l, int r){
        LL ret = 0;
        if(bel[l]==bel[r]){
            for(int i=l;i<=r;i++)
                ret += arr[i] + tag[bel[i]];
            return ret;
        }
        
        for(int i=l;i<=ed[bel[l]];i++)
            ret += arr[i] + tag[bel[i]];
        for(int i=st[bel[r]];i<=r;i++)
            ret += arr[i] + tag[bel[i]]; 
        for(int i=bel[l]+1;i<bel[r];i++)
            ret += sum[i] + tag[i] * size[i];
        return ret;
    }
};

BA ba;//注意，开了大数组，要声明在main函数外面，或者可以去用动态分配内存
```

## 平衡树（pbds实现）

平衡树在ACM中用的极少，就不手搓了，大部分情况下都可以用set和pbds搞定。

```cpp
#include<ext/pb_ds/assoc_container.hpp>
#include<ext/pb_ds/tree_policy.hpp> //仅限g++可以使用

namespace pbds = __gnu_pbds;

pbds::tree<LL, pbds::null_type, std::less<LL>, 
           pbds::rb_tree_tag, pbds::tree_order_statistics_node_update> tr;
```

声明如上，还是挺复杂的，但是ACM可以带资料所以不成问题。需要注意只能在g++上用，ACM赛场大多都有g++所以不是问题。

**模板参数解释**

LL是存储数据的类型；

pbds::null_type是映射规则（低版本g++为pbds::null_mapped_type，如果存入类型为std::map\<Key,Value\>则要填入Value）；

std::less\<LL\>则是我们选择大根还是小根；可选参数，默认为less

pbds::rb_tree_tag 则是我们选择的树的类型。总共有三种平衡树在pbds里，红黑树、splay、ov，但是后两个容易超时，一般不用。可选参数，默认为红黑树

pbds::tree_order_statistics_node_update是节点更新方法，如果使用order_of_key和find_by_key方法，则要用它。可选参数，但默认是null_node_update。

**方法**

```cpp
tr.insert(x); //插入一个元素x，返回std::pair<point_iterator, bool>
//若成功，则是插入之后的迭代器和true，否则是x的迭代器和false
tr.erase(x);  //成功返回true，也可以把迭代器作为参数
tr.order_of_key(x);  //返回x的排名，0为第一名，x不一定要在树里
tr.find_by_order(k); //返回排名为k的元素的迭代器，0为第一名
tr.lower_bound(x);   //返回迭代器，这个函数不用多说了吧，和经常见到的一样
tr.upper_bound(x);   //返回迭代器
tr.join(b);          //将b树并入当前树，两棵树的类型要一样，不能有重复元素，b树将会被删除
tr.split(x,b);       //小于等于x的保留在当前树，其他分给b树
tr.empty();
tr.size();
```

以上操作均为O(logn)复杂度，除了最后两个是O(1)

**注意事项**

tree里面的元素是唯一的，有点类似与set。但我们并没有multi-tree去使用，做例如洛谷上的平衡树模板题，他要求元素可重复。此时我们有以下奇技淫巧

```cpp
LL n;
std::cin>>n;

for(int i=1;i<=n;i++){
	int ope;
	LL x;
	std::cin>>ope>>x;
	
	if(ope==1) tr.insert((x<<20)+i);
	else if(ope==2) tr.erase(tr.lower_bound(x<<20));
	else if(ope==3) std::cout<<tr.order_of_key(x<<20)+1<<"\n";
	else if(ope==4) std::cout<<((*tr.find_by_order(x-1))>>20)<<"\n";
	else if(ope==5){
	    auto it = tr.lower_bound(x<<20);
	    it--;
	    std::cout<<((*it)>>20)<<"\n";
	}
	else{
	    auto it = tr.upper_bound((x<<20)+n);
    	std::cout<<((*it)>>20)<<"\n";
	}
}
```

假设有$n$个操作，共$6$种

1. 插入$x$
2. 删除$x$
3. 查询$x$的排名（比$x$小的数的个数$+1$）
4. 查询排名为$x$的数
5. 求小于$x$的最大的数
6. 求大于$x$的最小的数

看代码，我们将$x$左移20位，加上了操作序号，这样我们就可以实现可重复插入。只需要我们再最后把数字右移20位回来即可。erase注意是加入迭代器去erase，因为我们并没有等于x\<\<20的数字。最后一个操作，要$+n$，来处理所有的相等的$x$

# 堆

## 对顶堆

```cpp
//维护一个序列的第k大数，每次操作logn

#include <iostream>
#include <algorithm>
#include <queue>
#include <vector>

template<typename T>
class KthLargest{
private:
    std::priority_queue<T,std::vector<T>,std::less<T> > big{};
    std::priority_queue<T,std::vector<T>,std::greater<T> > small{};
    int kth{};
    int size{};
    
    void update(){
        kth = std::min(kth,size);
        while(kth<small.size()){
            big.push(small.top());
            small.pop();
        }
        while(kth>small.size()){
            small.push(big.top());
            big.pop();
        }
    }
    
public:
    KthLargest():kth(1),size(0){}
    
    T findK(int k){
        kth = k;
        update();
        return small.top();
    }
    
    void eraseK(int k){
        kth = k;
        update();
        small.pop();
        size--;
        update();
    }
    
    void insert(T x){
        size++;
        if(small.empty() || x>=small.top()){
            small.push(x);
        }
        else{
            big.push(x);
        }
        update();
    }
    
    int getSize(){
        return size;
    }
};
```

# 单调数据结构

## 单调栈

给出模板，以下是一个维护栈内单调不增的代码

```cpp
int arr[MAXN];
int stk[MAXN];

int top = 0;

for(int i=1;i<=n;i++){
    while(top&&arr[stk[top]]<arr[i]){
        //在这里进行一些操作
        top--;
    }

    stk[++top] = i;
}
```

## 单调队列

给出滑动窗口的模板，单调队列通常会使用双端队列。滑动窗口即给出一个长度为$n$的数组，以及一个长度为$k$的窗口，从左向右滑动，求出窗口中的最小值和最大值

```cpp
int arr[MAXN];

int main(){
    int n,k;
    std::cin>>n>>k;

    for(int i=1;i<=n;i++){
        std::cin>>arr[i];
    }

    std::deque<int> dq;

    for(int i=1;i<k;i++){
        while(!dq.empty()&&arr[dq.back()]>=arr[i]) dq.pop_back();
        dq.push_back(i);
    }

    for(int i=k;i<=n;i++){
        while(!dq.empty()&&arr[dq.back()]>=arr[i]) dq.pop_back();
        dq.push_back(i);
        while(dq.front()<=i-k) dq.pop_front();
        std::cout<<arr[dq.front()]<<" ";
    }

    std::cout<<"\n";

    dq.clear();

    for(int i=1;i<k;i++){
        while(!dq.empty()&&arr[dq.back()]<=arr[i]) dq.pop_back();
        dq.push_back(i);
    }

    for(int i=k;i<=n;i++){
        while(!dq.empty()&&arr[dq.back()]<=arr[i]) dq.pop_back();
        dq.push_back(i);
        while(dq.front()<=i-k) dq.pop_front();
        std::cout<<arr[dq.front()]<<" ";
    }

    return 0;
}
```

# 倍增

## ST表

对于经典的RMQ（即给定一个数组，求区间内的最大值）问题，有如下代码

```cpp
//复杂度 单次查询 logn 预处理 nlogn
//luogu P3865
#include <cstdio>
#include <iostream>

const int MAXN = 100005;
const int LOGN = 21;

int fmax[MAXN][LOGN+1];
//fmax[a][b]表示[a,a+2^b-1]中的最大值
int logn[MAXN];
//预先计算logn

int main(){
    int n,m;
    //数组大小以及查询次数
    scanf("%d%d",&n,&m);

    for(int i=1;i<=n;i++){
        scanf("%d",&fmax[i][0]);
    }

    logn[1] = 0;
    logn[2] = 1;

    for(int i=3;i<MAXN;i++){
        logn[i] = logn[i/2]+1;
        //预先计算logn
    }

    for(int j=1;j<=LOGN;j++){
        for(int i=1;i+(1<<j)-1<=n;i++){
            fmax[i][j] = std::max(fmax[i][j-1],fmax[i+(1<<(j-1))][j-1]);
        }
    }

    for(int i=1;i<=m;i++){
        int a,b;
        scanf("%d%d",&a,&b);
        //查询[a,b]分为两部分，即[a,a+2^s-1]与[b-2^s+1,b]
        int s = logn[b-a+1];
        printf("%d\n",std::max(fmax[a][s],fmax[b-(1<<s)+1][s]));
    }

    return 0;
}
```

## 倍增求最近公共祖先

```cpp
//复杂度 单次查询 logn 预处理 nlogn
//luogu P3379
#include <iostream>
#include <vector>
#include <cstdio>

const int MAXN = 500005;
const int LOGN = 31;

std::vector<int> edge[MAXN];//邻接表
int fa[MAXN][LOGN],deep[MAXN];
//fa[a][b]代表a的第2^b个祖先，deep是深度，根节点深度为1

void build(int v,int father){
    fa[v][0] = father;
    deep[v] = deep[father]+1;

    for(int i=1;i<LOGN;i++){
        fa[v][i] = fa[fa[v][i-1]][i-1];
    }

    for(auto v1:edge[v]){
        if(v1==father) continue;
        build(v1,v);
    }
}

int lca(int x,int y){
    if(deep[x]>deep[y]) std::swap(x,y);
    //保证y比x深

    int tmp = deep[y]-deep[x];
    for(int i=0;tmp;i++,tmp>>=1){
        if(tmp&1) y=fa[y][i];
    }

    if(x==y) return y;

    for(int i=LOGN-1;i>=0&&y!=x;i--){
        if(fa[x][i]!=fa[y][i]){
            x = fa[x][i];
            y = fa[y][i];
        }
    }

    return fa[y][0];
}

int main(){
    int n,m,s;
    scanf("%d%d%d",&n,&m,&s);
    //点数，询问数，根节点序号

    for(int i=1;i<=n-1;i++){
        int a,b;
        scanf("%d%d",&a,&b);
        //读入树
        edge[a].push_back(b);
        edge[b].push_back(a);
    }

    build(s,0);

    for(int i=1;i<=m;i++){
        int x,y;
        scanf("%d%d",&x,&y);
        //查询x,y的最近公共祖先
        printf("%d\n",lca(x,y));
    }

    return 0;
}
```

# 二分

## 二分答案

给出一个通用代码

```cpp
int l = 0;
int r = MAXR;

while(l+1<r){
    int mid = (l+r)>>1;
    if(judge(mid)) l=mid;
    else r = mid;
}

if(judge(l)) std::cout<<l<<"\n";
else std::cout<<r<<"\n";
```
judge函数应该根据题意写出。

如果是浮点数的二分，则不推荐使用EPS进行精度判断（有可能会丢精度）。而是使用计数器，一般迭代100次就能保证符合题目要求。复杂度显然logn

# 动态规划

## 01背包

```cpp
//复杂度 nW
//luogu P1048
#include <iostream>
#include <cmath>

int const MAXN = 1005;

using namespace std;

int dp[MAXN];
int w[MAXN];
int v[MAXN];

int main(){
    int n,W;//物品数，背包大小
    cin>>W>>n;
    for(int i=1;i<=n;i++){
        cin>>w[i]>>v[i];//物品体积，物品价值
    }
    for(int i=1;i<=n;i++){
        for(int j=W;j>=w[i];j--){
			dp[j] = std::max(dp[j],dp[j-w[i]]+v[i]);
        }
    }
    cout<<dp[W]<<endl;
    return 0;
}
```

## 完全背包

```cpp
//复杂度 nW
//luogu P1616
#include <iostream>
#include <cmath>

int const MAXN = 10007;
int const MAXW = 10000007;
typedef long long LL;

using namespace std;

LL dp[MAXW];
int w[MAXN];
int v[MAXN];

int main(){
    int n,W;//物品数，背包大小
    cin>>W>>n;
    for(int i=1;i<=n;i++){
        cin>>w[i]>>v[i];//物品体积，物品价值
    }
    for(int i=1;i<=n;i++){
        for(int j=w[i];j<=W;j++){
			dp[j] = std::max(dp[j],dp[j-w[i]]+v[i]);
        }
    }
    cout<<dp[W]<<endl;
    return 0;
}
```

## 多重背包

```cpp
//复杂度 Wsum(logk_i) 
//luogu P1776
#include <iostream>
#include <cmath>

int const MAXN = 100007;
int const MAXW = 40007;
typedef long long LL;

using namespace std;

LL dp[MAXW];
int w[MAXN];
int v[MAXN];

int main(){
    int m,W;//物品种类数，背包大小
    cin>>m>>W;
    int n = 0;
    for(int i=1;i<=m;i++){
        int c = 1;
        int p,h,k;//物品价值，物品体积，物品数量
        cin>>p>>h>>k;
        while(k>c){
            k -= c;
            v[++n] = c*p;
            w[n] = c*h;
            c *= 2;
        }
        v[++n] = p*k;
        w[n] = h*k;
    }
    for(int i=1;i<=n;i++){
	    for(int j=W;j>=w[i];j--){
	        dp[j] = std::max(dp[j],dp[j-w[i]]+v[i]);
        }
    }
    cout<<dp[W]<<endl;
    return 0;
}
```

# 概率论

## 处理分数期望、概率

有时候，题目中的期望是一个分数$\frac{P}{Q}$，而为了防止精度问题，往往会要求输出一个$R$，满足

$$
R\times Q\equiv P(mod\ 998244353)
$$

此时

$$
R = (P\times Q^{-1})\%998244353
$$

$Q^{-1}$是$Q$在模$998244353$意义下的乘法逆元

# 杂项

## 快速幂

```cpp
//复杂度logn
#include <iostream>

using namespace std;

long long binpow(long long n, long long p){
    long long res = 1;
    while(p>0){
        if(p&1){
            res = res * n;
        }
        n *= n;
        p>>=1;
    }
    return res;

}


int main(){
    long long n,p;
    cin>>n>>p;

    cout<<binpow(n,p)<<endl;

    return 0;
}
```

## 离散化

有两种，一种是unique函数版，一种是树状数组求逆序对里使用的，都可以，区别是，那个对于相同的数字根据先后顺序确定大小，这个则是一样大

```cpp
//复杂度nlogn
//离散化 例如将1,500,40,1000保持相对大小不变，离散化为1,3,2,4
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

vector<int> arr,assi;

int main(){
    int n;
    cin>>n;
    for(int i=1;i<=n;i++){
        int a;
        cin>>a;
        arr.push_back(a);
        assi.push_back(a);
    }
    sort(assi.begin(),assi.end());
    assi.erase(unique(assi.begin(),assi.end()),assi.end());

    for(int i=0;i<n;i++){
        arr[i] = upper_bound(assi.begin(),assi.end(),arr[i])-assi.begin();
    }

    for(int i=0;i<n;i++){
        cout<<arr[i]<<" ";
    }
    cout<<endl;

    return 0;
}
```

## 莫队算法

```cpp
//对于序列上的区域离线询问问题，如果[l,r]的答案能够O(1)拓展得到
//[l-1,r],[l+1,r],[l,r-1],[l,r+1]的答案，那么就可以在O(n sqrt(n))中解决所有询问
//SPOJ DQUERY
int nowAns, ans[MAXQ];
int sq;//分块数sq = sqrt(n)

struct Query{
    int l,r,id;//询问区间和询问下标
    bool operator<(Query const & x)const{
        if(l/sq != x.l/sq)//根据归属于哪个块排序
            return l<x.l;
        if(l/sq & 1)      //玄学奇偶排序
            return r<x.r;
        return r>x.r;
    }  
}Q[MAXQ];

int l=1,r=0;//初始化询问区间

inline void update(int p){
    //update here
}

void solve(){
    sq = std::sqrt(n);
    //输入数据（总数n个数据）、查询（总数q个查询）
    std::sort(Q,Q+q);
    for(int i=0;i<q;i++){
        while(l>Q[i].l)
	        update(--l);
	    while(r<Q[i].r)
	        update(++r);
	    while(l<Q[i].l)
	        update(l++);
	    while(r>Q[i].r)
	        update(r--);
	    //注意上述顺序，扩张区间是先移动再更新，缩减区间是先更新再移动
	    //四个update可能会有不一样，具体题目具体讨论
	    ans[Q[i].id] = nowAns;
    }
}

```

## 表达式求值

```cpp
inline bool isOper(char c){
    return c=='+'||c=='-'||c=='*'||c=='/';
}

inline bool isDigit(char c){
    return c>='0' && c<='9';
}

inline int priority(char oper){
    if(oper=='+' || oper=='-') return 1;
    if(oper=='*' || oper=='/') return 2;
    return -1;
}

std::string toRPN(std::string expr){
    std::string ret;
    
    std::stack<char> oper;
    int esize = expr.size();
    for(int i=0;i<esize;i++){
        char& c = expr[i];
        if(c==' ') continue;
        else if(isOper(c)){
            if(c=='-' && (i==0 || expr[i-1]=='(')){
                //判断一元运算符负号，这里采用了加个0-前缀的方法，如果题目要求输出RPN其实是做不到的
                //TODO: 把toRPN返回一个vector，实现真正的RPN
                ret.push_back('0');
                ret.push_back(' ');
            }
            while(!oper.empty() && priority(oper.top())>=priority(c)){
                //如果是右结合运算符，则要改成大于，如果只有一部分是右结合运算符，分类讨论
                ret.push_back(oper.top());
                ret.push_back(' ');
                oper.pop();
            }
            oper.push(c);
        }
        else if(c=='('){
            oper.push(c);
        }
        else if(c==')'){
            while(oper.top()!='('){
                ret.push_back(oper.top());
                ret.push_back(' ');
                oper.pop();
            }
            oper.pop();
        }
        else{
            while(i<esize && isDigit(expr[i])){
                ret.push_back(expr[i++]);
            }
            ret.push_back(' ');
            i--;
        }
    }
    
    while(!oper.empty()){
        ret.push_back(oper.top());
        ret.push_back(' ');
        oper.pop();
    }
    
    return ret;
}

void processOper(std::stack<int> & st, char oper){
    int r = st.top();
    st.pop();
    int l = st.top();
    st.pop();
    
    switch(oper){
        case '+':
            st.push(l+r);
            break;
        case '-':
            st.push(l-r);
            break;
        case '*':
            st.push(l*r);
            break;
        case '/':
            st.push(l/r);
            break;
    }
}

int RPNCalc(std::string expr){
    int ret = 0;

    std::stack<int> number;
    int esize = expr.size();
    for(int i=0;i<esize;i++){
        char& c = expr[i];
        if(c==' ') continue;
        else if(isOper(c)){
            processOper(number, c);
        }
        else{
            int res = 0;
            while(i<esize && isDigit(expr[i])){
                res = res*10 + expr[i++] - '0';
            }
            i--;
            number.push(res);
        }
    }
    
    ret = number.top();

    return ret;
}

int exprCalc(std::string expr){
    int ret = RPNCalc(toRPN(expr));
    return ret;
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);

	std::string str;
	std::cin>>str;
	
	std::cout<<toRPN(str)<<"\n";
	std::cout<<exprCalc(str)<<"\n";

    return 0;
}
```

# C++ STL用法

## std::swap

交换两个元素的内容（也可以交换数组，不重要不介绍）。复杂度：常数。

```cpp
int a,b;
std::swap(a,b);
```

注意其中的两个参数，类型要相同。不能一个是LL一个是int。

## std::sort

对数组、vector等进行排序。复杂度nlogn。

```cpp
int a[5];
std::vector<int> vec(5);
std::sort(a,a+5);
std::sort(vec.begin(),vec.end());
```

注意排序范围是左闭右开区间。

通常会按照类型的<操作符来进行比较。如果对结构体进行排序，可以重载运算符或者设定cmp函数。注意这两种方法一定不能是小于等于或者大于等于的运算，必须只使用小于号或者大于号（严格弱序）。

```cpp
bool cmp(const Type1& a, const Type2& b);
//然后在sort中加入第三个参数
std::sort(a,a+5,cmp);

struct node{
    bool operator<(const node& a);
};
//不用添加cmp参数
```

## std::lower_bound,std::upper_bound

对某个已经排序好的数组，查找第一个大于等于（lower_bound）或者大于(upper_bound)某个给定值的元素。复杂度：logn（对于随机访问的迭代器），n（对于其他迭代器）。

```cpp
int a[5]={0,1,3,4,6};
int first = std::lower_bound(a,a+5,3);
```

同样是左闭右开区间，第三个参数是指定的值。如果找到就会返回所查找元素的迭代器（或者指针）。找不到就会返回末尾元素的后一个指针（或者end迭代器）。

如果需要自定义比较方法，同sort函数。

## std::max,std::min

对于两个元素返回最大值和最小值。复杂度：准确一次比较。

```cpp
int a=1,b=2;
int maxv = std::max(a,b);
int minv = std::min(a,b);
```

同样，两个参数类型相同。自定义比较方法同sort。

## std::abs

计算绝对值。复杂度：文档没写但应该是常数。

```cpp
int a = -1;
int b = std::abs(a);
```

注意，函数只有float,double,long double的返回值类型。使用时如果给予整数参数会自动转换，这是否会导致精度问题有待观察。

## std::string

### ::swap

将两个字符串互换。复杂度：常数。

```cpp
std::string str = "123456";
std::string str2 = "456789";
str.swap(str2);
```

### ::begin,std::end

返回字符串的起始得带器和结尾迭代器。

```cpp
str.begin();str.end();
```

### ::size

返回字符串的大小。

```cpp
str.size();
```

### ::push_back

向字符串末尾添加一个字符，同时大小加一。复杂度：常数。

```cpp
str.push_back('a');
```

### ::pop_back

将字符串末尾的字符弹出，同时大小减一。如果字符串为空则未定义。复杂度：常数。

```cpp
str.pop_back();
```

### ::find

在字符串中寻找某个子串是否存在。复杂度：没有规定，编译器不一定都是使用的kmp算法。

```cpp
std::string::size_type n;
std::string s = "this is a string";

n = s.find("is");
```

如果找到则返回首个匹配的首字母位置。否则返回std::string::npos。如果是int n作为s.find的接收端，则会在找不到时接收到-1。

### ::replace

将字符串的某个片段替换为另一个字符串

```CPP
std::string s = "abcd efgh";
s.replace(1,3,"aaaa");
```

第一个参数是开始位置的下标，第二个参数是指从开始位置有几个字符，第三个参数是将要替换进去的字符串，上式结果是"aaaaa efgh"。

### ::substr

获取自字符串

```CPP
std::string s = "abcd efgh";
std::string b = s.substr(1,3);
```

从下标1开始的3个字符，即b="bcd"。

## std::memset

将值复制到dest所指对象的前count个字节中。复杂度：没有规定。

```cpp
int a[20];
std::memset(a,0,sizeof(a));
```

注意，赋的值不能随便取，这个函数是一个字节一个字节地去赋值的。如果取1并不会得到全部赋值为1的效果，通常只会取0和-1。

## std::map

map是有序键值对容器，通常用红黑树实现。元素的键是唯一的。

```cpp
template<
    class Key,
    class T,
    class Compare = std::less<Key>,
    class Allocator = std::allocator<std::pair<const Key, T> >
> class map;
```

可以通过迭代器来遍历

```cpp
for(std::map<int,int>::iterator it = mp.begin();it!=mp.end();it++);
//访问元素用it->first和it->second
//或者
for(auto it:mp);
//访问元素用it.first和it.second
```

### 自定义比较函数

map通常会按照key的大小关系进行升序排列。如果要自定义比较函数，则

```cpp
struct cmp{
    bool operator()(const int& a, const int& b) const{
        return a>b;
    }
};

std::map<int,std::string,cmp> mp;
```

### ::empty

检测是否为空。

### ::size

返回大小。

### ::clear

清除所有内容。复杂度：线性。

### ::erase

提供迭代器，删除迭代器所指的键值对。复杂度：常数。

```cpp
auto it = mp.begin();
mp.erase(it);
```

当然也可以提供两个迭代器，删除这之间的所有元素（左闭右开区间）

### ::find

寻找key等于给定值的元素，返回迭代器。如果没有找到则返回end迭代器。复杂度：对数。

```cpp
auto it = mp.find(1);
```

### ::lower_bound,::upper_bound

寻找首个大于等于(或大于，对upper_bound)给定值的key。复杂度：对数。

```cpp
auto it = mp.lower_bound(1);
```

## std::unordered_map

可以看作是无序的map，通常由哈希表实现。这意味着map中和排序有关的函数都不能使用。

**警告**

unordered_map可能不能用auto x:mp或者迭代器遍历。它的遍历可能是遍历bucket，而不是遍历元素。但是只是查找是可以的。

## std::set 

set是关联容器，含有Key类型对象的已排序集，通常用红黑树实现。

```cpp
template<

    class Key,
    class Compare = std::less<Key>,
    class Allocator = std::allocator<Key>
> class set;
```

遍历的方式与map相同。

### 使用方法

自定义比较函数,empty,size,clear,erase,find,lower_bound,upper_bound都与map相同。

### ::insert

map可以用\[\]进行插入，但是set只能用insert函数。

```cpp
std::set<int> st;
st.insert(1);
```

返回值是一个pair，first是迭代器，指向被插入进去的元素（如果插入不成功则指向没插进去的元素），second是bool，插入成功时为true，否则为false。

### 修改内部元素

有时候，我们需要修改内部元素。由于例如begin函数返回的迭代器是const的，我们无法直接修改数据。但我们可以给变量添加mutable修饰。这样我们就可以不用拿出来一个元素再插回去。例子可见珂朵莉树。

注意，如果你修改的数据和排序依据有关，set不会维护内部有序。要维护还得是拿出来再插入进去。

## std::unordered_set

用哈希实现，没有内部排序。

**警告**

可能跟unordered_map一样不能遍历。

## std::multiset

与普通的set不同的是，可以插入多个相同的元素。这在一些情况下是有用的，而且它还是满足内部有序。

可以用::count(x)来统计Key为x的元素的数量，普通的set也有这个方法，但是要么是0要么是1，与find功能可以说是重复。但是multiset可以统计数量。

注意，用erase方法时，如果给出的是一个值，那么会把等于这个值的元素都删掉，如果给出迭代器，那么之后删一个。所以，只想删一个等于这个值的元素时，用erase(find(...))

## std::stack

栈，有先入后出特性。

### ::top

访问栈顶元素，复杂度：常数。

### ::empty

检查是否为空，复杂度：常数

### ::size

返回元素个数，复杂度：常数

### ::push

将元素推入栈，复杂度：通常和deque的push_back相同，即常数

### ::pop

将栈顶弹出，复杂度：通常和deque的pop_back相同，即常数

### ::emplace

在顶部原位构造元素，通常会用在栈的元素是结构体的时候，复杂度：通常和deque的emplace_back相同，即常数

## std::queue

队列，拥有先入先出特性。

### ::front

访问队首元素，复杂度：常数

### ::back

访问队尾元素，复杂度：常数

### 其他方法

同stack，不过push是推入队尾，pop是弹出队首。

## std::priority_queue

优先队列，提供常数时间的最大（或最小）元素查找，以及对数时间的插入与删除。

### 自定义比较方法

通常我们会重载元素的运算符来自定义

```cpp
struct node {
  int dis, u;
  bool operator>(const node& a) const { return dis > a.dis; }
};

priority_queue<node, vector<node>, greater<node> > pq;
```

### 方法

其方法与stack相同，只不过没有先入后出特性，插入元素或弹出元素后会根据大小关系进行排序，保证栈顶是最大的（或最小的）元素。

## std::deque

双端队列，允许在队首和队尾进行插入和删除。另外，在 deque 任一端插入或删除不会非法化指向其余元素的指针或引用。通常也会用来实现单调队列。

### 方法

size、empty与stack、queue相同。其pop_back、push_back、pop_front、push_front、emplace_front、emplace_back用法也类似。

## std::vector

通常可以理解为一个可以变化长度的数组。

### 声明方法

```cpp
std::vector<int> vec;//声明一个初始大小为0的vector
std::vector<int> vec2(n);//声明一个初始大小为n的vector，每个元素都会初始化为0
std::vector<int> vec3(n,1);//与上一个不同的是，每一个元素都会初始化为1
```

### 元素访问

```cpp
vec[5];//像数组一样访问
vec.at(5);//与上一个方法的差别在会进行越界检查
vec.front();//访问第一个元素
vec.back();//访问最后一个元素
```

### ::size

获取大小，复杂度：常数

### ::empty

查看是否为空，复杂度：常数

### ::push_back

向末尾添加元素，复杂度：常数

### ::pop_back

把末尾元素弹出，复杂度：常数

### ::emplace_back

在末尾原位构造元素，复杂度：常数

### std::vector\<bool\>

这是一个特化的vector，它每一个元素所占的空间是一位，而不是sizeof(bool)（通常是一字节）。

## std::bitset

表示一串二进制位。

### 声明方法

```cpp
std::bitset<100> bs;//声明一个位数为100位的bitset
std::bitset<4> bs2{0xA};//声明一个四位的bitset，其值等于0xA
```

### 元素访问

同数组的访问方式。同样也可以用数组的方式进行修改。

### ::all,::any,::none

检查是否全部，存在、没有元素被设置为true。

### ::count

返回设置为true的数量。

### 运算

bitset和bitset之间能用所有的位运算符。也可以用等号和不等号比较。

### ::flip

翻转某一位的值。

如果没有提供位置，就翻转所有。

### ::to_string

转化为二进制数的字符串。

### ::to_ulong,::to_ullong

转化为unsigned long和unsigned long long。

### ::set

设置某一位为1，如果没有提供位置，则将所有位设为1

### ::reset

设置某一位为0，如果没有提供位置，则将所有位设置为0

## std::pair

定义一个二元组，例如std::pair\<int,int\>, std::pair\<int,std::string\>等。其定义是在\<utility\>中，但是也可以#include \<algorithm\>来使用

### 元素访问

```cpp
std::pair<int,int> p;
p->first;//访问第一个元素
p->second;//访问第二个元素
```

### ::swap

交换两个元素的内容。复杂度：没有定义。

### std::make_pair

```cpp
auto p = std::make_pair(1,1);//自动推断类型为std::pair<int,int>
```

## std::tuple

定义一个多元组，可以说pair是tuple的特例。其定义是在\<utility\>中，但是也可以#include \<algorithm\>来使用

### 元素访问

根据下标可以如下访问

```cpp
auto t = std::make_tuple(1, "Foo", 3.14);
std::get<0>(t);//1
std::get<1>(t);//Foo
std::get<2>(t);//3.14
```

### std::make_tuple

同pair。

如果一个函数要返回tuple

```cpp
std::tuple<int, int> foo_tuple() 
{
  return {1, -1};  // N4387 前错误
  return std::tuple<int, int>{1, -1};  // 始终有效
  return std::make_tuple(1, -1); // 始终有效
}
```

需要注意兼容性，有些编译器不支持第一种返回方式。

### std::tie

将tuple解包。

```cpp
auto t = std::make_tuple(1,2,"Foo");
int a,b;
std::string str;
std::tie(a,b,str) = t;
```

当然也可以用auto，都不需要指定变量类型。

```cpp
auto[c,d,str2] = t;
```

## std::next_permutation, std::prev_permutation

```CPP
bool next_permutation (Iterator first, Iterator last);
bool prev_permutation(Iterator first, Iterator last);
```

这两个算法都是“原地”算法，也就是说会直接更改原数组，而不会返回一个新数组。这两个函数的作用是，获取按字典序比当前排列小1号的排列，以及大1号的排列。

例如123是一个排列，比它正好大1号的排列是132，再大1号的是213。比321小1号的是312。

用法如下

```CPP
string number = "213";
next_permutation(number.begin(), number.end());
cout << number;
```

输出231。

如果当前已经是最小的还要得到更小的，则返回false。已经是最大的还要得到更大的，则返回false。其他情况返回true。

复杂度：线性。

## std::unique

对一个已经排好序的数组去除重复元素。或者说是，移除一个一般数组中相邻的、相同的元素。复杂度：线性。

```CPP
Iterator unique(Iterator first, Iterator last);
```

给出一个范围来进行这个操作。具体用例可以见离散化一节。

返回值是新数组的末尾的迭代器。

## std::cin

### 输入十六进制、八进制、二进制

```CPP
int a;
std::cin>>std::hex>>a;//16进制
std::cin>>std::dec>>a;//10进制
std::cin>>std::oct>>a;//8进制
```

注意，使用一次std::hex之后所有的输入都会是十六进制，需要用std::dec输入一次十进制才会转换回来。

输入二进制可以考虑用bitset

```CPP
std::bitset<32> bs;
std::cin>>bs;
std::cout<<bs.to_ulong();
```

### 输入不忽略空格、回车

虽然可以用cin.get()和cin.getline()来实现，但是我们还是考虑用getchar比较好，当getchar返回EOF时代表输入结束。类似于逗号表达式返回最后一个的值，等号表达式返回等号左边的值，所以我们可以写(c=getchar())!=EOF。

## std::cout

### 输出十六进制、八进制、二进制

```CPP
int a = 16;
std::cout<<std::hex<<a;//16进制
std::cout<<std::oct<<a;//8进制
std::cout<<std::dec<<a;//10进制
```

使用注意事项同前。

输出二进制也是考虑用bitset

```CPP
std::bitset<32> bs{64};
std::string ans = bs.to_string();
std::cout<<ans.substr(ans.find('1'),int(ans.end()-ans.begin()))<<"\n";
```

需要去除前导0，如果直接输出bs或者其to_string的话会带有前导0

### 浮点数精度

```CPP
std::cout<<std::fixed;//如果不用这个，则为有效数字四位。
std::cout.precision(4);
std::cout<<a;//这里输出小数点后4位。
```

## scanf

### 输入十六进制、八进制、二进制

## printf

### 输出十六进制、八进制、二进制

## 正则表达式

### 正则表达式语法

|符号|功能|例子|
|-|-|-|
|literal|匹配字符串的值|foo|
|\\|转义符，将一些正则表达式需要的符号进行转义|\\.|
|()|括选一些字符，方便星号、加号等当作一个整体处理|a(abc)\*|
|re1\|re2|匹配re1或匹配re2的值|foo\|bar|
|.|匹配换行符以外的单字符|a.b|
|\^|在字符串开头匹配（在方括号里除外）|\^Dear|
|\$|在字符串的结尾匹配|\\.sh\$|
|\*|匹配星号前面的零次、一次或多次|a(abc)\*|
|+|匹配加号前面的一次或多次|a(abc)+|
|?|匹配问号前面的零次或一次|a(abc)?|
|\{N\}|指定匹配次数|a(abc){5}|
|\{M,N\}|匹配出现次数在M,N之间的，如果左边留空代表0次，右边留空代表任意次|a(abc)\{2,8\}|
|[...]|匹配其中的任意字符|[aeiou],[0-9A-Za-z]|
|\[\^...\]|匹配除了其中字符的任意字符|\[\^aeiou\]|
|\\n|匹配回车符||
|\\s|匹配任何空白字符||
|\\S|匹配任何非空字符||

**贪婪匹配**

\*，+都是贪婪匹配的，也就是说，如果用"\<.\*\>"匹配"\<h1\>测试\</h1\>"会匹配到全文，而不是只匹配到"\<h1\>"这叫做贪婪匹配。如果遇到第一个满足的就停下，要用"\<.\*?\>""

### C++正则表达式库

#### std::regex

一个类型，可以设定匹配的模式串。

```CPP
std::regex pattern(".*<.*?>.*");
```

后续如果要更换pattern的内容，可以使用pattern.assign("sth.")或者直接pattern = "sth."

#### std::smatch

一个类型，可以用于接收匹配的结果。

#### std::regex_match

用于测试字符串是否匹配模式串

```cpp
bool regex_match(string s,regex pattern);
bool regex_match(string s,smatch res,regex pattern);
bool regex_match(s.cbegin(),s.cend(),smatch res,regex pattern);
```

如果匹配到，就会返回1，否则返回0。

s代表被匹配的字符串，pattern代表模式串。res代表，如果这个字符串被匹配了，就会返回这个字符串。注意这里是完全匹配，后面介绍的search函数是子串匹配。

res是smatch类型，不能直接cout，要输出res[0]。

如果要传入一个字符串的范围，需要传入const_iterator，也就是s.cbegin()和s.cend()返回的版本（这其实是因为smatch的模板是const_iterator，如果是c风格字符数组，传入的是头尾指针，则要用cmatch）。此时如果匹配上，返回res是这个范围的字符串，而不是整个。

const_iterator不是指迭代器指的位置不能变，而是指你不能通过这个迭代器去修改元素。

#### std::regex_search

```cpp
bool regex_search(string s,regex pattern);
bool regex_search(string s,smatch res,regex pattern);
bool regex_search(s.cbegin(),s.cend(),smatch res,regex pattern);
```

参数、返回值同前。

只不过，这个东西不是完全匹配，只要有任意子串匹配，就会返回1。而且只要匹配到一个就会返回（当然这里要注意贪婪和非贪婪），不会再往后搜索，所以如果你使用res[1]是不会输出第二个结果的。res[0]返回的是匹配到的子串的结果，res[1]以后的东西也有含义，但是我暂时觉得没什么用。

res[0]的类型是sub_match，可以使用.second方法得到res[0]在s中的匹配结果位置的末尾的迭代器，所以要遍历所有匹配的子序列，可以这么写：

```CPP
while(std::regex_search(it_begin,it_end,res,pattern)){
   std::cout<<res[0]<<" "<<res.position()<<"\n";
   it_begin = res[0].second;
}
```

其中你还可以用.position返回子串开头的位置（相对于it_begin而言）。

#### std::regex_replace

```CPP
string regex_replace(string s,regex p,string rs)
```

s为源字符串，p为模式串，rs为匹配到的子串将会被替换成的字符串。

返回替换后的字符串。

这里会替换一切匹配到的子串，不像search一样麻烦。

如果我们只替换被匹配的子串的一部分，我们可以用如下方法

```CPP
string s = "abcd123abcd";
regex p("(abcd)([0-9]+)");
string ss = regex_replace(s,p,"a$2");
```

则我们的ss会变成a123abcd。$2代表的是第二个捕捉组的意思（这里下标又是从1开始的，而不是0）。捕捉组是括号括起来的算一组。

|转义符|含义|
|-|-|
|\$n|表示第n个捕捉组捕捉到的字符串|
|\$&|表示匹配到的整个子串，相当于\$0|
|\$\`|在源字符串中，在匹配到的子串左边的部分|
|\$\'|在源字符串中，在匹配到的子串右边的部分|
|\$\$|美元符号|
