---
title: "算法竞赛常用模板整理"
date: 2021-12-18T15:57:37+08:00
draft: false
tags: [算法,模板]
description: 包含ICPC竞赛常用的一些模板
categories: 算法
mathjax: true
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

## 字符串

### KMP

```cpp
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
            pos = nxt[pos-1];
        }

    }

    for(int i=0;i<s2.length();i++){
        cout<<nxt[i]<<" ";
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

const int cnt = 10;

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


bool millerRabin(int n){
    if(n==2) return true;
    for(int i=0;i<cnt;i++){
        int a = rand() % (n-2) + 2;
        if(qPowMod(a,n,n)!=a) return false;
    }
    return true;
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
    for(int i=1;i<=k;i++){
        cin>>a[i]>>r[i];
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
    return 0;
}
```

## 图论

### 最短路

#### Dijkstra

```cpp
//dijkstra
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

    //freopen("in.in","r",stdin);
    //freopen("out.out","w",stdout);

    scanf("%d%d%d",&n,&m,&s);



    for(int i=1;i<=m;i++){
        int a,b,c;
        scanf("%d%d%d",&a,&b,&c);
        edge t;
        t.v=b;
        t.w=c;
        graph[a].push_back(t);
    }

    for(int i=1;i<=n;i++){
        dis[i] = MAXINT;
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
//bellman-ford
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
    spfa
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
    int n,m;
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
        cin>>a>>b>>v;
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
        cin>>a>>b;//本次是无权图
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
//kruskal
#include <iostream>
#include <algorithm>
#define MAXN 500005


using namespace std;


int u[MAXN],v[MAXN],w[MAXN];//一条边的两个点和边权值
int r[MAXN];//临时边序号，间接排序
int find_sets[MAXN];//并查集


int cmp(const int i, const int j){return w[i]<w[j];}

int find(int x){return find_sets[x]==x ? x : find_sets[x] = find(find_sets[x]);}

int main(){
    int n,m;//点数和边数


    cin>>n>>m;
    for(int i=1;i<=m;i++){
        cin>>u[i]>>v[i]>>w[i];
    }

    for(int i = 1;i<=n;i++){
        find_sets[i]=i;
    }
    for(int i=1;i<=m;i++){
        r[i]=i;
    }

    sort(r+1,r+m+1,cmp);

    int ans = 0;

    for(int i=1;i<=m;i++){
        int tmp = r[i];
        int x = find(u[tmp]);
        int y = find(v[tmp]);

        if(x!=y){
            ans += w[tmp];
            find_sets[x] = y;

        }
    }

    cout<<ans<<endl;


    return 0;
}
```

### 网络流

#### 最大流

##### Ford-Fulkerson

###### DFS实现的Ford-Fulkerson

```cpp
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
    int s,t;
    cin>>s>>t;
    for(int i=1;i<=m;i++){
        int a,b;
        ll c;
        scanf("%d%d%ld",&a,&b,&c);
        //cin>>a>>b>>c;
        G[a].push_back(Edge(b,c,G[b].size()));//这里第三个参数实际上是反向边的编号
        G[b].push_back(Edge(a,0,G[a].size()-1));
    }

    ll ans = max_flow(s,t);
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
#define MAXN 50005
#define INF 0x3f3f3f3f

using namespace std;

struct Edge{
    int from, to, cap, flow;
    Edge(int u, int v, int c, int f):from(u),to(v),cap(c),flow(f){}
};

struct EdmondKarp{
    int n,m;
    vector<Edge> edges;  
    vector<int> G[MAXN]; //邻接表
    int a[MAXN];
    int p[MAXN];

    void init(int n){
        for(int i=0;i<n;i++){
            G[i].clear();
        }
        edges.clear();
    }

    void AddEdge(int from, int to, int cap){
        edges.push_back(Edge(from, to, cap, 0));
        edges.push_back(Edge(to, from, 0, 0));
        m = edges.size();
        G[from].push_back(m-2);
        G[to].push_back(m-1);
    }

    int Maxflow(int s, int t){
        int flow = 0;
        for(;;){
            memset(a, 0 ,sizeof(a));
            queue<int> Q;
            Q.push(s);
            a[s] = INF;
            while(!Q.empty()){
                int x=Q.front();
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
    int s,t;
    int m;
    cin>>m>>s>>t;
    for(int i=1;i<=m;i++){
        int tmp1,tmp2,tmp3;
        cin>>tmp1>>tmp2>>tmp3;
        EK.AddEdge(tmp1,tmp2,tmp3);
    }

    cout<<EK.Maxflow(s,t)<<endl;

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
        //cout<<i<<" "<<j<<endl;
        if((ans[(i+1)%tn]-ans[i]).det(ans[(j+1)%tn]-ans[j])<0){
            i = (i+1)%tn;
        }else{
            j = (j+1)%tn;
        }

        cnt++;
    }
    cout<<res<<endl;
}




int main(){
    cin>>n;
    vector<Point> qs;
    for(int i=0;i<n;i++){
        cin>>po[i].x>>po[i].y;
    }
    sort(po,po+n,cmp);
    qs = convexHull();
    rc(qs);
    return 0;
}
```

## 组合数学

### 卡特兰数

```cpp
#include <iostream>

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
queue<int>freemale;

int main(){
    int t;
    scanf("%d",&t);
    while(t--){
        scanf("%d",&couple);
        while(!freemale.empty()){
            freemale.pop();
        }
        for(int i=0;i<couple;i++){
            scanf("%s",str);
            maleName[i]=str[0]-'a';
            freemale.push(maleName[i]);
        }
        sort(maleName, maleName+couple);

        for(int i=0;i<couple;i++){
            scanf("%s",str);
            femaleName[i]=str[0]-'A';
        }

        for(int i=0;i<couple;i++){
            scanf("%s",str);
            for(int j=0;j<couple;j++){
                maleLike[i][j]=str[j+2]-'A';
            }
        }
        for(int i=0;i<couple;i++){
            scanf("%s",str);
            for(int j=0;j<couple;j++){
                femaleLike[i][str[j+2]-'a']=couple-j;
            }
            femaleLike[i][couple]=0;
        }
        memset(maleChoice,0,sizeof(maleChoice));

        for(int i=0;i<couple;i++){
            femaleChoice[i]=couple;
        }
        while(!freemale.empty()){
            int male=freemale.front();
            int female=maleLike[male][maleChoice[male]];
            if(femaleLike[female][male]>femaleLike[female][femaleChoice[female]]){
                freemale.pop();
                if(femaleChoice[female]!=couple){
                    freemale.push(femaleChoice[female]);
                    maleChoice[femaleChoice[female]]++;
                }
                femaleChoice[female]=male;
            }
            else
                maleChoice[male]++;
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
            update(x,k);
        }
        else{
            scanf("%d",&y);
            cout<<query(y)-query(x-1)<<endl;
        }
    }
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
    for(int i=1;i<=n;i++){
        find_sets[i]=i;
    }
    int m;
    cin>>m;
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
    for(int i=1;i<=n;i++){
        scanf("%d",&arr[i]);   
    }
    build(1,n,1);

    for(int i=1;i<=m;i++){
        int ope;
        cin>>ope;
        if(ope==1){
            int x,y,z;
            scanf("%d%d%d",&x,&y,&z);
            update(x, y, 1, n, 1, z);
        }
        else{
            int x,y;
            scanf("%d%d",&x,&y);
            cout<<query(x, y, 1, n, 1)<<endl;
        }
    }

    return 0;
}
```
