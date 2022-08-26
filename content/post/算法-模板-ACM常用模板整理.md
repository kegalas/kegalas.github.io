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

## 排序

只给出归并排序，有可能在求逆序对的时候用得上。其他时候排序用sort函数即可。

### 归并排序

```c
#include <stdio.h>

void merge(long *num,long *tmp, int left, int mid_index ,int right){

    int first=left, second = mid_index+1, tmp_index = left;
    while(first<mid_index+1&&second<right+1){
        if(*(num+first)<*(num+second)){
            tmp[tmp_index] = *(num+first);
            tmp_index++;
            first++;
        }
        else{
            tmp[tmp_index] = *(num+second);
            tmp_index++;
            second++;
        }
    }
    while(first<mid_index+1){
        tmp[tmp_index++] = *(num+first);
        first++;
    }
    while(second<right+1){
        tmp[tmp_index++] = *(num+second);
        second++;
    }
    int i;
    for(i=left;i<=right;i++){
        num[i]=tmp[i];
    }

    return;
}

void merge_sort(long *num,long *tmp, int left, int right){
    int mid_index;
    if(left<right){
        mid_index = left + (right-left)/2;//这样写疑似可以避免int溢出
        merge_sort(num,tmp,left,mid_index);
        merge_sort(num,tmp,mid_index+1,right);
        merge(num,tmp,left,mid_index,right);
    } 
    return;
}

int main(){
    int num_count;
    long num[20000];
    long tmp[20000];
    scanf("%d",&num_count);
    int i;
    for(i=1;i<=num_count;i++){
        scanf("%d",&num[i]);
    }

    merge_sort(num,tmp,1,num_count);

    for(i=1;i<=num_count;i++){
        printf("%d ",num[i]);
    }

    return 0;
}
```

## 技巧

### 快速幂

```cpp
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

### 离散化

```cpp
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
    unique(assi.begin(),assi.end());

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

## 字符串

### KMP

```cpp
//kmp,luogu3375
#include <iostream>
#include <cstdio>
#include <string>

using namespace std;

#define MAXN 1000005

int nxt[MAXN];

string s1,s2;

int getNext(){
    nxt[0]=0;
    int r = 1;
    int l = 0;
    while (r<s2.length()){
        if (s2[l] == s2[r]){
            nxt[r]=l+1;
            r++;
            l++;
        }
        else if (l){
            l = nxt[l-1];
        }
        else{
            nxt[r]=0;
            r++;
        }
    }
    
    return 0;
}

int main(){
    cin>>s1>>s2;
    //给定两个字符串
    getNext();

    int pos=0,tar=0;

    while (tar<s1.length())
    {
        if(s1[tar]==s2[pos]){
            pos++;
            tar++;
        }
        else if (pos){
            pos = nxt[pos-1];
        }
        else{
            tar++;
        }

        if (pos==s2.length()){
            cout<<tar-pos+1<<endl;
            //输出s2在s1中出现的位置
            pos = nxt[pos-1];
        }
        
    }

    for(int i=0;i<s2.length();i++){
        cout<<nxt[i]<<" ";
        //表示s2​的长度为i的前缀的最长border长度。
    }

    cout<<endl;

    return 0;
}
```

### 字典树(Trie)

```cpp
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

## 数论

### 扩展欧几里得

```cpp
//求解ax+by=gcd(a,b)的一组解
//扩展欧几里得
#include <iostream>

using namespace std;

int exgcd(int a, int b, int &x, int &y){
    if(!b){
        x = 1;
        y = 0;//此时ax+by=gcd(a,b)中b=0，任何数与0的最大公约数是他本身，所以ax+0y=a，x=1 y=0
        return a;
    }
    int d = exgcd(b, a%b, x, y);
    int t = x;
    x = y;
    y = t-(a/b)*y;
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

### 欧几里得算法

```cpp
#include <iostream>

using namespace std;

int main(){
    long long a,b;
    cin>>a>>b;
    if(a<b){
        a=a+b;
        b=a-b;
        a=a-b;
    }

    long long ans = b;

    while((a%b)!=0){
        long long tmp = a%b;
        a = b;
        b = tmp;
        ans = b;
    }

    cout<<ans<<endl;
    //返回最大公约数

    return 0;
}
```

### 欧拉筛

```cpp
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

### Miller-Rabin素数测试

```cpp
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

### 乘法逆元

```cpp
//乘法逆元
//分为扩展欧几里得法、快速幂法、线性求逆元
//ax≡1(mod b)，x为a在乘法意义上的逆元，记作a^(-1)，或者inv(a)
//用扩展欧几里得法的角度看，就是求ax+by=1的整数解
//快速幂法利用费马小定理，需要b为素数

#include <iostream>
#include <cstdio>

using namespace std;

const int MAXN = 3000005;

int exgcd(int a, int b, int &x, int &y){
    if(!b){
        x=1;
        y=0;
        return a;
    }
    int d = exgcd(b,a%b,x,y);
    int tmp = x;
    x = y;
    y = tmp - a/b*y;
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

### 线性同余方程

```cpp
//ax≡c (mod b)求解x
//和ax+by=c等价
#include <iostream>

using namespace std;

int exgcd(int a, int b, int &x, int &y){
    if(!b){
        x=1;
        y=0;
        return a;
    }
    int d = exgcd(b, a%b, x, y);
    int tmp=x;
    x = y;
    y = tmp-a/b*y;
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

### 中国剩余定理

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

### 用乘法逆元计算组合数

根据

$$
(a/b)\%p=(a\times b^{-1})\%p=[(a\%p)\times(b^{-1}\%p)]\%p
$$

（如果加载不全，见[取余运算的分配律](https://kegalas.top/p/%E5%8F%96%E4%BD%99%E8%BF%90%E7%AE%97%E7%9A%84%E5%88%86%E9%85%8D%E5%BE%8B/)）

可以不用除法求出组合数。其中$b^{-1}$是$b$在模$p$意义下的逆元。

注意阶乘和其逆元的预处理。

```cpp
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

## 图论

### 最短路

#### Dijkstra

```cpp
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

#### Bellman-Ford

```cpp
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

#### SPFA

```cpp
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

#### Floyd

```cpp
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

### 拓扑排序

```cpp
//拓扑排序
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

### 最小生成树

#### Kruskal

```cpp
#include <iostream>
#include <algorithm>
#define MAXN 200005

using namespace std;

int u[MAXN],v[MAXN],w[MAXN];
int r[MAXN];//临时边序号，间接排序
int find_sets[MAXN];//并查集

int cmp(const int i, const int j){return w[i]<w[j];}

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

#### Prim算法

```cpp
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

### 最小树形图

#### 朱刘算法

```cpp
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

### 网络流

#### 最大流

##### Ford-Fulkerson

###### DFS实现的Ford-Fulkerson

```cpp
//luogu P3376
//超时
#include <iostream>
#include <vector>
#include <cstring>
#include <cstdio>

using namespace std;

typedef long long ll;
typedef unsigned long long ull;

const ll INF = 0xffffffff;
const int MAXM = 100005;

struct Edge{
    int to;
    int rev;
    ll cap;
    Edge()=default;
    Edge(int to, ll cap, int rev):to(to),cap(cap),rev(rev){}

};

vector<Edge> G[MAXM];
bool used[MAXM]; 

ll dfs(int v, int t, ll f){
    if(v==t) return f;
    used[v] = true;
    for(int i=0;i<G[v].size();i++){
        Edge& e=G[v][i];
        if(!used[e.to]&&e.cap>0){
            ll d = dfs(e.to, t, min(f,e.cap));
            if(d>0){
                e.cap-=d;
                G[e.to][e.rev].cap+=d;
                return d;
            }
        }
    }
    return 0;
}

ll max_flow(int s, int t){
    ll flow = 0;
    for(;;){
        memset(used,0,sizeof(used));
        ll f = dfs(s,t,INF);
        if(f==0) return flow;
        flow+=f;
    }
}

int main(){
    int n,m;
    cin>>n>>m;
    //点数，边数
    int s,t;
    cin>>s>>t;
    //源点，汇点
    for(int i=1;i<=m;i++){
        int a,b;
        ll c;
        scanf("%d%d%ld",&a,&b,&c);
        //起点，终点，边容量
        //cin>>a>>b>>c;
        G[a].push_back(Edge(b,c,G[b].size()));//这里第三个参数实际上是反向边的编号
        G[b].push_back(Edge(a,0,G[a].size()-1));
    }
    
    ll ans = max_flow(s,t);
    //得到最大流
    printf("%ld",ans);
    //cout<<ans<<endl;
    return 0;
}

/*
4 5 4 3
4 2 30
4 3 20
2 3 20
2 1 30
1 3 40
*/
```

###### EdmondsKarp

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <cstring>

using namespace std;

typedef long long ll;

const int MAXN = 205;
const ll INF = 1LL<<35;

struct Edge{
    int from, to;
    ll cap,flow;
    Edge(int u, int v, ll c, ll f):from(u),to(v),cap(c),flow(f){}
};

struct EdmondKarp{
    int n,m;
    vector<Edge> edges;  
    vector<int> G[MAXN]; //邻接表
    ll a[MAXN];
    ll p[MAXN];

    void init(int n){
        for(int i=0;i<n;i++){
            G[i].clear();
        }
        edges.clear();
    }

    void AddEdge(int from, int to, ll cap){
        edges.push_back(Edge(from, to, cap, 0));
        edges.push_back(Edge(to, from, 0, 0));
        m = edges.size();
        G[from].push_back(m-2);
        G[to].push_back(m-1);
    }

    int Maxflow(int s, int t){
        ll flow = 0;
        for(;;){
            memset(a, 0 ,sizeof(a));
            queue<ll> Q;
            Q.push(s);
            a[s] = INF;
            while(!Q.empty()){
                ll x=Q.front();
                Q.pop();
                for(int i=0;i<G[x].size();i++){
                    Edge& e = edges[G[x][i]];
                    if(!a[e.to]&&e.cap>e.flow){
                        p[e.to] = G[x][i];
                        a[e.to] = min(a[x], e.cap-e.flow);
                        Q.push(e.to);
                    }
                }
                if(a[t]) break;
            }
            if(!a[t]) break;
            for(int u=t;u!=s;u=edges[p[u]].from){
                edges[p[u]].flow+=a[t];
                edges[p[u]^1].flow -= a[t];
            }
            flow += a[t];
        }
        return flow;
    }
};

int main(){
    EdmondKarp EK;
    cin>>EK.n;
    //点数
    int s,t;
    int m;
    cin>>m>>s>>t;
    //边数，源点，汇点
    for(int i=1;i<=m;i++){
        int tmp1,tmp2,tmp3;
        cin>>tmp1>>tmp2>>tmp3;
        //起点，终点，边容量
        EK.AddEdge(tmp1,tmp2,tmp3);
    }

    cout<<EK.Maxflow(s,t)<<endl;

    return 0;
}
```

### 连通性相关

#### 强连通分量

##### Tarjan算法

```cpp
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

### 割点

#### Tarjan算法

```cpp
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

### 割边

#### Tarjan算法

```cpp
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

## 计算几何

### 二维凸包

#### Andrew扫描法

```cpp
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

### 旋转卡壳求最远点对

```cpp
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

## 组合数学

### 卡特兰数

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

### 稳定婚姻问题

#### Gale-Shapley算法

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

## 树

### 树状数组

```cpp
//树状数组

#include <iostream>
#include <cstdio>
#define MAXN 500005

using namespace std;

int arr[MAXN];
int bit[MAXN];

int n,m;

inline int lowbit(int n){
    return n&(-n);
}

void update(int p, int k){
    for(;p<=n;p+=lowbit(p)){
        bit[p]+=k;
    }
}

long long query(int p){
    int ans=0;
    for(;p;p-=lowbit(p)){
        ans+=bit[p];
    }
    return ans;
}

int main(){
    scanf("%d%d",&n,&m);
    //数组长度，查询数
    for(int i=1;i<=n;i++){
        scanf("%d",&arr[i]);
        update(i,arr[i]);
    }

    for(int i=1;i<=m;i++){
        int op;
        int x,y,k;
        scanf("%d",&op);
        scanf("%d",&x);
        if(op==1){
            scanf("%d",&k);
            //将单点增加k，如果想要改成修改，则可以update(x,-arr[x]+k)
            update(x,k);
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

#### 树状数组求逆序对

```cpp
//Luogu P1908
#include <iostream>
#include <algorithm>

#define LL long long

const int MAXN = 500005;

LL arr[MAXN];
LL n;

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

void update(LL p, LL k){
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
        update(arr[i],1);
    }

    ans = n*(n-1)/2-ans;//本来统计的是等于或顺序对，现在反过来计算逆序对

    std::cout<<ans<<"\n";

    return 0;
}
```

### 并查集

```cpp
//并查集

#include <iostream>
using namespace std;

const int MAXN = 1005;

int find_sets[MAXN];

int findf(int x){
    return find_sets[x]==x ? x : find_sets[x] = findf(find_sets[x]);
}

void unionSet(int x, int y){
    x = findf(x);
    y = findf(y);
    find_sets[x] = y;
}

int main(){
    int n;
    cin>>n;
    //点数
    for(int i=1;i<=n;i++){
        find_sets[i]=i;
    }
    int m;
    cin>>m;
    //边数
    for(int i=1;i<=m;i++){
        int a,b;
        cin>>a>>b;
        find_sets[b] = a;
    }
    cout<<findf(5)<<" "<<findf(8)<<endl;
    unionSet(5,8);
    cout<<findf(5)<<" "<<findf(8)<<endl;
    return 0;
}

/*
8 6
1 2
1 3
3 4
3 5
6 7
7 8
*/
```

### 线段树

```cpp
//luogu 3372
#include <iostream>
#include <cstdio>

using namespace std;

typedef long long ll;

const int MAXN = 100005;

ll st[MAXN*4+2];//对于一颗线段树，n个数所组成的树最多有4n-5个节点，开大了一点
ll tag[MAXN*4+2];
ll arr[MAXN];

void build(int s, int t, int p){//区间左端点、右端点、区间编号
    if(s==t){
        st[p] = arr[s];
        return;
    }
    int m = s+((t-s)>>1);//写成(s+t)>>1可能会爆
    build(s,m,p*2);
    build(m+1,t,p*2+1);
    st[p] = st[p*2]+st[p*2+1];
}

void update(int l, int r, int s, int t, int p, ll c){//c表示加减的数值
    if(l<=s&&t<=r){
        st[p]+=(t-s+1)*c;
        tag[p]+=c;
        return;
    }
    ll m = s + ((t-s)>>1);
    if(tag[p]&&s!=t){
        st[p*2]   += (m-s+1)*tag[p];
        st[p*2+1] += (t-m)*tag[p];
        tag[p*2]  += tag[p];
        tag[p*2+1]+= tag[p];
        tag[p]=0;
    }
    if(l<=m) update(l, r, s, m, p*2, c);
    if(r>m)  update(l, r, m+1, t, p*2+1, c);
    st[p] = st[p*2] + st[p*2+1];
}

ll query(int l, int r, int s, int t, int p){
    //查询[l,r]的和
    if(l<=s&&t<=r){
        return st[p];
    }
    ll sum=0;
    ll m = s+((t-s)>>1);
    if(tag[p]){
        st[p*2]   += (m-s+1)*tag[p];
        st[p*2+1] += (t-m)*tag[p];
        tag[p*2]  += tag[p];
        tag[p*2+1]+= tag[p];
        tag[p]=0;
    }
    if(l<=m) sum+=query(l,r,s,m,p*2);
    if(r>m)  sum+=query(l,r,m+1,t,p*2+1);
    return sum;
}

int main(){
    int n,m;
    scanf("%d%d",&n,&m);
    //数组长度，查询次数
    for(int i=1;i<=n;i++){
        scanf("%ld",&arr[i]);   
    }
    build(1,n,1);

    for(int i=1;i<=m;i++){
        int ope;
        cin>>ope;
        if(ope==1){
            int x,y,z;
            scanf("%d%d%d",&x,&y,&z);
            //[x,y]加上z
            update(x, y, 1, n, 1, z);
        }
        else{
            int x,y;
            scanf("%d%d",&x,&y);
            //查询[x,y]的和
            cout<<query(x, y, 1, n, 1)<<endl;
        }
    }

    return 0;
}
```

## 倍增

### ST表

对于经典的RMQ（即给定一个数组，求区间内的最大值）问题，有如下代码

```cpp
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

### 倍增求最近公共祖先

```cpp
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

## 二分

### 二分答案

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

如果是浮点数的二分，则不推荐使用EPS进行精度判断（有可能会丢精度）。而是使用计数器，一般迭代100次就能保证符合题目要求。

## 概率论

### 处理分数期望、概率

有时候，题目中的期望是一个分数$\frac{P}{Q}$，而为了防止精度问题，往往会要求输出一个$R$，满足

$$
R\times Q\equiv P(mod\ 998244353)
$$

此时

$$
R = (P\times Q^{-1})\%998244353
$$

$Q^{-1}$是$Q$在模$998244353$意义下的乘法逆元

## C++ STL用法

### std::swap

交换两个元素的内容（也可以交换数组，不重要不介绍）。复杂度：常数。

```cpp
int a,b;
std::swap(a,b);
```

注意其中的两个参数，类型要相同。不能一个是LL一个是int。

### std::sort

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

### std::lower_bound,std::upper_bound

对某个已经排序好的数组，查找第一个大于等于（lower_bound）或者大于(upper_bound)某个给定值的元素。复杂度：logn。

```cpp
int a[5]={0,1,3,4,6};
int first = std::lower_bound(a,a+5,3);
```

同样是左闭右开区间，第三个参数是指定的值。如果找到就会返回所查找元素的迭代器（或者指针）。找不到就会返回末尾元素的后一个指针（或者end迭代器）。

如果需要自定义比较方法，同sort函数。

### std::max,std::min

对于两个元素返回最大值和最小值。复杂度：准确一次比较。

```cpp
int a=1,b=2;
int maxv = std::max(a,b);
int minv = std::min(a,b);
```

同样，两个参数类型相同。自定义比较方法同sort。

### std::abs

计算绝对值。复杂度：文档没写但应该是常数。

```cpp
int a = -1;
int b = std::abs(a);
```

注意，函数只有float,double,long double的返回值类型。使用时如果给予整数参数会自动转换，这是否会导致精度问题有待观察。

### std::string

#### ::swap

将两个字符串互换。复杂度：常数。

```cpp
std::string str = "123456";
std::string str2 = "456789";
str.swap(str2);
```

#### ::begin,std::end

返回字符串的起始得带器和结尾迭代器。

```cpp
str.begin();str.end();
```

#### ::size

返回字符串的大小。

```cpp
str.size();
```

#### ::push_back

向字符串末尾添加一个字符，同时大小加一。复杂度：常数。

```cpp
str.push_back('a');
```

#### ::pop_back

将字符串末尾的字符弹出，同时大小减一。如果字符串为空则未定义。复杂度：常数。

```cpp
str.pop_back();
```

#### ::find

在字符串中寻找某个子串是否存在。复杂度：没有规定，编译器不一定都是使用的kmp算法。

```cpp
std::string::size_type n;
std::string s = "this is a string";

n = s.find("is");
```

如果找到则返回首个匹配的首字母位置。否则返回std::string::npos。如果是int n作为s.find的接收端，则会在找不到时接收到-1。

### std::memset

将值复制到dest所指对象的前count个字节中。复杂度：没有规定。

```cpp
int a[20];
std::memset(a,0,sizeof(a));
```

注意，赋的值不能随便取，这个函数是一个字节一个字节地去赋值的。如果取1并不会得到全部赋值为1的效果，通常只会取0和-1。

### std::map

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

#### 自定义比较函数

map通常会按照key的大小关系进行升序排列。如果要自定义比较函数，则

```cpp
struct cmp{
    bool operator()(const int& a, const int& b) const{
        return a>b;
    }
};

std::map<int,std::string,cmp> mp;
```

#### ::empty

检测是否为空。

#### ::size

返回大小。

#### ::clear

清除所有内容。复杂度：线性。

#### ::erase

提供迭代器，删除迭代器所指的键值对。复杂度：常数。

```cpp
auto it = mp.begin();
mp.erase(it);
```

#### ::find

寻找key等于给定值的元素，返回迭代器。如果没有找到则返回end迭代器。复杂度：对数。

```cpp
auto it = mp.find(1);
```

#### ::lower_bound,::upper_bound

寻找首个大于等于(或大于，对upper_bound)给定值的key。复杂度：对数。

```cpp
auto it = mp.lower_bound(1);
```

### std::unordered_map

可以看作是无序的map，通常由哈希表实现。这意味着map中和排序有关的函数都不能使用。

### std::set 

set是关联容器，含有Key类型对象的已排序集，通常用红黑树实现。

```cpp
template<

    class Key,
    class Compare = std::less<Key>,
    class Allocator = std::allocator<Key>
> class set;
```

遍历的方式与map相同。

#### 使用方法

自定义比较函数,empty,size,clear,erase,find,lower_bound,upper_bound都与map相同。

### std::unordered_set

用哈希实现，没有内部排序。

### std::stack

栈，有先入后出特性。

#### ::top

访问栈顶元素，复杂度：常数。

#### ::empty

检查是否为空，复杂度：常数

#### ::size

返回元素个数，复杂度：常数

#### ::push

将元素推入栈，复杂度：通常和deque的push_back相同，即常数

#### ::pop

将栈顶弹出，复杂度：通常和deque的pop_back相同，即常数

#### ::emplace

在顶部原位构造元素，通常会用在栈的元素是结构体的时候，复杂度：通常和deque的emplace_back相同，即常数

### std::queue

队列，拥有先入先出特性。

#### ::front

访问队首元素，复杂度：常数

#### ::back

访问队尾元素，复杂度：常数

#### 其他方法

同stack，不过push是推入队尾，pop是弹出队首。

### std::priority_queue

优先队列，提供常数时间的最大（或最小）元素查找，以及对数时间的插入与删除。

#### 自定义比较方法

通常我们会重载元素的运算符来自定义

```cpp
struct node {
  int dis, u;
  bool operator>(const node& a) const { return dis > a.dis; }
};

priority_queue<node, vector<node>, greater<node> > pq;
```

#### 方法

其方法与stack相同，只不过没有先入后出特性，插入元素或弹出元素后会根据大小关系进行排序，保证栈顶是最大的（或最小的）元素。

### std::deque

双端队列，允许在队首和队尾进行插入和删除。另外，在 deque 任一端插入或删除不会非法化指向其余元素的指针或引用。通常也会用来实现单调队列。

#### 方法

size、empty与stack、queue相同。其pop_back、push_back、pop_front、push_front、emplace_front、emplace_back用法也类似。

### std::vector

通常可以理解为一个可以变化长度的数组。

#### 声明方法

```cpp
std::vector<int> vec;//声明一个初始大小为0的vector
std::vector<int> vec2(n);//声明一个初始大小为n的vector，每个元素都会初始化为0
std::vector<int> vec3(n,1);//与上一个不同的是，每一个元素都会初始化为1
```

#### 元素访问

```cpp
vec[5];//像数组一样访问
vec.at(5);//与上一个方法的差别在会进行越界检查
vec.front();//访问第一个元素
vec.back();//访问最后一个元素
```

#### ::size

获取大小，复杂度：常数

#### ::empty

查看是否为空，复杂度：常数

#### ::push_back

向末尾添加元素，复杂度：常数

#### ::pop_back

把末尾元素弹出，复杂度：常数

#### ::emplace_back

在末尾原位构造元素，复杂度：常数

#### std::vector<bool>

这是一个特化的vector，它每一个元素所占的空间是一位，而不是sizeof(bool)（通常是一字节）。

### std::bitset

表示一串二进制位。

#### 声明方法

```cpp
std::bitset<100> bs;//声明一个位数为100位的bitset
std::bitset<4> bs2{0xA};//声明一个四位的bitset，其值等于0xA
```

#### 元素访问

同数组的访问方式。同样也可以用数组的方式进行修改。

#### ::all,::any,::none

检查是否全部，存在、没有元素被设置为true。

#### ::count

返回设置为true的数量。

#### 运算

bitset和bitset之间能用所有的位运算符。也可以用等号和不等号比较。

#### ::flip

翻转某一位的值。

#### ::to_string

转化为二进制数的字符串。

#### ::to_ulong,::to_ullong

转化为unsigned long和unsigned long long。



