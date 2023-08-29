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

#define debug(a) std::cout<<#a<<"="<<(a)<<"\n"
#define rep(i,x,y) for(int i=(x);i<=(y);i++)
#define rrep(i,x,y) for(int i=(x);i>=(y);i--)
#define mms(x) memset((x), 0, sizeof(x))
#define pb push_back
#define mkp std::make_pair
#define fi first
#define se second

using LL = long long;
using ULL = unsigned long long;
using DB = double;
using LD = long double;
using ui = unsigned;
using i128 = __int128;
using pii = std::pair<int,int>;
using pll = std::pair<LL,LL>;

int const MAXN = 200005;
int const INF = 0x7fffffff;
DB const EPS = 1e-8;
int const MOD = 998244353;
DB const PI = acos(-1);

int arr[MAXN];

void solve(){
    int n;
    std::cin>>n;
    for(int i=1;i<=n;i++){
        std::cin>>arr[i];
    }
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);

	int T;
	std::cin>>T;
	//T=1;
	while(T--){
	    solve();
	}

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
    //前缀函数pi[i]是满足s[0...x-1]==s[i-x+1...i]的最大的x
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

//我们可以通过把模式串和主串拼在一起，p+#+s，然后求这个字符串的前缀函数表（#代表不在主串、模式串字符集内的一个符号），然后pi[i]如果等于模式串的长度，那么i是匹配模式串的子串的起点。
//关于最小循环结，设字符串下标从1开始，长度为n，则如果n%(n-pf[n])==0，则有循环节，并且长度为n-pf[n]（当然长度可以为n）
```

## 字典树(Trie)

```cpp
//复杂度 插入或查找一次 模板串长度
//luogu p8306
class Trie{
public:
    int nxt[MAXM][26];
    int cnt;
    
    void init(){
        for(int i=0;i<=cnt;i++) for(int j=0;j<26;j++) nxt[i][j] = 0;
        cnt = 0;//起始节点编号为0
    }
    
    void insert(std::string const & s){
        int cur = 0;
        for(auto c:s){
            if(!nxt[cur][c-'a']){
                nxt[cur][c-'a']=++cnt;
            }
            cur = nxt[cur][c-'a'];
        }
    }
    
    bool find_prefix(std::string const & s){
        int cur=0;
        for(auto c:s){
            if (!nxt[cur][c-'a']){
               return false;
            }            
            cur = nxt[cur][c-'a'];
        }
        return true;
    }
};

Trie trie;
```

## 字符串哈希

主要是用于判断两个字符串是否相等。通常我们的Hash函数会把字符串看成是某个进制下的自然数。把这个自然数对某个大质数取模得到Hash值。

求Hash的复杂度是$O(n)$，如果我们要比较一大群字符串里有多少不同的，我们不应该两两比较，而要把它们的Hash全部记录下来，再去排序Hash、比较。

这种哈希函数的性质为：

1. 设已知字符串$S$的Hash值为$H(S)$，那么添加一个字符$c$后的新字符串的Hash值为$H(S+c)=(H(S)*base+value[c])\%MOD$。其中$value[c]$表示$c$代表的数值，如果用char类型直接把$c$转换为int或者ULL什么的就行了
2. 已知$H(S)$和$H(S+T)$，则$H(T)=(H(S+T)-H(S)*base^{length(T)})\%MOD$，直观理解的话，就是在$base$进制下，在S后面补$0$，把$S$左端和$S+T$左端对齐，相减得到$H(T)$。利用这个性质可以方便地先预处理字符串所有前缀的Hash，然后查询连续子串的Hash。

```cpp
//自然溢出法，相当于对2^64取模的单哈希，是比较容易被特殊数据卡掉的
//luogu P3370
using ULL = unsigned long long;
ULL H(std::string const & str){
    ULL ret = 0;
    ULL const base = 131;//ascii也就128个字符，质数131作为底数足够
    
    for(auto c:str){
        ret = ret*base+(ULL)c;
    }
    return ret;
}
```

```cpp
//双哈希法，两个字符串的两个哈希必须分别相同，才会被考虑为相同的字符串
//比起单哈希，更不容易被卡
//luogu P3370
ULL const MOD1 = 998244353;
ULL const MOD2 = 1e9+7;
ULL const base = 131;

struct Data{//把字符串的两个哈希捆起来，便于排序比较等操作
    ULL x,y;
};

ULL H1(std::string const & str){
    ULL ret = 0;
    for(auto c:str){
        ret = (ret*base+(ULL)c)%MOD1;
    }
    return ret;
}

ULL H2(std::string const & str){
    ULL ret = 0;
    for(auto c:str){
        ret = (ret*base+(ULL)c)%MOD2;
    }
    return ret;
}
```

## AC自动机

```cpp
//复杂度 文本串长度+模板串长度之和
//AC自动机，luogu P3808
//AC自动机会把Trie修改掉，并不是插入时候的字典树了，实际上变成了一张图+fail指针。
//trie数组表示从当前状态添加一个字符后到达的状态，fail数组表示，目前为止匹配，但是添加下一个字符后不匹配了，跳转至最长的后缀状态（不包括添加的下一个字符）。可以反复跳转。
//注意，一个状态是可行状态，当且仅当它自己可行，或者fail指针指向的状态可行，或者fail[fail]可行，以此类推

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

## Z函数

```cpp
//Z函数，复杂度O(n)
//luogu P5410
std::vector<int> zFunc(std::string const & str){
    //求出一个字符串的z函数，即满足z[i]是s[0...x-1]==s[i...i+x-1]的最大的x，这个子串也叫做LCP
    //特别的z[0]=0，也有些题目是等于串长，需要自行调整
    //kmp里面添加字符集外字符的操作在这里也可以用
    int n = str.size();
    std::vector<int> z(n);
    
    for(int i=1,l=0,r=0;i<n;i++){
        if(z[i-l]<r-i+1)
            z[i] = z[i-l];
        else{
            z[i] = std::max(r-i+1,0);
            while(i+z[i]<n && str[z[i]]==str[i+z[i]])
                z[i]++;
            l = i, r = i + z[i] - 1;
        }
    }
    
    return z;
}
```

## 后缀数组

首先是$O(n\log^2n)$的，没有用到基数排序（因为我不排除会求某种只给出偏序关系的后缀数组）

```cpp
//求后缀数组，复杂度O(nlog^2n)
//luogu p3809
int rk[MAXN<<1],sa[MAXN],tarr[MAXN<<1];
//rk[i]表示后缀i（从1开始，后缀i代表字符串从i开始到结束的子串）的排名，sa[i]表示所有后缀第i小的起点序号，排名和编号都从1开始

void getSA(std::string const & s){
    int n = s.size();
    if(n==1){
        rk[1] = sa[1] = 1;
        return;
    }
    
    for(int i=1;i<=n;i++)
        sa[i] = i, rk[i] = s[i-1];
    
    for(int w=1;w<n;w<<=1){
        auto rp = [&](int x){return std::make_pair(rk[x],rk[x+w]);};
        std::sort(sa+1,sa+n+1,[&](int x, int y){return rp(x)<rp(y);});
        for(int i=1,p=0;i<=n;i++)
            tarr[sa[i]] = rp(sa[i-1])==rp(sa[i]) ? p:++p;
        std::copy(tarr+1,tarr+n+1,rk+1);
    }
}
```

再给出$O(n\log n)$的

```cpp
//求后缀数组，复杂度O(nlogn)
//luogu p3809
int rk[MAXN<<1],sa[MAXN],tarr[MAXN<<1],cnt[MAXN],rkt[MAXN];
//rk[i]表示后缀i（从1开始，后缀i代表字符串从i开始到结束的子串）的排名，sa[i]表示所有后缀第i小的起点序号，排名和编号都从1开始

void getSA(std::string const & s){
    int n = s.size();
    if(n==1){
        rk[1] = sa[1] = 1;
        return;
    }
    
    int m = 128;
    for(int i=1;i<=n;i++)
        ++cnt[rk[i]=s[i-1]];
    for(int i=1;i<=m;i++)
        cnt[i] += cnt[i-1];
    for(int i=n;i>=1;i--)
        sa[cnt[rk[i]]--] = i;
    
    for(int w=1;;w<<=1){
        for(int i=n;i>n-w;i--)
            tarr[n-i+1] = i;
        for(int i=1,p=w;i<=n;i++)
            if(sa[i]>w) tarr[++p]=sa[i]-w;
        std::fill(cnt+1,cnt+m+1,0);
        for(int i=1;i<=n;i++)
            cnt[rkt[i] = rk[tarr[i]]]++;
        for(int i=1;i<=m;i++)
            cnt[i]+=cnt[i-1];
        for(int i=n;i>=1;i--)
            sa[cnt[rkt[i]]--] = tarr[i];
        m = 0;
        auto rp = [&](int x){return std::make_pair(rk[x],rk[x+w]);};
        for(int i=1;i<=n;i++)
            tarr[sa[i]] = rp(sa[i-1])==rp(sa[i]) ? m:++m;
        std::copy(tarr+1,tarr+n+1,rk+1);
        if(n==m) break;
    }
}
```

## 后缀自动机

```cpp
//后缀自动机，构建SAM的复杂度为O(n)，空间复杂度为O(|S|n)，|S|为字符集的大小
//luogu p3804
//SAM是动态构建的，每次插入一个字符即可
struct State{
    int fa,len,next[26];//似乎有些编译器next是保留字，需要注意
};
//endpos(p)为模式串p在s中所有出现的结束位置的集合，例如aababc中，ab出现了两次，结束位置是{3,5}。endpos等价类即，endpos相同的子串构成的集合。例如b和ab都是结束在{3,5}，则它们是等价类。这说明它们总是一起出现，可以归到一个节点，并且长度小的一定是长度大的的后缀。
//next[ch]表示接受ch后的状态；fa表示该状态在parent tree上的父节点；len表示该状态对应的endpos等价类中最长串的长度。
//假设b是a的fa，a等价类的最长字符串为s，则b的最长字符串为，s的所有后缀中，不在等价类a里的，最长的字符串。

//SAM可以接受字符串的所有后缀，但是它包含了所有子串的信息。也就是从任意一个状态开始的路径，都是字符串的子串。
//可行状态是last状态，以及fa[last]、fa[fa[last]]直到根节点（空字符串）。

//除了等价类的最长字符串长度len，我们也可以计算minlen。等价类里所有字符串的长度恰好覆盖[minlen,len]之间的每一个整数。除了根节点，st[i].minlen = st[fa[i]].len+1;
//每个状态i对应的子串数量是st[i].len-st[st[i].fa].len

//算法并没有维护endpos等效类中，字符串出现的位置个数，需要自己去树形dp。
//注意，aababb中，ab的等价类为{3,5}，根据ab前面一个字符不同可以划分不同的等价类，例如aab和bab划分为{3},{5}。但是a的等价类为{1,2,4}，因为第一个的前面一个字符是空字符，只能划分出两个，即aa{2},ba{4}，树形DP需要在parent tree上注意缺少的这一部分。在建SAM时预处理dp[cur] = 1

class SAM{
public:
    State st[MAXN<<1];//SAM总状态数不会超过2n-1，总转移数不超过3n-4
    int cnt = 1, last = 1;
    //起始节点编号为1；last表示加入新字符前整个字符串所在的等价类
    
    void insert(int ch){
        ch = ch-'a';
        int cur = ++cnt;
        int p = 0;
        st[cur].len = st[last].len + 1;
        for(p=last;p&&!st[p].next[ch];p=st[p].fa)
            st[p].next[ch] = cur;
        //对于每一个father状态，如果不存在ch的转移边，则连到cur
        int q = st[p].next[ch];
        if(q==0){
            //加入了从未加入的字符
            st[cur].fa = 1;
        }
        else if(st[p].len + 1 == st[q].len){
            //p到q是连续的转移
            st[cur].fa = q;
        }
        else{
            //不连续的转移
            //会新增一个节点r,拥有q的所有出边，father与q相同
            int r = ++cnt;
            st[r] = st[q];
            st[r].len = st[p].len + 1;
            for(;p&&st[p].next[ch]==q;p=st[p].fa){
                st[p].next[ch]=r;
            }
            st[cur].fa = st[q].fa = r;
        }
        last = cur;
    }
};

SAM sam;
```

## 广义后缀自动机

```cpp
//广义后缀自动机，其实就是插入多个字符串的后缀自动机，但只能离线build
//luogu p6139
//后缀自动机的性质都可以用过来
struct State{
    int fa,len,next[26];//似乎有些编译器next是保留字，需要注意
};

class GSA{
public:
    State st[MAXN<<1];
    int cnt = 1;//起始节点编号为1
    
    int insert(int last, int ch){
        //传入的是即将插入的字符的父节点编号，以及该字符
        //只用在build里，不要直接把字符串插入，应该先insertTrie再build
        int cur = st[last].next[ch];
        int p = 0;
        st[cur].len = st[last].len + 1;
        
        for(p=st[last].fa;p && !st[p].next[ch];p=st[p].fa)
            st[p].next[ch] = cur;
        
        int q = st[p].next[ch];
        if(q==0){
            st[cur].fa = 1;
        }
        else if(st[p].len+1==st[q].len){
            st[cur].fa = q;
        }
        else{
            int r = ++cnt;
            st[r].fa = st[q].fa;
            for(int i=0;i<26;i++)
                if(st[st[q].next[i]].len)
                    st[r].next[i] = st[q].next[i];
            st[r].len = st[p].len+1;
            for(;p && st[p].next[ch]==q;p=st[p].fa)
                st[p].next[ch] = r;
            st[cur].fa = st[q].fa = r;
        }
        return cur;
        //返回插入节点编号
    }
    
    void build(){
        std::queue<pii> qu;
        for(int i=0;i<26;i++){
            if(st[1].next[i]) qu.push({1,i});
        }
        while(!qu.empty()){
            int fa = qu.front().first;
            int ch = qu.front().second;
            qu.pop();
            int p = insert(fa,ch);
            for(int i=0;i<26;i++){
                if(st[p].next[i]) qu.push({p,i});
            }
        }
    }
    
    void insertTrie(std::string const & str){
        int cur = 1;
        for(auto c:str){
            if(!st[cur].next[c-'a']){
                st[cur].next[c-'a']=++cnt;
            }
            cur = st[cur].next[c-'a'];
        }
    }
};

GSA gsa;

int main(){
    int n;
    std::cin>>n;
    
    for(int i=1;i<=n;i++){
        std::string str;
        std::cin>>str;
        gsa.insertTrie(str);
    }
    gsa.build();

    return 0;
}
```

## 回文字动机

```cpp
//回文字动机，复杂度：线性
//luogu p5496
struct State{
    int len, fail, next[26];
};

//PAM的每一个状态代表原字符串的一个回文子串，每一个转移代表从当前状态字符串的前后同时加一个相同字符后的状态。可以接受其所有回文子串。除了奇根都是可行状态（当然不能为空时偶根不可行）
//fail指针指向该状态的最长回文真后缀。例如ayawaya就指向aba。总体和AC自动机的fail转移很像，都是没有ch的转移，则看fail有没有ch的转移，若fail没有则看fail[fail]的，以此类推。
//回文串分为奇长度和偶长度的，所以PAM有奇根和偶根，偶根的fail指向奇根，奇根不可能失配。
//PAM和SAM一样是动态构建的。

class PAM{
public:
    int last,cnt,pos;
    //last代表上个前缀的最长回文后缀的状态
    State st[MAXN];//最多有n+2个状态和n个转移
    std::string str;
    
    void init(std::string const & s){
        last = 1,pos = -1;
        cnt = 2;//起始两个根为1奇根，2偶根，len分别为-1和0
        st[1].len = -1, st[2].len = 0, st[2].fail = 1;
        str = s;
    }
    
    int up(int p){
        while(str[pos-1-st[p].len]!=str[pos])
            p = st[p].fail;
        return p;
    }
    
    void insert(int ch){
        pos++;
        ch = ch-'a';
        int p = up(last);
        int & q = st[p].next[ch];
        if(!q){
            q=++cnt;
            st[q].len = st[p].len+2;
            st[q].fail = p==1 ? 2 : st[up(st[p].fail)].next[ch];
        }
        last = q;
    }
};

PAM pam;

int main(){
    std::string str
	std::cin>>str;
	pam.init(str);
    for(auto c:str){
        pam.insert(c);
    }

    return 0;
}
```

## 序列自动机

```cpp
//序列自动机，复杂度 O(n|S|)
//luogu p5826
//字符集很大时需要用可持久化线段树维护next
struct State{
    int next[26];  
};

//SQA接受字符串的所有子序列（不需要连续，可以为空）

class SQA{
public:
    State st[MAXN];
    
    void build(std::string const & str){
        int nxt[26];
        std::fill(nxt,nxt+26,0);
        int n = str.size();
        
        for(int i=n-1;i>=0;i--){
            nxt[str[i]-'a'] = i+2;//1号节点是起始空节点
            for(int ch=0;ch<26;ch++){
                st[i+1].next[ch] = nxt[ch];
            }
        }
    }
    
    bool query(std::string const & str){
        //查询str是否是子序列（可以不连续）
        int p = 1, n=str.size();
        for(int i=0;i<n;i++){
            p = st[p].next[str[i]-'a'];
            if(p==0) return false;
        }
        
        return true;
    }
};

SQA sqa;
```

# 数论

## 欧几里得算法

```cpp
//复杂度 logn
//luogu B3736
//gcd是可重复贡献的，gcd(x,x)=x，可以用st表维护区间gcd
//x*y=gcd(x,y)*lcm(x,y)，lcm是最小公倍数
#include <iostream>

inline int gcd(int a,int b){
    return b==0 ? a : gcd(b, a%b);
}

int main(){
    int x,y,z;
    std::cin>>x>>y>>z;
    std::cout<<gcd(gcd(x,y),z)<<"\n";
    
    return 0;
}

```

## 扩展欧几里得

```cpp
//复杂度 logn
//求解ax+by=c的一组解，c不是gcd(a,b)的倍数则无解
//扩展欧几里得
//luogu P5656
#include <iostream>
#include <cmath>

using LL = long long;

LL exgcd(LL a,LL b,LL& x,LL& y){
    //求出的是ax+by=gcd(a,b)的一组解，等于c的需要转换一下
    if(b==0){
        x = 1;
        y = 0;//此时ax+by=gcd(a,b)中b=0，任何数与0的最大公约数是他本身，所以ax+0y=a，x=1 y=0
        return a;
    }
    LL d = exgcd(b,a%b,y,x);
    y -= (a/b)*x;
    return d;
}

int main(){
    int T;
    std::cin>>T;
    while(T--){
        LL a,b,c;
        std::cin>>a>>b>>c;
        LL x0=0, y0=0;
        LL d = exgcd(a,b,x0,y0);//d=gcd(a,b)
        if(c%d!=0){//c不是gcd(a,b)的倍数则无解
            std::cout<<"-1\n";
            continue;
        }
        LL x1 = x0*c/d, y1 = y0*c/d;//ax+by=gcd(a,b)的一组解转化为ax+by=c的一组解
        //通解的形式为x=x1+s*dx, y=y1-s*dy
        //其中s为任意整数，dx=b/d, dy = a/d;
        LL dx = b/d, dy = a/d;
        LL l = std::ceil((-x1+1.0)/dx);
        LL r = std::floor((y1-1.0)/dy);
        //x>0,y>0时，s的取值为[l,r]中的整数，若l>r，则说明不存在正整数解
        if(l>r){
            std::cout<<x1+l*dx<<" ";//所有解中x的最小正整数值
            std::cout<<y1-r*dy<<"\n";//所有解中y的最小正整数值
        }
        else{
            std::cout<<r-l+1<<" ";//正整数解的个数
            std::cout<<x1+l*dx<<" ";//正整数解中x的最小值
            std::cout<<y1-r*dy<<" ";//正整数解中y的最小值
            std::cout<<x1+r*dx<<" ";//正整数解中x的最大值
            std::cout<<y1-l*dy<<"\n";//正整数解中y的最大值
        }
        
    }
    return 0;
}
```

## 欧拉筛

TODO: 用模板元编程实现编译期算素数

```cpp
//复杂度 n
//欧拉筛, luogu p3383

int const MAXN = 1e8+5;

std::vector<int> prime;
bool isnp[MAXN];

void sieve(int n){
    for(int i=2;i<=n;i++){
        if(!isnp[i]) prime.push_back(i);
        for(auto p:prime){
            if(i*p>n) break;
            isnp[i*p] = 1;
            if(i%p==0) break;
        }
    }
}
```

## Miller-Rabin素数测试

```cpp
//对数 n 进行 k 轮测试的时间复杂度是 klog^3(n)
//miller-rabin
//loj 143
//通过测试的有可能是素数，不通过的一定不是素数
#include <iostream>
#include <ctime>
#include <cstdio>
#include <cstdint>

using LL = __int128;//本题数据范围过大，防止运算中爆掉

LL const test_time = 10;

LL qPowMod(LL n, LL p, LL m){
    LL res = 1;
    while(p>0){
        if(p&1){
            res = (res * n)%m;
        }
        n = (n*n)%m;
        p>>=1;
    }
    return res;
}

bool millerRabin(LL n) {
    if (n < 3 || n % 2 == 0) return n == 2;
    LL a = n - 1, b = 0;
    while (a % 2 == 0) a /= 2, ++b;
    // test_time 为测试次数,建议设为不小于 8
    // 的整数以保证正确率,但也不宜过大,否则会影响效率
    for (LL i = 1, j; i <= test_time; ++i) {
        LL x = rand() % (n - 2) + 2;
        LL v = qPowMod(x, a, n);
        if (v == 1) continue;
        for (j = 0; j < b; ++j) {
            if (v == n - 1) break;
            v = v * v % n;
        }
        if (j == b) return 0;
    }
    return 1;
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    
    srand(time(NULL));
    long long n;
    while(std::cin>>n){
        if(millerRabin(n)){
            std::cout<<"Y\n";
        }
        else{
            std::cout<<"N\n";
        }
    }
    return 0;
}

```

## 乘法逆元

```cpp
//复杂度 扩展欧几里得法和费马小定理法都是logn
//乘法逆元
//ax≡1(mod b)，x为a在乘法意义上的逆元，记作a^(-1)，或者inv(a)
//用扩展欧几里得法的角度看，就是求ax+by=1的整数解
//快速幂法利用费马小定理，需要b为素数，并且疑似比exgcd常数大
//luogu P3811，会TLE，需要线性求逆元
//但loj 110不会TLE

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

int exgcd_inv(int a, int b){
    //a在模b意义下的逆元
    int x,y;
    int d = exgcd(a,b,x,y);
    if(d!=1){//显然a，b要互质才会有逆元
        return -1;
    }
    else{
        return (x+b)%b;//实际上是为了防止出现x为负数的情况
    }
}

int qPowMod(int x, int p, int mod){
    //x^p % m
    int ans = 1;
    while(p){
        if(p&1){
            ans = (ans*x)%mod;
        }
        x = (x*x)%mod;
        p>>=1;
    }
    return ans;
}

int fermat_inv(int a, int b){//a在模b意义下的逆元
    return qPowMod(a,b-2,b);
}
```

## 线性求逆元

```cpp
//线性求逆元，对于1~n这些数，复杂度总共n
//luogu p3381, loj 110
#include <iostream>

using LL = long long;

int const MAXN = 3000005;
LL inv[MAXN];

void getinv(LL n, LL m){
    //求1~n中，每个数在模m意义下的乘法逆元
    inv[1] = 1;
    for(LL i=2;i<=n;i++){
        //inv[i] = -(b/i)*inv[b%i]; //这样写会出现负数
        inv[i] = (LL)(m-m/i)*inv[m%i]%m;
    }
}
```

## 线性同余方程

```cpp
//复杂度 logn
//ax≡c (mod b)求解x
//和ax+by=c等价
//luogu p1082
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
    //d是a,b的最大公约数，如果无解d==-1
    //下面输出的是最小整数解
    int t = b/d;
    x = (x%t+t)%t;
    cout<<x<<endl;

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

其中$r_i$两两互质，如果不满足则需要扩展CRT

```cpp
//中国剩余定理 复杂度 klogk
//luogu p1495
typedef long long ll;

const int MAXN = 10005;

ll exgcd(ll a, ll b, ll &x, ll &y){
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

ll exgcd_inv(ll a, ll b){
    ll x,y;
    ll d = exgcd(a,b,x,y);
    return (x+b)%b;
}

class CRT{
public:
    ll ax[MAXN],rx[MAXN];//每个方程的形式为x≡ai(mod ri)，要求ri互质
    int k=0;//k个方程
    
    void add(ll a, ll r){
        ax[++k] = a;
        rx[k] = r;
    }
    
    ll solve(){
        ll n=1, ans=0;
        for(int i=1;i<=k;i++){
            n = n * rx[i];
        }
        for(int i=1;i<=k;i++){
            ll m = n/rx[i];
            ans = (ans+ax[i]*m*exgcd_inv(m,rx[i]))%n;
        }
        
        return ans;
    }
};

CRT crt;
```

## 积性函数

积性函数是数论函数的一种。数论函数则是定义域为正整数的函数。

若函数$f(n)$满足$f(1)=1$且$\forall x,y\in N^*,gcd(x,y)=1$都有$f(xy)=f(x)f(y)$，则$f(n)$为积性函数

如果不要求$gcd(x,y)=1$也有这个性质的函数叫完全积性函数。同理可知加性函数的定义。

**性质**

1. 若$f(x),g(x)$均为积性函数，则以下函数也是积性函数
    1. $h(x)=f(x^p)$
    2. $h(x)=f^p(x)$
    3. $h(x)=f(x)g(x)$
    4. $h(x)=\sum_{d|x}f(d)g(x/d)$（即狄利克雷卷积）
    5. 其逆元
2. 若$f$是积性函数，且在算术基本定理中$n=\prod^{m}_{i=1}p_i^{c_i}$，则$f(n)=\prod^{m}_{i=1}f(p_i^{c_i})$。如果$f$是完全积性函数，则$f(n)=\prod^{m}_{i=1}f(p_i)^{c_i}$

**例子**

1. $\varepsilon(n)=[n=1]$，单位函数，括号是艾弗森括号，是完全积性。
2. $id_k(n)=n^k$，幂函数，是完全积性。$k=1$是恒等函数$id(n)=n$，$k=0$是常数函数$1(n)=1$
3. $\sigma_k(n)=\sum_{d/n}d^k$，除数函数。当$k=1$时，为因数和函数$\sigma(n)$，当$k=0$时为因数个数函数$\sigma_0(n)$
4. 欧拉函数
5. 莫比乌斯函数

## 欧拉函数 TODO：习题

$1\sim N$中与$N$互质的数的个数被称为欧拉函数，记为$\varphi(N)$

若在算数基本定理中，$N=p_1^{c_1}p_2^{c_2}\cdots p_m^{c_m}$，则

$$
\varphi(N)=N\times\dfrac{p_1-1}{p_1}\times\dfrac{p_2-1}{p_2}\times\cdots\times\dfrac{p_m-1}{p_m}
$$

欧拉函数也可以写成艾弗森括号的形式为

$$
\sum^n_{i=1}[\gcd(i,n)=1]
$$

```cpp
//复杂度 根号n
int phi(int n){
    int ans = n;
    for(int i=2;i*i<=n;i++){
        if(n%i==0){
            ans = ans/i*(i-1);
            while(n%i==0) n/=i;
        }
    }
    if(n>1) ans = ans/n*(n-1);
    return ans;
}
```

欧拉函数有以下性质

1. $\forall n>1$，$1\sim n$中与$n$互质的数的和为$n\times\varphi(n)/2$
2. 若$a,b$互质，则$\varphi(ab)=\varphi(a)\varphi(b)$。也就是说欧拉函数是积性函数
3. 设$p$为质数，若$p|n$且$p^2|n$，则$\varphi(n)=\varphi(n/p)\times p$
4. 设$p$为质数，若$p|n$但不满足$p^2|n$，则$\varphi(n)=\varphi(n/p)\times (p-1)$
5. $\sum_{d|n}\varphi(d)=n$
6. 若$p$是质数，则$\varphi(p^n)=p^{n-1}(p-1)$
7. 若$a|x$，则$\varphi(ax)=a\varphi(x)$

```cpp
//求1-N的所有欧拉函数值，使用筛法，埃氏筛复杂度NloglogN，线性筛复杂度N，这里是线性筛
std::vector<int> prime;
bool isnp[MAXN];
int phi[MAXN];

void euler(int n){
    phi[1] = 1;
    for(int i=2;i<=n;i++){
        if(!isnp[i]){
            prime.push_back(i);
            phi[i] = i-1;
        }
        for(auto p:prime){
            if(i*p>n) break;
            isnp[i*p] = 1;
            
            if(i%p==0){
                phi[i*p] = phi[i] * p;
                break;
            }
            else{
                phi[i*p] = phi[i] * phi[p];
            }
        }
    }
}
```

## 狄利克雷卷积

两个数论函数$f(n),g(n)$的狄利克雷卷积定义为

$$
(f\ast g)(n) = \sum_{xy=n}f(x)g(y)
$$

也可以写作

$$
(f\ast g)(n) = \sum_{d|n}f(d)g(n/d)
$$

**性质**

1. 两个积性函数的卷积还是积性函数
2. $(f\ast 1)(n) = \sum_{d|n}f(d)$
3. $(id_k\ast 1)(n)=\sum_{d|n}d^k=\sigma_k$
4. $\varphi\ast 1=id$
5. 满足交换率、结合律、对加法的分配律
6. 等式性质，$f=g$的充要条件是$f\ast h = g\ast h$，其中数论函数$h(1)\neq 0$
7. 幺元是$\varepsilon$，即$f\ast \varepsilon=f$
8. 逆元，即满足$f\ast g=\varepsilon$的函数$g$为（显然$f(1)\neq 0$才有逆元）

$$
g(x)=\dfrac{\varepsilon(x)-\sum_{d|x,d\neq 1}f(d)g(x/d)}{f(1)}
$$
9. 积性函数一定有逆元，且逆元也是积性函数。

## 莫比乌斯反演 TODO：习题

莫比乌斯函数是常数函数$1$的逆元。即

$$
\mu(n) = \left\{\begin{matrix}
1, & n=1 \\
(-1)^m  & n=p_1p_2\cdots p_m\\
0  & \text{otherwise}
\end{matrix}\right.
$$

其中第二个条件就是$n$质因数分解后每个因子的次数是$1$

莫比乌斯反演公式即为

$$
g(n)=\sum_{d|n}f(d)\Leftrightarrow f(n) = \sum_{d|n}\mu(d)g(n/d)
$$

用狄利克雷卷积来写就是

$$
f\ast 1=g\Leftrightarrow f=g\ast \mu
$$

还有一种形式（倍数形式，之前的叫因数形式）是

$$
g(n) = \sum_{n|N}f(N)\Leftrightarrow f(n) = \sum_{n|N}\mu(N/n)g(N)
$$

**性质**

1. 是积性函数
2. $\sum_{d|n}\mu(d)=\varepsilon(n),\mu\ast 1=\varepsilon$
3. $\sum_{d|\gcd(i,j)}\mu(d)=[\gcd(i,j)=1]$ 

```cpp
//线性筛求莫比乌斯函数，复杂度n
int mu[MAXN];
std::vector<int> prime;
bool isnp[MAXN];

void mobius(int n){
    mu[1] = 1;
    for(int i=2;i<=n;i++){
        if(!isnp[i]){
            prime.push_back(i);
            mu[i] = -1;
        }
        
        for(auto p:prime){
            if(i*p>n) break;
            isnp[i*p] = 1;
            if(i%p==0){
                mu[i*p] = 0;
                break;
            }
            else{
                mu[i*p] = mu[i] * mu[p];
            }
        }
    }
}
```

## 数论分块

在计算形如

$$
\sum^n_{i=1}f(i)g(\left \lfloor \dfrac{n}{i} \right \rfloor )
$$

的式子时，注意到$\left \lfloor \dfrac{n}{i} \right \rfloor$的取值个数会比$n$小很多，我们可以将取值相同的合在一起计算。如果可以在$O(1)$内计算$f(l)+\cdots+f(r)$（比如等差数列求和公式）或者有$f$的前缀和时，数论分块可以在$O(\sqrt{n})$计算和式的值。

**定理1**

$$
\forall a,b,c\in Z, \left \lfloor \dfrac{a}{bc} \right \rfloor=\left \lfloor \dfrac{\left \lfloor \dfrac{a}{b} \right \rfloor}{c} \right \rfloor
$$

**定理2**

当$i$取正整数且$i\leq n$时，$\left \lfloor \dfrac{n}{i} \right \rfloor$的不同取值的数量不超过$\left \lfloor 2\sqrt n \right \rfloor$

**定理3**

使得式子

$$
\left \lfloor \dfrac{n}{i} \right \rfloor = \left \lfloor \dfrac{n}{j} \right \rfloor
$$

成立的最大的满足$i\leq j\leq n$的$j$值是$\left \lfloor \dfrac{n}{\left \lfloor \dfrac{n}{i} \right \rfloor} \right \rfloor$。也就是这个分块的右端点。

```cpp
//UVA 11526
//数论分块模板 复杂度sqrt n
//要求计算i=1~n, ans = ans+n/i
void solve(){
    LL n;
    std::cin>>n;
    LL ans = 0;
    
    LL l = 1, r = 0;
    
    while(l<=n){
        r = n/(n/l);
        ans += (r-l+1) * (n/l);
        l = r+1;  
    }
    std::cout<<ans<<"\n";
}
```

注意如果不是$\left \lfloor \dfrac{n}{i} \right \rfloor$而是某个$\left \lfloor \dfrac{k}{i} \right \rfloor$，要注意特判判$k/l$等于$0$，以及$r$要特判不能超过$n$。

当有多个取整$\left \lfloor \dfrac{a_1}{i} \right \rfloor,\left \lfloor \dfrac{a_2}{i} \right \rfloor,\cdots$ 时，我们取的右端点就变成每一个块的右端点的最小值。

## 杜教筛

杜教筛可以在$O(n^{2/3})$的时间复杂度下求得一类数论函数（不一定需要积性）$f(n)$的前缀和。

我们需要找到一个数论函数$g(n)$，使得$g(n)$和$f\ast g(n)$的前缀和都很容易求出（最好在$O(1)$），那我们就能以低于线性复杂度的算法求出$f(n)$的前缀和$S(n)$。证明略，结论为：

$$
g(1)S(n) = \sum^n_{i=1}(f\ast g)(i)-\sum^n_{i=2}g(i)S(\left \lfloor \dfrac{n}{i} \right \rfloor)
$$

**筛莫比乌斯函数**

之前我们学到$\mu\ast 1=\varepsilon$，所以我们自然的令$g(n)=1$，得到

$$
S(n)=\sum^n_{i=1}\varepsilon(i)-\sum^n_{i=2}S(\left \lfloor \dfrac{n}{i} \right \rfloor) = 1-\sum^n_{i=2}S(\left \lfloor \dfrac{n}{i} \right \rfloor)
$$

此时如果我们直接用数论分块去算$S(n)$，我们的算法复杂度是$O(n^{3/4})$，但是如果我们用线性筛预处理前$n^{2/3}$的$S(n)$的值，就可以优化复杂度到$O(n^{2/3})$，通常我们还会开一个哈希表去维护大于$n^{2/3}$的值来优化。

**筛欧拉函数**

同样取$g(n)=1$，有$\varphi(n)\ast 1=id$

$$
S(n) = \sum^n_{i=1}id-\sum^n_{i=2}S(\left \lfloor \dfrac{n}{i} \right \rfloor) = \dfrac{n(1+n)}{2}-\sum^n_{i=2}S(\left \lfloor \dfrac{n}{i} \right \rfloor)
$$

跟之前可以说没什么区别。

给出求欧拉函数和莫比乌斯函数前缀和的例子代码。这里筛法一次把两个函数都筛了，其他不难理解。注意要用unordered_map以及数论分块的时候$l$从$2$开始。以及洛谷上这题数据范围为2^31，要筛出大概前170w个数。我筛了200w也过了，分类讨论，前200w直接返回，大于200w的如果在map里就返回，否则递归计算后放入map里。

```cpp
//杜教筛 复杂度n^(2/3)
//luogu p4213
int const MAXN = 2000005;

int mu[MAXN];
LL phi[MAXN];
std::vector<int> prime;
bool isnp[MAXN];
LL sum_mu[MAXN],sum_phi[MAXN];
std::unordered_map<LL,LL> mp_mu,mp_phi;

void sieve(int n=MAXN-1){
    mu[1] = 1;
    phi[1] = 1;
    for(int i=2;i<=n;i++){
        if(!isnp[i]){
            prime.push_back(i);
            mu[i] = -1;
            phi[i] = i-1;
        }
        
        for(auto p:prime){
            if(i*p>n) break;
            isnp[i*p] = 1;
            if(i%p==0){
                mu[i*p] = 0;
                phi[i*p] = phi[i] * p;
                break;
            }
            else{
                mu[i*p] = mu[i] * mu[p];
                phi[i*p] = phi[i] * phi[p];
            }
        }
    }
    for(int i=1;i<=n;i++){
        sum_mu[i] = sum_mu[i-1]+mu[i];
        sum_phi[i] = sum_phi[i-1]+phi[i];
    }
}

LL sum1(LL n){
    if(n<MAXN){
        return sum_phi[n];
    }
    if(mp_phi.count(n)){
        return mp_phi[n];
    }
    LL l=2, r=0;
    LL ret = n*(1+n)/2;
    while(l<=n){
        r = n/(n/l);
        ret -= (r-l+1)*sum1(n/l);
        l = r+1;
    }
    mp_phi[n] = ret;
    return ret;
}

LL sum2(LL n){
    if(n<MAXN){
        return sum_mu[n];
    }
    if(mp_mu.count(n)){
        return mp_mu[n];
    }
    LL l=2, r=0;
    LL ret = 1;
    while(l<=n){
        r = n/(n/l);
        ret -= (r-l+1)*sum2(n/l);
        l = r+1;
    }
    mp_mu[n] = ret;
    return ret;
}

void solve(){
    LL n;
    std::cin>>n;
    std::cout<<sum1(n)<<" "<<sum2(n)<<"\n";
    
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);

    sieve();

	int T;
	std::cin>>T;
	//T=1;
	while(T--){
	    solve();
	}

    return 0;
}
```

# 图论

## 存图

### 邻接矩阵

```cpp
int graph[MAXN][MAXN];
//加边删边、访问很方便，a->b，权值为w，则graph[a][b]=w
//空间占用大，并且不能存重边
```

### 邻接表（vector版）

```cpp
struct Edge{
    int v,w;//下一点，权
};

std::vector<Edge> edges[MAXN];

//加边u->v,权值w
edges[u].push_back(w);
//访问只能遍历u所连的出边
for(auto x:edges[u]){
    std::cout<<x.v<<" "<<x.w<<"\n";
}

//缺点是，删边难，以及清空边复杂度过高。快速清边见链式前向星传统数组版
```

### 链式前向星（vector版）

```cpp
struct Edge{
    int v;LL w;//指向的点，容量
    Edge(int v_, LL w_):v(v_),w(w_){}
};

std::vector<Edge> edges;
std::vector<std::vector<int> > graph(MAXN);
//常用于网络流，例如加u->v，权值为w，及其反向边v->w，权值为0
graph[u].push_back(edges.size());
edges.push_back(Edge(v,w));
graph[v].push_back(edges.size());
edges.push_back(Edge(u,0));
//遍历u的边时
for(auto x:graph[u]){
    auto e=edges[x];
    auto v=e.v, w=e.w;
    //e的反向边就是edges[x^1]
}

//清空某个点连出的所有边时，graph[u].clear()，不需要管edges的size
//清楚整个图时edges.clear()，graph要对每个下标clear
//这种清楚方式复杂度比传统数组版高很多，需要反复建图时不推荐使用。
```

### 链式前向星（传统数组版）

```cpp
struct Edge{
    int v,w,next;//指向的点，边权，下一条边
};

Edge edges[MAXM];//存无向图记得开两倍
int head[MAXN],cnt;

inline void add(int u, int v, int w){
    edges[++cnt].w = w;
    edges[cnt].v = v;
    edges[cnt].next = head[u];//把下一条边设置为当前起点的第一条边
    head[u] = cnt;//该边称为当前起点的第一条边
}

//遍历，与vector版不同，vector版按加入先后顺序遍历，而这里是反向顺序遍历。绝大多数情况不影响
//例如遍历1号节点所连的边
for(int e=head[1];e;e=edges[e].next){
    std::cout<<edges[e].v<<" "<<edges[e].w<<"\n";
}

//当需要清空某个点的所有连出边时, head[u] = 0，不需要管cnt
//清空全图时，cnt = 0, 对于所有点head = 0， 由于还可以用memset，比vector版更是快了不少
```

## 最短路

### Dijkstra

```cpp
//复杂度 优先队列实现为mlogm
//dijkstra，单源最短路
//luogu p4779
#include <iostream>
#include <cstring>
#include <vector>
#include <queue>
#define MAXN 500005
#define MAXINT 0x7fffffff

struct Edge{
    int v,w;//下一点，权
    Edge(int v_, int w_):v(v_),w(w_){}
};

struct Node {
    int dis, u;//存储起点到u点的距离
    Node(int dis_, int u_):dis(dis_),u(u_){};
    bool operator>(Node const & a) const { return dis > a.dis; }
};

std::vector<Edge> graph[MAXN];
int dis[MAXN];
bool tag[MAXN];

std::priority_queue<Node, std::vector<Node>, std::greater<Node> > pq;

void init(int n){
    for(int i=1;i<=n;i++){
        dis[i] = MAXINT;
        //初始化为无限远
        tag[i] = 0;
        graph[i].clear();
    }
    while(!pq.empty()) pq.pop();
}

void dijk(int s){
    dis[s]=0;
    pq.push(Node(0,s));

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
                pq.push(Node(dis[v],v));
            }
        }
    }
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n,m,s;
    std::cin>>n>>m>>s;
    //点数，边数，起点
    init(n);

    for(int i=1;i<=m;i++){
        int u,v,w;
        std::cin>>u>>v>>w;//起点，终点，边权
        graph[u].push_back(Edge(v,w));
    }
    dijk(s);

    for(int i=1;i<=n;i++){
        std::cout<<dis[i]<<" ";
    }
    return 0;
}
```

### Bellman-Ford

```cpp
//复杂度 nm
//bellman-ford, 单源最短路
//luogu p4779，有一个点TLE
#include <iostream>
#include <vector>
#include <cstring>

int const MAXN = 100005;
int const INF = 0x6fffffff;

struct Edge{
    int v,w;//下一点,权
    Edge(int v_, int w_):v(v_),w(w_){}
};

int dis[MAXN];
std::vector<Edge> graph[MAXN];

void init(int n){
    for(int i=1;i<=n;i++){
        dis[i]=INF;
        graph[i].clear();
    }
}

bool BF(int n, int s){
    //如果不存在最短路就返回0，否则返回1
    dis[s] = 0;
    bool flag = 1;
    for (int i=1;i<=n;i++){//松弛n-1轮，若第n轮还能松弛，就说明有负环
        flag = 1;
        for(int u=1;u<=n;u++){//这里看似是两层循环，实际上总数是边数，整个算法的复杂度是mn
            for (auto e : graph[u]){
                int w=e.w,v=e.v;
                if(dis[v]>dis[u]+w){
                    dis[v]=dis[u]+w;
                    flag = 0;
                }
            }
        }
        if(flag){
            break;
        }
    }
    
    return flag;
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n,m,s;//点数，边数，起点
    std::cin>>n>>m>>s;
    init(n);

    for(int i=1;i<=m;i++){
        int u,v,w;
        std::cin>>u>>v>>w;
        //起点，终点，边权
        graph[u].push_back(Edge(v,w));
    }
    
    BF(n,s);

    for(int i=1;i<=n;i++){
        std::cout<<dis[i]<<" ";
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
//spfa 单源最短路
//luogu P3371
#include <iostream>
#include <vector>
#include <queue>

int const MAXN = 100005;
int const INF = 0x5fffffff;

struct Edge{
    int v,w;
    Edge(int v_, int w_):v(v_),w(w_){}
};

int dis[MAXN];//距离
int cnt[MAXN];//算到达本节点所要经过的边数，若cnt>=n，则说明有负权环
bool tag[MAXN];//用于判断是否为上次松弛过的节点的边所连的点

std::queue<int> qu;
std::vector<Edge> graph[MAXN];

void init(int n){
    while(!qu.empty()) qu.pop();
    for(int i=1;i<=n;i++){
        dis[i] = INF;
        cnt[i] = 0;
        tag[i] = 0;
        graph[i].clear();
    }
}

bool SPFA(int n, int s){
    //如果不存在最短路就返回0，否则返回1
    dis[s] = 0;
    tag[s] = 1;
    qu.push(s);
    bool flag = 1;

    while(!qu.empty()){
        if(!flag) break;
        int u = qu.front();
        qu.pop();
        tag[u]=0;
        for(auto e : graph[u]){
            int v = e.v, w = e.w;
            if(dis[v]>dis[u]+w){
                dis[v]=dis[u]+w;
                cnt[v]=cnt[u]+1;
                if(cnt[v]>=n) {
                    flag = 0;
                    break;
                }
                if(!tag[v]){
                    qu.push(v);
                    tag[v]=1;
                }
            }
        }
    }
    return flag;
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n,m,s;
    std::cin>>n>>m>>s;
    init(n);

    for(int i=1;i<=m;i++){
        int u,v,w;
        std::cin>>u>>v>>w;
        //起点，终点，边权
        graph[u].push_back(Edge(v,w));
    }

    SPFA(n,s);
    for(int i=1;i<=n;i++){
        if(dis[i]!=INF){
            std::cout<<dis[i]<<" ";
        }
        else{
            std::cout<<"2147483647 ";//根据luogu P3371要输出这个数
        }
    }

    return 0;
}

```

### Floyd

```cpp
//复杂度 n^3
//floyd全源最短路
//luogu p5905，由于不能判断负环和速度慢，会wa和tle一些
//floyd虽然不能处理负环但是可以接受负边
#include <iostream>
#include <cstring>

using LL = long long;

int const MAXN = 3005;
LL const INF = 1e17; //不能设置为int的最大值，否则后面加法可能导致溢出
LL graph[MAXN][MAXN];

int main(){
    int n,m;//点数，边数
    std::cin>>n>>m;
    for(int i = 1;i<=n;i++){
        for(int j = 1;j<=n;j++){
            graph[i][j] = INF;
        }
    }
    for(int i=1;i<=m;i++){
        int u,v;LL w;
        std::cin>>u>>v>>w;//起点，终点，边权
        graph[u][v] = std::min(graph[u][v], w);//处理重边
    }
    for(int i = 1;i<=n;i++){
        graph[i][i] = 0;
    }

    for(int k=1;k<=n;k++){
        for(int i=1;i<=n;i++){
            for(int j=1;j<=n;j++){
                graph[i][j] = std::min(graph[i][j],graph[i][k]+graph[k][j]);
            }
        }
    }
    
    for(int i=1;i<=n;i++){
        LL res=0;
        for(LL j=1;j<=n;j++){
            if(graph[i][j]>1e9) graph[i][j] = 1e9;
            res += j*graph[i][j];
        }
        std::cout<<res<<"\n";
    }

    return 0;
}

```

### Johnson TODO

## 差分约束

给出一组不等式

$$
\left\{\begin{matrix}
x_{c_1}-x_{c_1'}\leq y_1 \\
x_{c_2}-x_{c_2'}\leq y_2 \\
\vdots \\
x_{c_m}-x_{c_m'}\leq y_m
\end{matrix}\right.
$$

其中一共有$n$个未知数$x_1,x_2,\cdots,x_n$，$m$个不等式，求一组可行解。

我们连边，连一条$x_{c_i'}$到$x_{c_i}$，权值为$y_i$的边。然后增加$n+1$号节点，从它到所有点连一条权值为$0$的边。然后以$n+1$为源点求到各点的最短路，这个最短距离dis\[i\]就是$x_i$的一个解。

当然，出现负环就无解。

不难理解，$x_1,x_2,\cdots,x_n$全部加上或减去同一个数，仍然是可行解。

之前我们假设dis\[n+1\]=0，如果我们设置dis\[n+1\] = w，那么我们求得的就是$x_1,x_2,\cdots,x_n\leq w$的一组解。实际上，可以证明这个解是满足$x_1,x_2,\cdots,x_n\leq w$的最大解（每个变量能取得的最大值）。

如果题目上的约束条件全部变为$\geq$型，我们要求满足$x_1,x_2,\cdots,x_n\geq w$的最小解，则只需要求最长路即可。对于Bellman-Ford和SPFA来说，初始化dis为-INF，然后颠倒比较符号即可。

题目中不总是给出$x_1-x_2\leq y$，但我们可以转化

- $x_1-x_2\geq y\Rightarrow x_2-x_1\leq -y$
- $x_1-x_2=y\Rightarrow x_1-x_2\leq y \wedge x_2-x_1\leq -y$
- $x_1-x_2<y\Rightarrow x_1-x_2\leq y-1$（要求取值只能是整数）
- $x_1-x_2>y\Rightarrow x_2-x_1\leq -y-1$（要求取值只能是整数）

```cpp
//差分约束，复杂度同SPFA
//luogu p5960
//使用介绍见markdown
#include <iostream>
#include <vector>
#include <queue>

int const MAXN = 5005;
int const INF = 0x5fffffff;

struct Edge{
    int v,w;
    Edge(int v_, int w_):v(v_),w(w_){}
};

int dis[MAXN];//距离
int cnt[MAXN];//算到达本节点所要经过的边数，若cnt>=n，则说明有负权环
bool tag[MAXN];//用于判断是否为上次松弛过的节点的边所连的点

std::queue<int> qu;
std::vector<Edge> graph[MAXN];

void init(int n){
    while(!qu.empty()) qu.pop();
    for(int i=1;i<=n;i++){
        dis[i] = INF;
        cnt[i] = 0;
        tag[i] = 0;
        graph[i].clear();
    }
}

bool SPFA(int n, int s){
    //如果不存在最短路就返回0，否则返回1
    dis[s] = 0;
    tag[s] = 1;
    qu.push(s);
    bool flag = 1;

    while(!qu.empty()){
        if(!flag) break;
        int u = qu.front();
        qu.pop();
        tag[u]=0;
        for(auto e : graph[u]){
            int v = e.v, w = e.w;
            if(dis[v]>dis[u]+w){
                dis[v]=dis[u]+w;
                cnt[v]=cnt[u]+1;
                if(cnt[v]>=n) {
                    flag = 0;
                    break;
                }
                if(!tag[v]){
                    qu.push(v);
                    tag[v]=1;
                }
            }
        }
    }
    return flag;
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    int n,m;
    std::cin>>n>>m;
    init(n+1);
    
    for(int i=1;i<=m;i++){
        int v,u,w;
        std::cin>>v>>u>>w;
        graph[u].push_back(Edge(v,w));
    }
    for(int i=1;i<=n;i++){
        graph[n+1].push_back(Edge(i,0));
    }
    n++;
    
    if(!SPFA(n,n)){
        std::cout<<"NO\n";
        return 0;
    }
    for(int i=1;i<=n-1;i++){
        std::cout<<dis[i]<<" ";
    }
    

    return 0;
}
```

## 拓扑排序

```cpp
//复杂度 n
//拓扑排序, luogu B3644
#include <iostream>
#include <vector>
#include <queue>

int const MAXN = 105;

std::vector<int> graph[MAXN];
int in[MAXN];

int main(){
    int n;
    std::cin>>n;

    for(int i=1;i<=n;i++){
        while(true){
            int v;
            std::cin>>v;
            if(v==0) break;
            graph[i].push_back(v);
            in[v]++;
        }
    }
    
    std::queue<int> qu;
    for(int i=1;i<=n;i++){
        if(in[i]==0) qu.push(i);
    }
    
    while(!qu.empty()){
        int u = qu.front();
        qu.pop();
        std::cout<<u<<" ";
        for(auto v:graph[u]){
            in[v]--;
            if(in[v]==0) qu.push(v);
        }
    }

    return 0;
}


```

## 最小生成树

### Kruskal

```cpp
//复杂度 mlogm
//最小生成树Kruskal，luogu p3366
#include <iostream>
#include <algorithm>

int const MAXM = 200005;
int const MAXN = 5005;

struct Edge{
    int u,v,w;//最小生成树是在无向图上跑的，由于要排序，所以记录uvw
    bool operator<(Edge const & x) const {
        return w<x.w;
    }  
};

Edge edges[MAXM];
int find_sets[MAXN];//并查集

int find(int x){return find_sets[x]==x ? x : find_sets[x] = find(find_sets[x]);}

int main(){
    int n,m;//点数和边数

    std::cin>>n>>m;
    for(int i=1;i<=m;i++){
        std::cin>>edges[i].u>>edges[i].v>>edges[i].w;
    }
    std::sort(edges+1,edges+1+m);

    for(int i=1;i<=n;i++){
        find_sets[i]=i;
    }

    int ans = 0;
    int cnt=0;

    for(int i=1;i<=m;i++){
        int u = edges[i].u, v = edges[i].v;
        int x = find(u);
        int y = find(v);

        if(x!=y){
            ans += edges[i].w;
            find_sets[x] = y;
            cnt++;
        }
    }
    //计数，如果小于n-1则不连通
    
    if(cnt<n-1){
        std::cout<<"orz\n";
    }
    else{
        std::cout<<ans<<"\n";
    }

    return 0;
}

```

### Prim算法

```cpp
//复杂度 (m+n)logn
//最小生成树prim，luogu p3366
#include <iostream>
#include <vector>
#include <queue>

const int MAXN = 5005;
const int MAXM = 200005;
const int INF = 0x5fffffff;

struct edge{
    int v,w;

    edge(){};
    edge(int v,int w):v(v),w(w){}

    bool operator>(const edge& x) const {return w>x.w;}
};

std::vector<edge> graph[MAXN];
bool vis[MAXN];

std::priority_queue<edge, std::vector<edge>, std::greater<edge> > pq;

int main(){
    int n,m;//点数，边数
    std::cin>>n>>m;
    int ans = 0;
    int cnt = 1;

    for(int i=1;i<=m;i++){
        int u,v,w;//起点，终点，边权
        std::cin>>u>>v>>w;
        graph[u].push_back(edge(v,w));
        graph[v].push_back(edge(u,w));
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
                std::cout<<"orz\n";//不连通
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
        std::cout<<"orz\n";
    }
    else{
        std::cout<<ans<<"\n";
    }

    return 0;
}

```

## 最小树形图（朱刘算法）

```cpp
//复杂度 nm
//最小树形图，朱刘算法
//从根节点能到达其他所有点
//luogu4716
#include <iostream>

const int MAXN = 105;
const int MAXM = 10005;
const int INF = 0x7fffffff;

struct Edge{
    int u,v,w;
};

Edge edge[MAXM]; 
int vis[MAXN],id[MAXN];
int in[MAXN],pre[MAXN];

int zhuliu(int n, int m, int root){
    //返回最小树形图的边权和，如果不存在则返回-1
    int ans = 0;
    for(;;){
        for(int i=1;i<=n;i++) in[i]=INF;
        
        for(int i=1;i<=m;i++){
            int u = edge[i].u;
            int v = edge[i].v;
            if(u!=v && edge[i].w<in[v]){//遍历所有边，找到对每个点的最短入边
                in[v] = edge[i].w;
                pre[v] = u;
            }
        }
        
        for(int i=1;i<=n;i++){
            if(i!=root && in[i]==INF){
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
            if(v!=root && id[v]==-1){
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
    int n,m,root;
    std::cin>>n>>m>>root;
    //点数，边数，根节点序号
    for(int i=1;i<=m;i++){
        std::cin>>edge[i].u>>edge[i].v>>edge[i].w;
        //起点，终点，边权
    }
    std::cout<<zhuliu(n,m,root)<<"\n";
    return 0;
}
```

## 二分图判定

一张无向图是二分图，当且仅当图中不存在长度为奇数的环。

我们可以用染色法来判定。假设染成两种颜色，一个节点被染色后，所有相连节点都应该染成另一种颜色，如果有冲突，则说明不是二分图。

## 二分图匹配

### 最大匹配（匈牙利算法）

```cpp
//复杂度 nm
//luogu p3386
//求二分图最大匹配，根据定理，最大匹配=最小点覆盖，以及最小边覆盖=点数-最大匹配

//二分图是"可以将点集分为两个不相交的部分，所有边连接的两个顶点在不同的部分中"的图

//二分图的匹配：边集的任意子集的任意两条边都没有公共顶点，则这个子集是一个匹配
//二分图的最大匹配：所有匹配中边数最多的
//最小点覆盖：选最少的点，满足每条边至少有一个端点被选
//最大独立集：选最多的点，满足两两之间没有边相连
//这里的二分图是无向图
//如果最大匹配中所有点都被匹配，那么叫做完美匹配

#include <iostream>
#include <cstring>

const int MAXN = 505;
bool graph[MAXN][MAXN];
bool vis[MAXN];
int toLeft[MAXN];//标记右边节点i连到了哪个左边界点，即toLeft[i]

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
    //返回最大的边数
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

### 二分图的相关定理

Konig定理：一个二分图中的最大匹配数等于这个图中的最小点覆盖数。

最大独立集=点数-最小点覆盖。

### 最大匹配转换为网络流模型

将源点连上左边所有点，右边所有点连上汇点，容量都为1。原来的每条边从左往右连边（转成了有向有容量图），容量也为1，最大流即最大匹配。用Dinic算法求复杂度为$O(\sqrt nm)$

### 二分图最大权完美匹配（KM算法） TODO: BFS版

```cpp
//luogu p6577
//二分图的最大权匹配，必须是完美匹配才能正确运行，即左右各n个点，最大匹配有n条边。虽然KM算法必须是完美匹配才可以运行而转化为费用流则不需要，但是KM算法在稠密图上的效率会高于费用流
//随机数据O(n^3)，最坏O(n^4)，所以luogu p6577上会超时一些数据
//这主要是他卡dfs版的，bfs版的可以通过。但luogu p3967不卡dfs

//最大权匹配指二分图中边权和最大的匹配，最大权匹配不一定是最大匹配
//如果要跑多次KM算法记得把toLeft数组初始化

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
    for(int i=1;i<=pointNum;i++){
        labelL[i] = -INF;
        labelR[i] = 0;
        for(int j=1;j<=pointNum;j++) labelL[i] = std::max(labelL[i], graph[i][j]);
    }

    for(int i=1;i<=pointNum;i++){
        while(true){
            std::memset(visL,0,sizeof(visL));
            std::memset(visR,0,sizeof(visR));
            for(int j=1;j<=pointNum;j++) upd[j] = INF;
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
        std::cin>>graph[x][y];//这里是左边有n个点，右边有n个点
        //左边第x个点到右边第y个点的边权，并不是双向边
    }

    std::cout<<KM(n)<<"\n";
    for(int i=1;i<=n;i++){
        std::cout<<toLeft[i]<<" ";
    }
    std::cout<<"\n";

    return 0;
}
```

### 最大权匹配转化为费用流

新增一个源点和一个汇点，从源点向二分图的每个左部点连一条流量为1，费用为0的边；从每个右部点向汇点连一条流量为1，费用为0的边；从左部点i向右部点j连一条流量为1，费用为c的边。然后这些边的反向边也要注意连上。然后求这个网络的最大费用最大流即可。

具体而言，最大费用的求法最好不要去该内部算法实现。把费用取相反数，然后最后答案再取相反数即可。

如果要输出方案，就遍历右边点到左边点的反向边，如果实际流量w变为1了，则说明走了这条边，也就是这两个点配对。

目前为止还只能处理完美匹配的情况。因为最大费用最大流是在最大流的前提下采取计算最大费用，也就是说它会去计算最大匹配再去计算其中的最大权。而最大权匹配是只要求权最大而不用一定是最大匹配。

解决方法是把左部点连一条边到汇点，容量为1，费用为0，再去求最大费用最大流。这样如果这条边有实际流量通过（即w变成0），他是失配的。

## 动态维护二分图判定 TODO: 例题

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

#### DFS实现的Ford-Fulkerson

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

#### EdmondsKarp

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

#### Dinic TODO: 如果可能换成链式前向星

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

bool BFS(int s, int t){//BFS分层
    std::memset(level, -1, sizeof(level));
    level[s] = 0;
    cur.assign(MAXN,0);//初始化当前弧
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
    for(int i=cur[p];i<size && surplus;i++){
        int eg = graph[p][i];
        cur[p] = i;//更新当前弧
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

### ISAP算法 TODO

### 最大流最小割定理

网络流的最大流等于其所有割的最小容量。

割：从网络中选择一些边，去掉这些边后，剩下恰好两个互相不连通的分别包含源点和汇点的点集（当然其他边不去掉）。去掉的这些边就是一个割。

割的大小就是去掉的这些边的容量之和。

### 最小费用最大流

即在使流最大的前提下，最小化费用。费用是一条边的属性，一条边的总费用等于它的单位费用$\times$流过的流量。

建边的时候，反向边的容量为0，费用为相反数。

#### EK+SPFA

```cpp
//luogu P3381
//EK+SPFA的实现，复杂度为O(nmf)，即点数、边数、最大流
#include <iostream>
#include <cstring>
#include <vector>
#include <queue>

typedef long long LL;
typedef std::pair<LL,LL> pll;

const int MAXN = 5005;
const LL INF = 0xffffffff;

struct Edge{
    int v;LL w;LL c;//指向的点，容量，费用
    Edge(int v_, LL w_, LL c_):v(v_),w(w_),c(c_){}
};

std::vector<Edge> edges;
std::vector<std::vector<int> > graph(MAXN);//vector版的链式前向星
int last[MAXN];
LL flow[MAXN];
LL dis[MAXN];
bool inq[MAXN];

bool SPFA(int s, int t){
    std::queue<int> qu;
    qu.push(s);
    
    std::memset(last,-1,sizeof(last));
    std::memset(dis,127,sizeof(dis));
    std::memset(inq,0,sizeof(inq));

    flow[s] = INF;
    dis[s] = 0;
    inq[s] = 1;
    
    while(!qu.empty()){
        int p = qu.front();
        qu.pop();
        inq[p] = 0;

        int size = graph[p].size();
        for(int i=0;i<size;i++){
            int eg = graph[p][i];
            int to = edges[eg].v;
            LL vol = edges[eg].w;
            if(vol>0 && dis[to]>dis[p]+edges[eg].c){
                last[to] = eg;
                flow[to] = std::min(flow[p], vol);
                dis[to] = dis[p]+edges[eg].c;
                if(!inq[to]){
                    qu.push(to);
                    inq[to] = 1;
                }
            }
        }
    }
    return last[t] != -1;
}

pll MCMF(int s, int t){
    LL maxflow = 0, mincost = 0;

    while(SPFA(s,t)){
        maxflow += flow[t];
        mincost += dis[t] * flow[t];
        for(int i=t;i!=s;i=edges[last[i]^1].v){
            edges[last[i]].w -= flow[t];
            edges[last[i]^1].w += flow[t];
        }
    }

    return {maxflow,mincost};
}

int main(){
    int n,m,s,t;//点数，边数，源点，汇点
    std::cin>>n>>m>>s>>t;

    for(int i=1;i<=m;i++){
        int u,v;LL w,c;
        std::cin>>u>>v>>w>>c;
        graph[u].push_back(edges.size());
        edges.push_back(Edge(v,w,c));
        graph[v].push_back(edges.size());
        edges.push_back(Edge(u,0,-c));
    }

    pll ans = MCMF(s,t);
    std::cout<<ans.first<<" "<<ans.second<<"\n";

    return 0;
}

```

#### Dinic+SPFA

```cpp
//luogu P3381
//Dinic+SPFA的实现，复杂度为O(nmf)，即点数、边数、最大流
#include <iostream>
#include <cstring>
#include <vector>
#include <queue>

typedef long long LL;
typedef std::pair<LL,LL> pll;

const int MAXN = 5005;
const LL INF = 0xffffffff;

struct Edge{
    int v;LL w,c;//指向的点，容量，费用
    Edge(int v_, LL w_, LL c_):v(v_),w(w_),c(c_){}
};

std::vector<Edge> edges;
std::vector<std::vector<int> > graph(MAXN);//vector版的链式前向星
std::vector<int> cur(MAXN);
LL dis[MAXN];
bool inq[MAXN];

bool SPFA(int s, int t){//BFS分层
    std::fill(dis,dis+MAXN,INF);
    std::memset(inq, 0, sizeof(inq));

    dis[s] = 0;
    inq[s] = 1;

    cur.assign(MAXN,0);//初始化当前弧
    std::queue<int> qu;
    qu.push(s);

    while(!qu.empty()){
        int p = qu.front();
        qu.pop();
        inq[p] = 0;
        
        int size = graph[p].size();
        for(int i=0;i<size;i++){
            int eg = graph[p][i];
            int to = edges[eg].v;
            LL vol = edges[eg].w;
            if(vol>0 && dis[to] > dis[p]+edges[eg].c){
                dis[to] = dis[p]+edges[eg].c;
                if(!inq[to]){
                    qu.push(to);
                    inq[to] = 1;
                }
            }
        }
    }

    return dis[t] != INF;
}

LL DFS(int const & p, LL const & flow, int const & s, int const & t){
    if(p==t) return flow;

    LL surplus = flow;//剩余流量
    inq[p] = 1;//由于在SPFA中都会清零，可以复用

    int size = graph[p].size();
    for(int i=cur[p];i<size && surplus;i++){
        int eg = graph[p][i];
        cur[p] = i;//更新当前弧
        int to = edges[eg].v;
        LL vol = edges[eg].w;
        if(!inq[to] && vol>0 && dis[to]==dis[p]+edges[eg].c){
            LL cx = DFS(to, std::min(vol, surplus), s, t);
            surplus -= cx;
            edges[eg].w -= cx;
            edges[eg^1].w += cx;
        }
    }
    inq[p] = 0;

    return flow - surplus;
}

pll MCMF(int const & p, LL const & flow, int const & s, int const & t){
    LL maxflow = 0, mincost = 0;
    while(SPFA(s,t)){
        LL ret = DFS(p,flow,s,t);
        maxflow += ret;
        mincost += ret * dis[t];
    }
    return {maxflow,mincost};
}

int main(){
    int n,m,s,t;//点数，边数，源点，汇点
    std::cin>>n>>m>>s>>t;

    for(int i=1;i<=m;i++){
        int u,v;LL w,c;
        std::cin>>u>>v>>w>>c;
        graph[u].push_back(edges.size());
        edges.push_back(Edge(v,w,c));
        graph[v].push_back(edges.size());
        edges.push_back(Edge(u,0,-c));
    }
    pll ans = MCMF(s,INF,s,t);
    std::cout<<ans.first<<" "<<ans.second<<"\n";

    return 0;
}

```

### 上下界流

#### 无源汇上下界可行流

给定一个没有源点和汇点的网络，每条边的容量都有一个上界和下界，问是否有一个可行流使得流量平衡（即每个点的流入等于流出）。

```cpp
//loj 115
//前面的Dinic算法省略
LL in[MAXN];

int main() {
    int n, m, s, t; //点数，边数，源点，汇点
    std::cin >> n >> m;
    s = n + 1;//虚拟源点
    t = n + 2;//虚拟汇点

    std::vector<LL> ans;

    for (int i = 1; i <= m; i++) {
        int u, v;
        LL w1, w2;//下界，上界
        std::cin >> u >> v >> w1 >> w2;
        ans.push_back(w1);
        graph[u].push_back(edges.size());
        edges.push_back(Edge(v, w2 - w1));//只用建立差网络即可
        graph[v].push_back(edges.size());
        edges.push_back(Edge(u, 0));

        in[u] -= w1;
        in[v] += w1;
    }

    for (int i = 1; i <= n; i++) {
        if (in[i] > 0) {//在下界网络中，有净流入的节点，要从源点连一条边，大小等于净流入
            graph[s].push_back(edges.size());
            edges.push_back(Edge(i, in[i]));
            graph[i].push_back(edges.size());
            edges.push_back(Edge(s, 0));
        } else {//有净流出的节点，要向汇点连一条边，大小等于净流出
            graph[i].push_back(edges.size());
            edges.push_back(Edge(t, -in[i]));
            graph[t].push_back(edges.size());
            edges.push_back(Edge(i, 0));
        }
    }

    Dinic(s, INF, s, t);

    for (auto x : graph[s]) {
        if (edges[x].w != 0) {
            //如果源点的附加边没有满流，说明不存在可行流
            //也可以换成判断汇点没有满流，二者等价
            std::cout << "NO\n";
            return 0;
        }
    }

    std::cout << "YES\n";

    for (int i = 1; i < 2 * m; i += 2) {
        //反向边就是这条边的流量，再加之前输入的下界得到每条边的流量
        ans[i / 2] += edges[i].w;
    }

    for (auto x : ans) {
        std::cout << x << "\n";
    }

    return 0;
}
```

#### 有源汇上下界可行流

比起无源汇的情况，我们可以把图中的汇点向源点连一条下界0，上界无限大的边。然后就变成无源汇图了。处理的时候，新建附加源点汇点$S',T'$，原来的$S,T$就变成了普通点，思路一致。

若有解，则$S$到$T$的可行流流量等于$T$到$S$的附加边的流量。

#### 有源汇上下界最大流

在有源汇上下界可行流有解的时候，$S$到$T$的可行流量就是$T$到$S$的附加边的流量。然后我们删去所有添加的附加边，包括$S',T'$连的以及$T-S$附加边，再在跑完的网络上再跑一次Dinic，得到的流加上可行流就是最后的答案。

具体实践上，我们不需要真的把边删了，$S',T'$所连的边根本不会影响结果，可以不用管，至于$T-S$这条边，我们获取了流量之后，直接把正向、反向边置零即可。

```cpp
//loj 116
//前面的Dinic算法省略
LL in[MAXN];

int main() {
    int n, m, s, t; //点数，边数，源点，汇点
    std::cin >> n >> m >> s >> t;

    for (int i = 1; i <= m; i++) {
        int u, v;
        LL w1, w2;
        std::cin >> u >> v >> w1 >> w2;
        graph[u].push_back(edges.size());
        edges.push_back(Edge(v, w2 - w1));
        graph[v].push_back(edges.size());
        edges.push_back(Edge(u, 0));

        in[u] -= w1;
        in[v] += w1;
    }

    graph[t].push_back(edges.size());
    edges.push_back(Edge(s, INF * 4ll));
    graph[s].push_back(edges.size());
    edges.push_back(Edge(t, 0));
    //T-S的边
    int s2 = n + 1, t2 = n + 2;

    for (int i = 1; i <= n; i++) {
        if (in[i] > 0) {
            graph[s2].push_back(edges.size());
            edges.push_back(Edge(i, in[i]));
            graph[i].push_back(edges.size());
            edges.push_back(Edge(s2, 0));
        } else {
            graph[i].push_back(edges.size());
            edges.push_back(Edge(t2, -in[i]));
            graph[t2].push_back(edges.size());
            edges.push_back(Edge(i, 0));
        }
    }

    Dinic(s2, INF, s2, t2);

    for (auto x : graph[s2]) {
        if (edges[x].w != 0) {
            std::cout << "please go home to sleep\n";
            return 0;
        }
    }

    LL flow = edges[2 * m + 1].w;
    edges[2 * m].w = 0, edges[2 * m + 1].w = 0;
    std::cout << Dinic(s, INF, s, t) + flow << "\n";

    return 0;
}
```

#### 有源汇上下界最小流

和上面几乎一模一样，只需在拆掉附加边后，从汇点到源点跑一次Dinic，然后flow删去这个结果就得到最小流。Loj 117。

## 割边（Tarjan算法）

```cpp
//复杂度 n+m
//tarjan求割边，可以正确处理重边
//如果无向图中删掉某条边会使无向图的连通分量数增多，那么这条边叫割边
//luogu p1656
#include <iostream>
#include <vector>
#include <stack>
#include <algorithm>

const int MAXN = 20005;
const int MAXM = 100005;

int dfn[MAXN], low[MAXN], cnt=0;
//dfn为对一个图进行dfs时，dfs的顺序序号
//low[x]为以下所有符合要求的节点的dfn中的最小值
//1.以x为根的子树的所有节点
//2.通过非dfs生成树上的边能够到达该子树的所有节点
struct Edge{
    int v,next;//指向的点，边权，下一条边
};

Edge edges[MAXM*2];//存无向图记得开两倍
int head[MAXN],ecnt=1;//注意这个ecnt=1，这是用来方便in_edge判断的
bool bridges[MAXM*2];//判断一条边是不是割边

inline void add(int u, int v){
    edges[++ecnt].v = v;
    edges[ecnt].next = head[u];
    head[u] = ecnt;
}

void tarjan(int u, int in_edge){
    low[u] = dfn[u] = ++cnt;
    for(int i=head[u];i;i=edges[i].next){
        int v = edges[i].v;
        if(!dfn[v]){
            tarjan(v,i);
            low[u] = std::min(low[u],low[v]);
            if(low[v]>dfn[u])//边u-v是割边的充要条件
                bridges[i] = bridges[i^1] = true;          
            
        }
        else if(i != (in_edge ^ 1)){
            low[u] = std::min(low[u], dfn[v]);
        }
    }
}

int main(){
    int n,m;
    std::cin>>n>>m;
    //点数，边数
    for(int i=1;i<=m;i++){
        int a,b;
        std::cin>>a>>b;
        //起点，终点
        add(a,b);
        add(b,a);
		//无向图
    }
    for(int i=1;i<=n;i++){
        if(!dfn[i])
            tarjan(i,0);
    }
    
    std::vector<std::pair<int,int> > ans;
    for(int i=2;i<ecnt;i+=2){
        if(bridges[i]){
            int u = edges[i].v;
            int v = edges[i^1].v;
            if(u>v) std::swap(u,v);
            ans.push_back({u,v});
        }
    }
    std::sort(ans.begin(),ans.end());
    for(auto x:ans) std::cout<<x.first<<" "<<x.second<<"\n";
    
    return 0;
}

```

## 割点（Tarjan算法）

```cpp
//复杂度 n+m
//tarjan求割点,luogu P3388
//如果无向图中删掉某个点和其所有相连的边边会使无向图的连通分量数增多，那么这个点叫割点
#include <iostream>
#include <vector>
#include <stack>
#include <algorithm>

const int MAXN = 20005;
const int MAXM = 100005;

int dfn[MAXN], low[MAXN], cnt=0;
//含义见割边模板
struct Edge{
    int v,next;//指向的点，边权，下一条边
};

Edge edges[MAXM*2];//存无向图记得开两倍
int head[MAXN],ecnt;//这个ecnt和割边那里不一样，但也可以等于1
bool cut[MAXN];//判断割点

inline void add(int u, int v){
    edges[++ecnt].v = v;
    edges[ecnt].next = head[u];
    head[u] = ecnt;
}

void tarjan(int u, int root){
    int tot = 0;
    low[u] = dfn[u] = ++cnt;
    for(int i=head[u];i;i=edges[i].next){
        int v = edges[i].v;
        if(!dfn[v]){
            tarjan(v,root);
            low[u] = std::min(low[u],low[v]);
            //一个点x是割点的充要条件是，它至少一个子节点y满足dfn[x]<=low[y]，特别的，对于根节点，需要至少两个这样的子节点
            if(low[v]>=dfn[u]){
                tot++;
                if(u!=root || tot>1) cut[u] = true;
            }
        }
        else{
            low[u] = std::min(low[u], dfn[v]);
        }
    }
}

int main(){
    int n,m;
    std::cin>>n>>m;
    //点数，边数
    for(int i=1;i<=m;i++){
        int a,b;
        std::cin>>a>>b;
        //起点，终点
        add(a,b);
        add(b,a);
        //无向图
    }
    for(int i=1;i<=n;i++){
        if(!dfn[i])
            tarjan(i,i);
    }
    
    std::vector<int> ans;
    for(int i=1;i<=n;i++) if(cut[i]) ans.push_back(i);
    std::sort(ans.begin(),ans.end());
    std::cout<<ans.size()<<"\n";
    for(auto x:ans) std::cout<<x<<" ";
    
    return 0;
}
```

## 强连通分量（Tarjan算法）

```cpp
//强连通分量，复杂度 n+m
//luogu P2863
//一个有向图是强连通的当且仅当其中任意两个顶点相互可达
//强连通分量是有向图中的极大的强连通子图。极大意味着把一个图分为若干个强连通分量，分量之间互相不可达。或者，不存在包含该子图的更大的子图也是强连通分量。

#include <iostream>
#include <vector>
#include <stack>

const int MAXN = 10005;
const int MAXM = 50005;

int dfn[MAXN], low[MAXN], instk[MAXN], scci[MAXN], cnt=0, cscc=0;
std::vector<int> edges[MAXN];
std::stack<int> stk;
std::vector<int> scc[MAXN];
//dfn是dfs时的顺序的序号
//stk中存入两类点，访问到节点x时
//1.搜索树上x的祖先节点
//2.已经访问过，并且存在一条路径到达x祖先的节点
//low[x]定义为满足以下两个条件的节点的最小dfn
//1.该点在stk中
//2.存在一条从subtree(x)出发的有向边，以该点为终点
//scci[x]代表，x这个结点在第几个分量中
//cscc代表有几个分量
//scc[j]中表示，第j个分量的所有节点

void tarjan(int u){
    low[u] = dfn[u] = ++cnt;
    instk[u] = 1;
    stk.push(u);
    for(int i=0;i<edges[u].size();i++){
        int v = edges[u][i];
        if(!dfn[v]){
            tarjan(v);
            low[u] = std::min(low[u],low[v]);
        }
        else if(instk[v]){
            low[u] = std::min(low[u], dfn[v]);
        }
    }
    if(low[u]==dfn[u]){
        int top;
        cscc++;
        do{
            top = stk.top();
            stk.pop();
            instk[top] = 0;
            scci[top] = cscc;
            scc[cscc].push_back(top);
        }while(top!=u);
    }
}

int main(){
    int n,m;
    std::cin>>n>>m;
    
    for(int i=1;i<=m;i++){
        int a,b;
        std::cin>>a>>b;
        edges[a].push_back(b);//有向边
    }
    
    for(int i=1;i<=n;i++){
        if(!dfn[i]) tarjan(i);//注意遍历所有dfn为零的点
    }
    int ans = 0;
    for(int i=1;i<=cscc;i++){
        if(scc[i].size()>1) ans++;
    }
    std::cout<<ans<<"\n";
    return 0;
}
```

## 2-SAT问题

```cpp
//2-SAT算法，复杂度n+m
//luogu P4782

//2-SAT是用来解决一些条件是否能够满足的算法。
//每个条件都能转化为形如"若x赋值为a，则y必须赋值为b"的形式。其中a,b的取值只能有两个，通常是true和false。
//例如总共有m个这样的条件，我们要判断是否存在一种赋值情况满足所有的条件。如果有还要输出一种可行方案

#include <iostream>
#include <vector>
#include <stack>

const int MAXN = 2e6+5;

int dfn[MAXN], low[MAXN], instk[MAXN], scci[MAXN], cnt=0, cscc=0;
std::vector<int> edges[MAXN];
std::stack<int> stk;
std::vector<int> scc[MAXN];
//含义见强连通分量tarjan算法
bool ans[MAXN];

void tarjan(int u){
    low[u] = dfn[u] = ++cnt;
    instk[u] = 1;
    stk.push(u);
    for(int i=0;i<edges[u].size();i++){
        int v = edges[u][i];
        if(!dfn[v]){
            tarjan(v);
            low[u] = std::min(low[u],low[v]);
        }
        else if(instk[v]){
            low[u] = std::min(low[u], dfn[v]);
        }
    }
    if(low[u]==dfn[u]){
        int top;
        cscc++;
        do{
            top = stk.top();
            stk.pop();
            instk[top] = 0;
            scci[top] = cscc;
            scc[cscc].push_back(top);
        }while(top!=u);
    }
}

int main(){
    int n,m;
    std::cin>>n>>m;
    //n个点，m个条件
    
    while(m--){
        int i,j;
        bool a,b;
        std::cin>>i>>a>>j>>b;
        //本题的条件为，i为a或（不是异或）j为b，其他题按情况处理
        //每个点x拆为两个点y和y+n,y代表x为0，y+n代表x为1
        
        //每条边x->y代表着，如果x，那么一定有y
        //本题如“i为假或j为真”可以拆为两个条件，这个条件满足（为真）时
        //i为真则j一定为真
        //j为假则i一定为假
        edges[i+(!a)*n].push_back(j+b*n);
        edges[j+(!b)*n].push_back(i+a*n);//逆否命题
        //逆否命题是一定要插入的，不能只插入原命题，但是本题拆出来的两个条件正好互为逆否命题，所以只插入了两条边。其他题并不一定总会给出逆否命题
        //这里的逻辑运算可能有些不容易理解，怕错可以写成很长的if else判断a和b的具体取值
    }
    
    for(int i=1;i<=2*n;i++){
        if(!dfn[i]) tarjan(i);//注意遍历所有dfn为零的点
    }
    
    for(int i=1;i<=n;i++){
        if(scci[i]==scci[i+n]){//如果i和i+n在一个强连通分量，则不可满足
            std::cout<<"IMPOSSIBLE\n";
            return 0;
        }
        else if(scci[i]>scci[i+n])
            ans[i] = 1;
        else
            ans[i] = 0;
    }
    std::cout<<"POSSIBLE\n";
    for(int i=1;i<=n;i++){
        std::cout<<ans[i]<<" ";
    }

    return 0;
}
```

## 边双联通分量

```cpp
//复杂度 n+m
//tarjan求边双联通分量
//luogu p8436

//如果一张无向连通图不存在割边，则称之为边双联通图
//双连通分量是图的极大双联通子图
//若u-v边双联通，v-w边双联通，则u-w边双联通
//一张图是边双联通，当且仅当每条边都在至少一个简单环中

//无向连通图中，对于任意两个点，如果无论删去哪条边（只能一条），都不能使它们不连通，则为边双联通
//同时这也意味着，把割边删去后的图，就是若干个双联通分量

/*这一段是求割边的核心代码，省略*/

int dcci[MAXN], cdcc;//记录点i属于双联通分量dcci[i]，以及总的dcc个数
std::vector<std::vector<int> > dcc(MAXN);//存储双联通分量dcc[i]中有哪些点

void getDCC(int u){
    dcci[u] = cdcc;
    dcc[cdcc].push_back(u);
    for(int i=head[u];i;i=edges[i].next){
        int v = edges[i].v;
        if(dcci[v] || bridges[i]) continue;
        getDCC(v);
    }
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    
    int n,m;
    std::cin>>n>>m;
    //点数，边数
    for(int i=1;i<=m;i++){
        int a,b;
        std::cin>>a>>b;
        //起点，终点
        add(a,b);
        add(b,a);
		//无向图
    }
    for(int i=1;i<=n;i++){
        if(!dfn[i])
            tarjan(i,0);
    }
    
    //以上求完了割边
    
    for(int i=1;i<=n;i++){
        if(!dcci[i]){
            cdcc++;
            getDCC(i);
        }
    }
    
    std::cout<<cdcc<<"\n";
    for(int i=1;i<=cdcc;i++){
        std::cout<<dcc[i].size()<<" ";
        for(auto x:dcc[i]){
            std::cout<<x<<" ";
        }
        std::cout<<"\n";
    }
     
    return 0;
}
```

## 边双联通缩点TODO

## 点双联通分量

```cpp
//复杂度 n+m
//tarjan求点双联通分量
//luogu p8435

//如果一张无向连通图不存在割点，则称之为点双联通图
//双连通分量是图的极大双联通子图
//极大指的是，不存在包含这个子图的更大的子图也是边双联通图
//若u-v点双联通，v-w点双联通，则u-w[并不一定]点双联通
//一张图是点双联通，当且仅当以下两个条件之一成立
//1. 图的顶点数不超过2
//2. 图中任意两点都同时包含在至少一个简单环中

//无向连通图中，对于任意两个点，如果无论删去哪个点（只能一个，且不能删除这两个点自己），都不能使它们不连通，则为点双联通
//但是，虽然边双联通中的割边不属于任何连通分量，但割点却可以属于多个点双联通分量

#include <iostream>
#include <vector>
#include <stack>
#include <algorithm>

const int MAXN = 500005;
const int MAXM = 2000005;

int dfn[MAXN], low[MAXN], cnt=0;
//含义见割边模板
struct Edge{
    int v,next;//指向的点，边权，下一条边
};

Edge edges[MAXM*2];//存无向图记得开两倍
int head[MAXN],ecnt;//这个ecnt和割边那里不一样，但也可以等于1
bool cut[MAXN];//判断割点

inline void add(int u, int v){
    edges[++ecnt].v = v;
    edges[ecnt].next = head[u];
    head[u] = ecnt;
}

std::stack<int> stk;
int cdcc;
std::vector<std::vector<int> > dcc(MAXN);

void tarjan(int u, int root){
    low[u] = dfn[u] = ++cnt;
    stk.push(u);
    if(u==root && head[u]==0){
        dcc[++cdcc].push_back(u);
        return;
    }
    int tot=0;
    for(int i=head[u];i;i=edges[i].next){
        int v = edges[i].v;
        if(!dfn[v]){
            tarjan(v,root);
            low[u] = std::min(low[u],low[v]);
            if(low[v]>=dfn[u]){
                tot++;
                if(u!=root || tot>1) cut[u] = true;
                cdcc++;
                int z;
                do{
                    z = stk.top();
                    stk.pop();
                    dcc[cdcc].push_back(z);
                }while(z!=v);
                dcc[cdcc].push_back(u);
            }
        }
        else{
            low[u] = std::min(low[u], dfn[v]);
        }
    }
}

int main(){
    int n,m;
    std::cin>>n>>m;
    //点数，边数
    for(int i=1;i<=m;i++){
        int a,b;
        std::cin>>a>>b;
        //起点，终点
        if(a==b) continue;//点双联通需要注意排除自环才能处理孤立点
        add(a,b);
        add(b,a);
        //无向图
    }

    for(int i=1;i<=n;i++){
        if(!dfn[i])
            tarjan(i,i);
    }
    
    std::cout<<cdcc<<"\n";
    for(int i=1;i<=cdcc;i++){
        std::cout<<dcc[i].size()<<" ";
        for(auto x:dcc[i]) std::cout<<x<<" ";
        std::cout<<"\n";
    }
    
    return 0;
}
```

## 点双联通缩点TODO

## 树的直径

```cpp
//树的直径，复杂度n
//poj1985，输出树上最长路径的长度，即树的直径
//两遍dfs版可以求出路径上的点，但树形dp的可以处理负边权问题
std::vector<pii> edges[MAXN];//first是v，second是w
int dis[MAXN];
int far;

void dfs(int u, int fa){
    int size = edges[u].size();
    for(int i=0;i<size;i++){
        pii e = edges[u][i];
        int v = e.first, w = e.second;
        if(v==fa) continue;
        dis[v] = dis[u]+w;
        if(dis[v]>dis[far]) far=v;
        dfs(v,u);
    }
}

void solve(){
    int n,m;
    std::cin>>n>>m;
    for(int i=1;i<=m;i++){
        int u,v,w;
        char trash;//poj 1985的输入数据问题
        std::cin>>u>>v>>w>>trash;
        edges[u].push_back(std::make_pair(v,w));
        edges[v].push_back(std::make_pair(u,w));
    }
    dfs(1,0);
    dis[far] = 0;
    dfs(far,0);
    std::cout<<dis[far]<<"\n";
}
```

```cpp
//树的直径，复杂度n
//poj1985，输出树上最长路径的长度，即树的直径
//两遍dfs版可以求出路径上的点，但树形dp的可以处理负边权问题
std::vector<pii> edges[MAXN];//first是v，second是w
int dis[MAXN];
bool vis[MAXN];
int ans;

void dp(int u){
    vis[u] = 1;
    int size = edges[u].size();
    for(int i=0;i<size;i++){
        pii e = edges[u][i];
        int v = e.first, w = e.second;
        if(vis[v]) continue;
        dp(v);
        ans = std::max(ans,dis[u]+dis[v]+w);
        dis[u] = std::max(dis[u],dis[v]+w);
    }
}

void solve(){
    int n,m;
    std::cin>>n>>m;
    for(int i=1;i<=m;i++){
        int u,v,w;
        char trash;//poj 1985的输入数据问题
        std::cin>>u>>v>>w>>trash;
        edges[u].push_back(std::make_pair(v,w));
        edges[v].push_back(std::make_pair(u,w));
    }
    dp(1);
    std::cout<<ans<<"\n";
}
```

若树上所有边边权均为正，则树的所有直径中点重合。

## 树的重心

计算无根树的每一个节点作为根时，其最大子树的大小。最大子树的大小最小的节点叫做树的重心。

**性质如下**

1. 重心如果不唯一，则最多只有两个，且它们相邻。并且此时树有偶数个节点，可以被划分为两个大小相等的连通块，每个块各自包含一个重心。
2. 以树的重心为根时，所有子树的大小都不超过整棵树的一半
3. 树中所有点到某个点的距离之和中，到重心的距离之和是最小的。如果有两个重心，它们是并列最小的。反过来距离之和最小的点一定是重心。
4. 两棵树通过一条边连成一棵树，则新树的重心在连接原来两颗树的重心的路径上。如果两棵树大小一样，那么重心就是两个连接点。
5. 在一棵树上添加或删除一个叶节点，它的重心最多只移动一条边的距离。如果原树有奇数个节点，那么重心可能会增加一个，原重心仍然是重心。如果有偶数个节点，那么重心可能减少一个，另一个重心仍然是重心。

```cpp
//复杂度 n
//poj 1655
std::vector<int> edges[MAXN];
int sz[MAXN], mss[MAXN];//树的大小（含自己），最大子树大小（不含自己）
std::vector<int> ctr;//存重心

void dfs(int u, int fa, int const n){//需要传入点的个数
    sz[u] = 1, mss[u] = 0;
    int size = edges[u].size();
    for(int e=0;e<size;e++){
        int v = edges[u][e];
        if(v==fa) continue;
        dfs(v,u,n);
        mss[u] = std::max(mss[u],sz[v]);
        sz[u] += sz[v];
    }
    mss[u] = std::max(mss[u],n-sz[u]);
    if(mss[u]<=n/2) ctr.push_back(u);
}
```

## 倍增求最近公共祖先

```cpp
//复杂度 单次查询 logn 预处理 nlogn，常数小点的可以用重链剖分
//luogu P3379
//倍增求最近公共祖先
#include <iostream>
#include <vector>
#include <cstdio>

int const MAXN = 500005;
int const LOGN = 31;

std::vector<int> edge[MAXN];//邻接表
int logn[MAXN];
int fa[MAXN][LOGN],deep[MAXN];
//fa[a][b]代表a的第2^b个祖先，deep是深度，根节点深度为1

void build(int u,int father){
    fa[u][0] = father;
    deep[u] = deep[father]+1;

    for(int i=1;i<=logn[deep[u]];i++){
        fa[u][i] = fa[fa[u][i-1]][i-1];
    }

    for(auto v:edge[u]){
        if(v==father) continue;
        build(v,u);
    }
}

int lca(int x,int y){
    if(deep[x]>deep[y]) std::swap(x,y);
    //保证y比x深

    while(deep[x]!=deep[y]){
        y = fa[y][logn[deep[y]-deep[x]]];
    }
    
    if(x==y) return x;
    
    for(int k=logn[deep[x]];k>=0;k--){
        if(fa[x][k]!=fa[y][k]){
            x = fa[x][k], y = fa[y][k];
        }
    }
    return fa[x][0];
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    
    int n,m,s;
    std::cin>>n>>m>>s;
    //点数，询问数，根节点序号
    
    for(int i=2;i<=n;i++){
        logn[i] = logn[i/2] + 1;
        //必须的初始化
    }

    for(int i=1;i<=n-1;i++){
        int a,b;
        std::cin>>a>>b;
        //读入树
        edge[a].push_back(b);
        edge[b].push_back(a);
    }

    build(s,0);//必须build才能用

    for(int i=1;i<=m;i++){
        int x,y;
        std::cin>>x>>y;
        //查询x,y的最近公共祖先
        std::cout<<lca(x,y)<<"\n";
    }

    return 0;
}

```

## 虚树 TODO

```cpp

```

## 点分治

```cpp
//点分治 复杂度nlog^2n
//poj 1741
//查询树上长度小于等于k的路径的数量
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
#include <cstdio>

int const MAXN = 10007;

struct Edge{
    int v,w,next;//指向的点，边权，下一条边
};

Edge edges[MAXN*2];
int head[MAXN],cnt;

inline void add(int u, int v, int w){
    edges[++cnt].w = w;
    edges[cnt].v = v;
    edges[cnt].next = head[u];
    head[u] = cnt;
}

int sz[MAXN], mss[MAXN];//树的大小（含自己），最大子树大小（不含自己）
int ctr=-1;//重心
bool del[MAXN];//这个点是否在分治的时候被删除

void dfsCtr(int u, int fa, int const n){//需要传入树的点的个数
    //执行完后ctr为本子树的重心
    sz[u] = 1, mss[u] = 0;
    for(int e=head[u];e;e=edges[e].next){
        int v = edges[e].v;
        if(v==fa || del[v]) continue;
        dfsCtr(v,u,n);
        if(ctr!=-1) return;
        mss[u] = std::max(mss[u],sz[v]);
        sz[u] += sz[v];
    }
    mss[u] = std::max(mss[u],n-sz[u]);
    if(mss[u]<=n/2) ctr = u, sz[fa] = n-sz[u];//注意要改编sz以保证复杂度正确
}

int dis[MAXN];//dis[x]存储点x到根root的距离
int indexx[MAXN];//要对节点编号按照dis进行排序，indexx[0]代表元素个数
int belong[MAXN];//判断子树节点属于哪一个子子树
int cntsame[MAXN];//查询[l,r]时，维护[l+1,r]中belong与l的belong相同的个数，见calc函数

bool cmp(int x,int y){return dis[x]<dis[y];}

void dfsDis(int u, int fa, int from){
    //获得子树到根节点的距离，from用于计算belong
    indexx[++indexx[0]] = u;
    belong[u] = from;
    for(int e=head[u];e;e=edges[e].next){
        int v = edges[e].v, w = edges[e].w;
        if(v==fa || del[v]) continue;
        dis[v] = dis[u] + w;
        cntsame[from]++;
        dfsDis(v,u,from);
    }
}

int calc(int u,int k){
    indexx[0] = 0;
    indexx[++indexx[0]] = u;
    belong[u] = u;
    dis[u] = 0;
    cntsame[u] = 1;
    for(int e=head[u];e;e=edges[e].next){
        int v = edges[e].v, w =edges[e].w;
        if(del[v]) continue;
        dis[v] = dis[u] + w;
        cntsame[v] = 1;
        dfsDis(v,u,v);
    }
    std::sort(indexx+1,indexx+1+indexx[0],cmp);
    
    int l=1, r=indexx[0],ans=0;
    while(l<r){
        int x = indexx[l], y = indexx[r];
        if(dis[x]+dis[y]>k){
            cntsame[belong[y]]--;//把cntsame由[l,r]转移,r-1]
            r--;
        }
        else{
            //显然，如果不考虑两个点在同一个子子树内，则l和l+1~r的每个点都满足dis[x]+dis[y]<=k
            //减去同子子树的情况，即减去[l+1,r]中和l拥有相同belong的点
            cntsame[belong[x]]--;//把cntsame由[l,r]转移到[l+1,r]，一定要注意顺序
            ans += r-l-cntsame[belong[x]];
            l++;
        } 
    }
    return ans;
}

int res = 0;

void divide(int u, int k){
    del[u] = 1;
    res += calc(u,k);
    for(int e=head[u];e;e=edges[e].next){
        int v = edges[e].v, w = edges[e].w;
        if(del[v]) continue;
        ctr = -1;
        dfsCtr(v,0,sz[v]);
        divide(ctr,k);
    }
}

void solve(int n, int k){
    for(int i=1;i<n;i++){
        int u,v,w;
        std::cin>>u>>v>>w;
        add(u,v,w);
        add(v,u,w);
    }
    dfsCtr(1,0,n);
    divide(ctr,k);
    std::cout<<res<<"\n";
    
    ctr = -1;
    cnt = 0;
    res = 0;
    for(int i=1;i<=n;i++) head[i] = 0,del[i] = 0;
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);

	int n,k;
	
	while(std::cin>>n>>k){
	    if(n==0&&k==0) break;
	    solve(n,k);
	}

    return 0;
}

```

## 重链剖分

```cpp
//树根节点的子节点中子树最大的为它的重子节点，其他的为轻子节点（整棵树的根节点是轻点，其他子树的根节点可轻可重）
//节点连向其轻子节点的边叫轻边，否则叫重边
//节点数为n，则从任意节点向上到根节点，经过的轻边数不超过logn

struct Node{
    int fa, sz, dep, hson;//父节点、子树大小（包含自己）、深度、重子节点
    int top;//链头，即所在的重链中深度最小的那个节点 
}node[MAXN];

std::vector<int> edges[MAXN];

void dfs1(int u, int d=1){
    //在dfs2之前先用dfs1
    int size = 1, ma = 0;
    node[u].dep = d;
    for(auto v:edges[u]){
        if(!node[v].dep){
            dfs1(v,d+1);
            node[v].fa = u;
            size += node[v].sz;
            if(node[v].sz > ma){
                node[u].hson = v, ma = node[v].sz;
            }
        }
    }
    node[u].sz = size;
}

void dfs2(int u){
    //需要先把根节点的top设置为自己
    for(auto v:edges[u]){
        if(!node[v].top){
            if(v==node[u].hson) node[v].top = node[u].top;
            else node[v].top = v;
            dfs2(v);
        }
    }
}

void cut(int r=1){
    //进行树剖预处理
    dfs1(r);
    node[r].top = r;
    dfs2(r);
}

```

### 重链剖分求LCA

```cpp
//树剖求LCA，每次查询复杂度 logn，常数很小
//luogu p3379
int lca(int a, int b){
    while(node[a].top!=node[b].top){
        if(node[node[a].top].dep>node[node[b].top].dep)
            a = node[node[a].top].fa;
        else
            b = node[node[b].top].fa;
    }
    if(node[a].dep > node[b].dep) return b;
    return a;
}
```

### 重链剖分+线段树维护树上路径点权和

```cpp
//树剖维护树上路径的点权和，维护和查询一次复杂度 logn
//luogu p3384

//树根节点的子节点中子树最大的为它的重子节点，其他的为轻子节点（整棵树的根节点是轻点，其他子树的根节点可轻可重）
//节点连向其轻子节点的边叫轻边，否则叫重边
//节点数为n，则从任意节点向上到根节点，经过的轻边数不超过logn
#include <iostream>
#include <algorithm>
#include <cmath>
#include <cstring>
#include <vector>

#define pb push_back
#define mkp std::make_pair
#define fi first
#define se second

using LL = long long;

int const MAXN = 100005;
int const INF = 0x7fffffff;
LL MOD = 998244353;

struct Node{
    int fa, sz, dep, hson;//父节点、子树大小（包含自己）、深度、重子节点
    int top;//链头，即所在的重链中深度最小的那个节点 
    int dfn, mdfn;//该节点的dfs序，该节点子树的最大dfs序
    LL v;//点上的权
}node[MAXN];
int dfnmap[MAXN];//映射dfn对应的点编号

std::vector<int> edges[MAXN];

void dfs1(int u, int d=1){
    //在dfs2之前先用dfs1
    int size = 1, ma = 0;
    node[u].dep = d;
    for(auto v:edges[u]){
        if(!node[v].dep){
            dfs1(v,d+1);
            node[v].fa = u;
            size += node[v].sz;
            if(node[v].sz > ma){
                node[u].hson = v, ma = node[v].sz;
            }
        }
    }
    node[u].sz = size;
}

int cnt=0;

void dfs2(int u){
    //需要先把根节点的top设置为自己
    node[u].dfn = ++cnt;
    dfnmap[cnt] = u;
    if(node[u].hson!=0){
        node[node[u].hson].top = node[u].top;
        dfs2(node[u].hson);
    }
    //采取这个改变的原因是，每棵子树的dfs序是连续的，根节点dfs序最小
    //而如果我们强制先遍历重子节点，那么重链上的dfs序是连续的，并且链头dfs序最小。这样就能用线段树维护链上的信息了
    
    for(auto v:edges[u]){
        if(!node[v].top){
            node[v].top = v;
            dfs2(v);
        }
    }
    
    node[u].mdfn = cnt;
}

void cut(int r){
    dfs1(r);
    node[r].top = r;
    dfs2(r);
}

struct Nodest
{
    int s,t;//该端点的起点和终点下标
    LL tag, v;
};

Nodest st[MAXN*4+2];

void build(int s, int t, int p){
    st[p].s = s;
    st[p].t = t;
    if(s==t) {
        st[p].v = node[dfnmap[s]].v%MOD;
        st[p].tag = 0;
        return;
    }
    int m = s+((t-s)>>1);
    build(s,m,p*2);
    build(m+1,t,p*2+1);
    st[p].v = (st[p*2].v + st[p*2+1].v)%MOD;
    st[p].tag = 0;
}

void spreadTag(int p){
    if(st[p].tag){
        int s = st[p].s, t = st[p].t;
        int m = s+((t-s)>>1);
        st[p*2].v     = (st[p*2].v + (m-s+1)*st[p].tag)%MOD;
        st[p*2+1].v   = (st[p*2+1].v + (t-m)*st[p].tag)%MOD;
        st[p*2].tag   = (st[p].tag + st[p*2].tag)%MOD;
        st[p*2+1].tag = (st[p].tag + st[p*2+1].tag)%MOD;
        st[p].tag=0;
    }
}

void update(int l, int r, int p, LL k){
    int s = st[p].s, t = st[p].t;
    if(l<=s && t<=r){
        st[p].v   = (st[p].v + (t-s+1) * k)%MOD;
        st[p].tag = (st[p].tag + k)%MOD;
        return;
    }
    spreadTag(p);
    
    int m = s+((t-s)>>1);
    if(l<=m) update(l, r, p*2, k);
    if(r>m)  update(l, r, p*2+1, k);
    st[p].v = (st[p*2].v + st[p*2+1].v)%MOD;
}

LL query(int l, int r, int p){
    int s = st[p].s, t = st[p].t;
    if(l<=s && t<=r) return st[p].v%MOD;
    
    spreadTag(p);
    int m = s+((t-s)>>1);
    LL ret = 0;
    if(l<=m) ret = (ret + query(l,r,p*2))%MOD;
    if(r>m)  ret = (ret + query(l,r,p*2+1))%MOD;
    
    return ret;
}

void update_path(int x, int y, LL k){
    while(node[x].top != node[y].top){
        if(node[node[x].top].dep > node[node[y].top].dep){
            update(node[node[x].top].dfn, node[x].dfn, 1, k);
            x = node[node[x].top].fa;
        }
        else{
            update(node[node[y].top].dfn, node[y].dfn, 1, k);
            y = node[node[y].top].fa;
        }
    }
    if(node[x].dep>node[y].dep){
        update(node[y].dfn, node[x].dfn, 1, k);
    }
    else{
        update(node[x].dfn, node[y].dfn, 1, k);
    }
}

LL query_path(int x, int y){
    LL ans = 0;
    while(node[x].top != node[y].top){
        if(node[node[x].top].dep > node[node[y].top].dep){
            ans += query(node[node[x].top].dfn, node[x].dfn, 1);
            x = node[node[x].top].fa;
        }
        else{
            ans += query(node[node[y].top].dfn, node[y].dfn, 1);
            y = node[node[y].top].fa;
        }
    }
    if(node[x].dep>node[y].dep){
        ans += query(node[y].dfn, node[x].dfn, 1);
    }
    else{
        ans += query(node[x].dfn, node[y].dfn, 1);
    }
    
    return ans%MOD;
}

void update_subtree(int x, LL k){
    update(node[x].dfn, node[x].mdfn, 1, k);
}

LL query_subtree(int x){
    return query(node[x].dfn, node[x].mdfn, 1)%MOD;
}

void solve(){
    int n,m,r;
    std::cin>>n>>m>>r>>MOD;//节点个数，操作个数，根节点序号，取模数
    for(int i=1;i<=n;i++){
        std::cin>>node[i].v;
    }
    for(int i=1;i<n;i++){
        int x,y;
        std::cin>>x>>y;
        edges[x].pb(y);
        edges[y].pb(x);
    }
    
    cut(r);
    build(1,n,1);
    
    while(m--){
        int ope,x,y;
        LL z;
        std::cin>>ope;
        
        if(ope==1){
            std::cin>>x>>y>>z;
            update_path(x,y,z);
        }
        else if(ope==2){
            std::cin>>x>>y;
            std::cout<<query_path(x,y)<<"\n";
        }
        else if(ope==3){
            std::cin>>x>>z;
            update_subtree(x,z);
        }
        else if(ope==4){
            std::cin>>x;
            std::cout<<query_subtree(x)<<"\n";
        }
    }
    
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);

	int T;
	T=1;
	while(T--){
	    solve();
	}

    return 0;
}

```

## 长链剖分

### 求K级祖先

```cpp
//长链剖分定义子树最深深度最深的节点为重子节点
//任意节点p的k级祖先q所在的链长度一定大于k
//任意节点p到根节点最多经过sqrt n级别的轻边

//luogu p5903
//求任意节点的第k级祖先，预处理nlogn，查询常数

#define pb push_back

using LL = long long;

int const MAXN = 500005;

struct Node{
    int fa, dep, hson;//父节点、深度、重子节点
    int top;//链头，即所在的长链中深度最小的那个节点 
    int len,dfn,mdfn;//部分链长，dfs序，子树最大dfs序
}node[MAXN];

int dfnmap[MAXN];//dfs序对应节点

std::vector<int> edges[MAXN];

void dfs1(int u, int d=1){
    node[u].len = 1, node[u].dep = d;
    for(auto v:edges[u]){
        if(!node[v].dep){
            dfs1(v,d+1);
            node[v].fa = u;
            if(node[v].len+1>node[u].len)
                node[u].hson = v, node[u].len = node[v].len+1;
        }
    }
}

int cnt = 0;
void dfs2(int u, int tp){
    node[u].dfn = ++cnt;
    node[u].top = tp;
    dfnmap[cnt] = u;
    if(node[u].hson) dfs2(node[u].hson, tp);
    for(auto v:edges[u]){
        if(!node[v].top) dfs2(v,v);
    }
    node[u].mdfn = cnt;
}

void cut(int r=1){
    dfs1(r);
    dfs2(r,r);
}

std::vector<int> anc[MAXN], des[MAXN];
//分别存储（链头）节点p的1,2,...,node[p].len-1级祖先节点和子孙节点
int const LOGN = 21;
int fa[MAXN][LOGN];
int logn[MAXN];

void init(int r, int n){
    cut(r);
    
    logn[1] = 0;
    logn[2] = 1;
    for(int i=3;i<MAXN;i++){
        logn[i] = logn[i/2]+1;
        //预先计算logn
    }
    
    for(int i=1;i<=n;i++) fa[i][0] = node[i].fa;
    for(int j=1;j<LOGN;j++){
        for(int i=1;i<=n;i++){
            fa[i][j] = fa[fa[i][j-1]][j-1];
        }
    }
    
    for(int i=1;i<=n;i++){
        if(node[i].top==i){
            for(int j=0,p=i;j<node[i].len;j++,p=fa[p][0])
                anc[i].pb(p);
            for(int j=0;j<node[i].len;j++)
                des[i].pb(dfnmap[node[i].dfn+j]);
        }
    }
}

int query(int u, int k){
    //查询节点u的k级祖先
    if(k==0) return u;
    int i = logn[k];
    int v = fa[u][i];
    int tp = node[v].top;
    int d = k - (1<<i) + node[tp].dep - node[v].dep;
    if(d>0)
        return anc[tp][d];
    else
        return des[tp][-d];
}

```

# 计算几何 TODO

## 基础板子

```cpp
/////////////////////////////////////////////////
//数据类型定义

typedef double db;
typedef long double LD;
struct Point{db x,y;};
typedef Point Vec;
struct Line{Point p; Vec v;};//点向式直线，不保证方向向量为单位向量
struct Seg{Point a,b;};//线段
struct Circle{Point o;db r;};//圆心和半径

/////////////////////////////////////////////////
//常数定义

Point const o{0.0,0.0};//原点
Line const ox{o,{1.0,0.0}}, oy{o,{0.0,1.0}};//横轴纵轴
db const PI = acos(-1);
db const EPS = 1e-9;

/////////////////////////////////////////////////
//可调整精度的比较

bool eq(db a, db b)  {return std::abs(a - b)< EPS;}//等于
bool ge(db a, db b)  {return a - b          > EPS;}//大于
bool le(db a, db b)  {return a - b          < -EPS;}//小于
bool geq(db a, db b) {return a - b          > -EPS;}//大于等于
bool leq(db a, db b) {return a - b          < EPS;}//小于等于

/////////////////////////////////////////////////
//基础运算

Vec r90a(Vec v){return {-v.y, v.x};}//向量逆时针90度
Vec r90c(Vec v){return {v.y, -v.x};}//向量顺时针90度
Vec operator+(Vec a, Vec b){return {a.x+b.x, a.y+b.y};}
Vec operator-(Vec a, Vec b){return {a.x-b.x, a.y-b.y};}
Vec operator*(db k, Vec v){return {k*v.x, k*v.y};}
Vec operator*(Vec v, db k){return {v.x*k, v.y*k};}
db operator*(Vec a, Vec b){return a.x*b.x+a.y*b.y;}
db operator^(Vec a, Vec b){return a.x*b.y-a.y*b.x;}//叉积
db len2(Vec v){return v.x*v.x+v.y*v.y;}//长度平方
db len(Vec v){return std::sqrt(len2(v));}//向量长度
db slope(Vec v){return v.y/v.x;}//斜率，不存在时，用后面的paral_y函数，不要判断是否是无穷

/////////////////////////////////////////////////
//向量操作

db cos_v(Vec a, Vec b){return a*b/len(a)/len(b);}//向量夹角余弦
Vec norm(Vec v){return {v.x/len(v), v.y/len(v)};}//求其单位向量
Vec pnorm(Vec v){return (v.x<0?-1:1)/len(v)*v;}//与原向量平行且横坐标大于零的单位向量
Vec dvec(Seg l){return l.b-l.a;}//线段转化为向量（没有归一化）

/////////////////////////////////////////////////
//直线操作

Line line(Point a, Point b){return {a,b-a};}//两点式直线
Line line(db k, db b){return {{0,b},{1,k}};}//斜截式直线y=kx+b
Line line(Point p, db k){return {p,{1,k}};}//点斜式直线
Line line(Seg l){return {l.a, l.b-l.a};}//线段所在直线
db at_x(Line l, db x){return l.p.y+(x-l.p.x)*l.v.y/l.v.x;}//给定直线上的横坐标求纵坐标，要确保直线不与y轴平行
db at_y(Line l, db y){return l.p.x+(y+l.p.y)*l.v.x/l.v.y;}//与上相反
Point pedal(Point p, Line l){return l.p-(l.p-p)*l.v/(l.v*l.v)*l.v;}//求点到直线的垂足
Line perp(Line l, Point p){return {p,r90c(l.v)};}//过某点作直线的垂线
Line bisec(Point p, Vec a, Vec b){return {p,norm(a)+norm(b)};}//角平分线

/////////////////////////////////////////////////
//线段操作

Point midp(Seg l){return {(l.a.x+l.b.x)/2.0,(l.a.y+l.b.y)/2.0};}//线段中点
Line perp(Seg l){return {midp(l), r90c(l.b-l.a)};}//线段中垂线

/////////////////////////////////////////////////
//几何关系

bool verti(Vec a, Vec b){return eq(a*b,0.0);}//向量是否垂直
bool paral(Vec a, Vec b){return eq(a^b,0.0);}//向量是否平行
bool paral_x(Vec v){return eq(v.y,0.0);}//是否平行x轴
bool paral_y(Vec v){return eq(v.x,0.0);}//是否平行y轴
bool on(Point p, Line l){return eq((p.x-l.p.x)*l.v.y, (p.y-l.p.y)*l.v.x);}//点是否在直线上
bool on(Point p, Seg l){return eq(len(p-l.a)+len(p-l.b),len(l.a-l.b));}//点是否在线段上
bool operator==(Point a, Point b){return eq(a.x,b.x)&&eq(a.y,b.y);}//点重合
bool operator==(Line a, Line b){return on(a.p,b)&&on(a.p+a.v,b);}//直线重合
bool operator==(Seg a, Seg b){return ((a.a==b.a&&a.b==b.b)||(a.a==b.b&&a.b==b.a));}//线段（完全）重合
bool operator<(Point a, Point b){return le(a.x,b.x)||(eq(a.x,b.x)&&le(a.y,b.y));}//横坐标第一关键字，纵坐标第二关键字
bool tangency(Line l, Circle c){return eq(std::abs((c.o^l.v)-(l.p^l.v)),c.r*len(l.v));}//直线和圆是否相切
bool tangency(Circle c1, Circle c2){return eq(len(c1.o-c2.o),c1.r+c2.r);}//两个圆是否相切

/////////////////////////////////////////////////
//距离

db dis(Point a, Point b){return len(a-b);}//两点距离
db dis(Point p, Line l){return std::abs((p^l.v)-(l.p^l.v))/len(l.v);}//点到直线的距离
db dis(Line a, Line b){return std::abs((a.p^pnorm(a.v))-(b.p^pnorm(b.v)));}//两直线距离，需要确保平行

/////////////////////////////////////////////////
//平移

Line operator+(Line l, Vec v){return {l.p+v, l.v};}//直线平移
Seg operator+(Seg l, Vec v){return {l.a+v,l.b+v};}//线段平移

/////////////////////////////////////////////////
//旋转

Point rotate(Point p, db rad){return {cos(rad)*p.x-sin(rad)*p.y,sin(rad)*p.x+cos(rad)*p.y};}//绕原点旋转rad弧度
Point rotate(Point p, db rad, Point c){return c+rotate(p-c,rad);}//绕c旋转rad弧度
Line rotate(Line l, db rad, Point c=o){return {rotate(l.p,rad,c),rotate(l.v,rad)};}//直线绕c点旋转rad弧度
Seg rotate(Seg l, db rad, Point c=o){return {rotate(l.a,rad,c), rotate(l.b,rad,c)};};

/////////////////////////////////////////////////
//对称

Point reflect(Point a, Point p){return {p.x*2.0-a.x, p.y*2.0-a.y};}//a关于p的对称点
Line reflect(Line l, Point p){return {reflect(l.p,p),l.v};}//直线l关于p的对称直线
Seg reflect(Seg l, Point p){return {reflect(l.a,p),reflect(l.b,p)};}//线段l关于p的对称线段

Point reflect(Point a, Line ax){return reflect(a, pedal(a,ax));}//点a关于直线ax的对称点
Point reflect_v(Vec v, Line ax){return reflect(v,ax)-reflect(o,ax);}//向量v关于直线ax的对称向量
Line reflect(Line l, Line ax){return {reflect(l.p, ax),reflect_v(l.v, ax)};}//直线l关于直线ax的对称直线
Seg reflect(Seg l, Line ax){return {reflect(l.a, ax), reflect(l.b, ax)};}

/////////////////////////////////////////////////
//交点

std::vector<Point> inter(Line a, Line b){
    //两直线的交点，没有交点返回空vector，否则返回一个大小为1的vector
    db c = a.v^b.v;
    std::vector<Point> ret;
    if(eq(c,0.0)) return ret;
    Vec v = 1/c*Vec{a.p^(a.p+a.v), b.p^(b.p+b.v)};
    ret.push_back({v*Vec{-b.v.x, a.v.x},v*Vec{-b.v.y, a.v.y}});
    return ret;
}

std::vector<Point> inter(Line l, Circle c){
    //直线与圆的交点
    Point p = pedal(c.o, l);
    db h = len(p-c.o);
    std::vector<Point> ret;
    if(ge(h,c.r)) return ret;
    if(eq(h,c.r)) {ret.push_back(p);return ret;};
    db d = std::sqrt(c.r*c.r - h*h);
    Vec v = d/len(l.v)*l.v;
    ret.push_back(p+v);ret.push_back(p+v);
    return ret;
}

std::vector<Point> inter(Circle c1, Circle c2){
    //两个圆的交点
    Vec v1 = c2.o - c1.o, v2 = r90c(v1);
    db d = len(v1);
    std::vector<Point> ret;
    if(ge(d, c1.r+c2.r)||ge(std::abs(c1.r-c2.r),d)) return ret;
    if(eq(d, c1.r+c2.r)||eq(std::abs(c1.r-c2.r),d)){ret.push_back(c1.o+c1.r/d*v1);return ret;}
    db a = ((c1.r*c1.r-c2.r*c2.r)/d+d)/2.0;
    db h = std::sqrt(c1.r*c1.r-a*a);
    Vec av = a/len(v1)*v1, hv = h/len(v2)*v2;
    ret.push_back(c1.o+av+hv);ret.push_back(c1.o+av-hv);
    return ret;
}

/////////////////////////////////////////////////
//三角形四心

Point barycenter(Point a, Point b, Point c){
    //重心
    return {(a.x+b.x+c.x)/3.0, (a.y+b.y+c.y)/3.0};
}

Point circumcenter(Point a, Point b, Point c){
    //外心
    db a2 = a*a, b2 = b*b, c2 = c*c;
    db d = 2.0*(a.x*(b.y-c.y))+b.x*(c.y-a.y)+c.x*(a.y-b.y);
    return 1/d * r90c(a2*(b-c)+b2*(c-a)+c2*(a-b));
}

Point incenter(Point a, Point b, Point c){
    //内心
    db a1 = len(b-c), b1 = len(a-c), c1 = len(a-b);
    db d = a1+b1+c1;
    return 1/d * (a1*a+b1*b+c1*c);
}

Point orthocenter(Point a, Point b, Point c){
    //垂心
    db n = b*(a-c), m = a*(b-c);
    db d = (b-c)^(a-c);
    return 1/d * r90c(n*(c-b)-m*(c-a));
}
```

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

## Pick定理

给定顶点均为整点的简单多边形，其面积$A$和内部格点数目$i$，边上格点数目$b$的关系为

$$
A = i+\dfrac{b}{2}-1
$$

# 组合数学

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
//luogu P3414，只能过50%（因为这道题考的不是这个）
using LL = long long;

const int MAXN = 200005;
const LL MOD = 6662333;

LL fac[MAXN];
LL invFac[MAXN];

LL qPowMod(LL x, LL p, LL m){
    //x^p % m
    LL ans = 1;
    while(p){
        if(p&1){
            ans = (ans*x)%m;
        }
        x = (x*x)%m;
        p>>=1;
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
    //n里面选m个
    if(n<0||m<0||m>n) return 0;
    return (((fac[n]*invFac[m])%MOD)*invFac[n-m])%MOD;
}
```

## 组合数的性质

**二项式定理**

$$
(a+b)^n = \sum^n_{i=0}\binom{n}{i}a^{n-i}b^i
$$

**对称性**

$$
\binom{n}{m} = \binom{n}{n-m}
$$

**递推式1**

$$
\binom{n}{k}=\dfrac{n}{k}\binom{n-1}{k-1}
$$

**递推式2**

$$
\binom{n}{m} = \binom{n-1}{m}+\binom{n-1}{m-1}
$$

**二项式定理的特例1**

$$
\binom{n}{0}+\binom{n}{1}+\cdots+\binom{n}{n} = 2^n
$$

**二项式定理的特例2**

$$
\sum^n_{i=0}(-1)^i\binom{n}{i} = [n=0]
$$

**组合数拆分**

$$
\sum^m_{i=0}\binom{n}{i}\binom{m}{m-i} = \binom{m+n}{m} (n\geq m)
$$

**组合数拆分的特例**

$$
\sum^m_{i=0}\binom{n}{i}^2= \binom{2n}{n}
$$

**带权和1**

$$
\sum^n_{i=0}i\binom{n}{i} = n2^{n-1}
$$

**带权和2**

$$
\sum^n_{i=0}i^2\binom{n}{i} = n(n+1)2^{n-2}
$$

**性质1**

$$
\sum^n_{l=0}\binom{l}{k} = \binom{n+1}{k+1}
$$

**性质2**

$$
\binom{n}{r}\binom{r}{k} = \binom{n}{k}\binom{n-k}{r-k}
$$

**斐波那契数列性质**

$$
\sum^n_{i=0}\binom{n-i}{i} = F_{n+1}
$$

## 圆排列

$n$个人全部来围成一圈，所有的排列数记为$Q^n_n$。考虑其中已经拍好的一圈，从不同位置断开可以变成不同的队列，则有

$$
Q^n_n\times n = A^n_n
$$

由此可知

$$
Q^r_n = \dfrac{A^r_n}{r}
$$

## 二项式反演

记$f_n$表示恰好使用$n$个不同元素形成特定结构的方案数，$g_n$表示从$n$个不同元素中选出$i\geq 0$个元素形成特定结构的总方案数。有

$$
g_n = \sum^n_{i=0}\binom{n}{i}f_i
$$

二项式反演就是已知$g_n$求$f_n$

$$
f_n = \sum^n_{i=0}\binom{n}{i}(-1)^{n-i}g_i
$$

## 斐波那契数列的性质

**通项公式**

$$
F_n = \dfrac{\left(\dfrac{1+\sqrt{5}}{2}\right)^n-\left(\dfrac{1-\sqrt{5}}{2}\right)^n}{\sqrt{5}}
$$

$$
F_n = \left[\dfrac{\left(\dfrac{1+\sqrt{5}}{2}\right)^n}{\sqrt{5}}\right]
$$

中括号表示取离他最近的整数。

**性质1**

$$
F_{n-1}F_{n+1}-F_n^2=(-1)^n
$$

**性质2**

$$
F_{n+k}=F_kF_{n+1}+F_{k-1}F_n
$$

**性质3**

$$
F_{2n}=F_{n}(F_{n+1}+F_{n-1}),\quad F_{2n+1}=F^2_{k+1}+F^2_k
$$

**性质4**

$$
\forall k\in N,F_n|F_{nk}
$$

**性质5**

$$
\forall F_a|F_b,a|b
$$

**性质6**

$$
\gcd(F_m,F_n)=F_{\gcd(m,n)}
$$

**性质7**

$$
F_{n+1}=\sum_{0\leq i\leq n}\binom{n-i}{i}
$$

## 和式性质

### 基本性质

$$
\sum_{k\in K}ca_k=c\sum_{k\in K}a_k
$$

$$
\sum_{k\in K}(a_k+b_k)=\sum_{k\in K}a_k+\sum_{k\in K}b_k
$$

$$
\sum_{k\in K}a_k = \sum_{p(k)\in K}a_{p(k)}
$$

此处$p(k)$是$k$的任意排列。

### 多重和式分配律

$$
\sum_{j\in J,k\in K}a_jb_k = (\sum_{j\in J}a_j)(\sum_{k\in K}b_k)
$$

### 多重和式次序交换

$$
\sum_{j\in J}\sum_{k\in K}a_{j,k} = \sum_{j\in J,k\in K}a_{j,k} = \sum_{k\in K}\sum_{j\in J}a_{j,k}
$$

当$J,K$相互独立时成立。

$$
\sum_{j\in J}\sum_{k\in K(j)}a_{j,k} = \sum_{k\in K'}\sum_{j\in J'(k)}a_{j,k}
$$

这里$J,K$不独立，并且要满足

$$
[j\in J][k\in K(j)]=[k\in K'][j\in J'(k)]
$$
例如

$$
\sum_{j=1}^n\sum_{k=j}^na_{j,k}=\sum_{i\leq j\leq k\leq n}a_{j,k}=\sum_{k=1}^n\sum_{j=1}^ka_{j,k}
$$

## 卡特兰数

TODO: 用模板元编程实现编译期算卡特兰数

第$n$个记作$C_n$

$n$对括号形成的字符串，合法的情况数是$C_n$

$n$个节点的二叉树，总共有$C_n$种

$2n+1$个节点组成的满二叉树，有$C_n$种

$n\times n$的格点网中，从左下角格点出发，到达右上角格点，不穿过对角线（但可以碰到）的单调路径个数有$C_n$个。

在圆上有$2n$个点，将这些点成对连接起来使得所得到的$n$条线段不相交的方法数为$C_n$种

一个栈（无穷大）的进栈序列为$1,2,3,\cdots,n$，合法的出栈序列有$C_n$个

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
//前几项：1, 1, 2, 5, 14, 42, 132, 429, 1430, 4862, 16796
//luogu p1044
using namespace std;

typedef long long ll;
const int MAXN = 3005;

ll h[MAXN];

ll comb(ll a,ll b){
    ll ans=1;
    for(ll i=1;i<=b;i++){
        ans*=a;//数字太大会爆
        a--;
    }
    for(ll i=1;i<=b;i++){
        ans/=i;
    }
    return ans;
}

int main(){
    ll n;
    cin>>n;
    for(ll i=1;i<=n;i++){
        cout<<comb(2*i,i)/(i+1)<<endl;//n>=15的时候ll都能爆
    }
    cout<<"###"<<endl;
    //下面是递推求法，不容易爆
    h[1]=1;
    cout<<h[1]<<endl;
    for(ll i=2;i<=n;i++){
        h[i] = h[i-1]*(4*i-2)/(i+1);
        cout<<h[i]<<endl;
    }

    return 0;
}
```

## 生成函数

生成函数是一种形式幂级数

$$
F(x) = \sum_na_nx^n
$$

$\{a_i\}$序列可以是有限的，也可以是无限的。$a_i$下标以$0$为起点。生成函数例如

1. $a=<1,2,3>$的普通生成函数为$1+2x+3x^2$
2. $a=<1,1,1,\cdots>$的普通生成函数为$\sum_{n\geq 0}x^n$
3. $a=<2,4,6,8,\cdots>$的普通生成函数为$\sum_{n\geq 0}(2n+2)x^n$

**加减运算**

设序列$a,b$的普通生成函数分别为$F(x),G(x)$，则

$$
F(x)\pm G(x) = \sum_n(a_n\pm b_n)x^n
$$

**乘/卷积运算**

$$
F(x)G(x) = \sum_n x^n\sum^n_{i=0}a_i b_{n-i}
$$

给出一些常见封闭形式，这其实和幂级数收敛时的求和公式差不多（目前只会普通生成函数来应对组合问题，之后更新指数生成函数应对排列问题TODO）：

$$
\sum_{n\geq 0}x^n = \dfrac{1}{1-x}
$$

$$
\sum_{n\geq 0}p^nx^n=\dfrac{1}{1-px}
$$

$$
\sum_{n\geq 1}x^n = \dfrac{x}{1-x}
$$

$$
\sum_{n\geq 0}x^{cn} = \dfrac{1}{1-x^c}
$$

$$
1+2x+3x^2+\cdots = \sum_{n\geq 0}(n-1)x^n = \dfrac{1}{(1-x)^2} 
$$

$$
\sum_{n\geq 0}\binom{m}{n}x^n = (1+x)^m
$$

$$
\sum_{n\geq 0}\binom{m+n-1}{n}x^n = \dfrac{1}{(1-x)^{m}}
$$

其他有限项生成函数应该用等比数列求和公式，转化成分式形式。之后再来进行生成函数的计算。

**例题**

在许多不同种类的食物中选出$n$个，每种食物的限制如下（每种食物选出来的个数必须满足该限制）

1. 汉堡：偶数个
2. 可乐：0或1个
3. 鸡腿：0或1或2个
4. 蜜桃多：奇数个（注：0个不满足条件）
5. 鸡块：4的倍数个
6. 包子，0、1、2、3个
7. 土豆炒肉：不超过1个
8. 面包：3的倍数个

所有食物选出来的总数加起来等于$n$就可以算作一种方案。计算方案总数模$10007$

我们设$a_n$表示这种食物选$n$个的方案数，并求出其生成函数。显然，假设只选两个食品，如果食品1选了$i$个，那么食品2就只能选$n-i$个。这和我们之前的卷积形式是一样的。所以我们应该把各种食品的生成函数的封闭形式乘起来得到答案。生成函数构造如下

1. $\sum_{n\geq 0}x^{2n}=\dfrac{1}{1-x^2}$
2. $1+x$
3. $1+x+x^2=\dfrac{1-x^3}{1-x}$（这里食品都是相同的，所以选1个只有1种方案。求法是等比数列求和。有些题的物品是不同的，这里就要变成其他序列）
4. $\dfrac{x}{1-x^2}$
5. $\dfrac{1}{1-x^4}$
6. $\dfrac{1-x^4}{1-x}$
7. $1+x$
8. $\dfrac{1}{1-x^3}$

全部乘起来得到的生成函数为

$$
F(x) = \dfrac{(1+x)(1-x^3)x(1-x^4)(1+x)}{(1-x^2)(1-x)(1-x^2)(1-x^4)(1-x)(1-x^3)} = \dfrac{x}{(1-x)^4}
$$

再转化为展开形式

$$
F(x) = \sum_{n\geq 0}\binom{n+3}{n}x^{n+1}=\sum_{n\geq 1}\binom{n+2}{n-1}x^n
$$

可得答案就是$\binom{n+2}{n-1}=\binom{n+2}{3}$

## 稳定婚姻问题(Gale-Shapley算法) TODO

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

# 数据结构 

## 树状数组

```cpp
//复杂度 单次查询 logn 单次修改 logn
//树状数组，维护的是数组的前缀和，有大量的应用
//luogu P3374

//普通的树状数组要维护的信息，其运算要满足结合律和可差分
//结合律不难理解，可差分指的是若已知x op y和x，则可以求出y
//这样的运算例如加，乘，异或。乘如果在模意义下可差分，需要保证每个数都有逆元，如果模数为质数则肯定有
//gcd,max这种是不可差分的

#include <iostream>

int const MAXN = 1000005;
using LL = long long;

class Fenwick{
public:
    LL data[MAXN];
    int size = 0;
    
    void init(int size_){size=size_;}
    
    inline int lowbit(int x){
        return x&(-x);
    }
    
    void update(int p, LL k){//位置p的元素加k
        for(;p<=size;p+=lowbit(p)){
            data[p]+=k;
        }
    }
    
    LL query(int p){//查询[1,p]的和
        LL ret=0;
        for(;p;p-=lowbit(p)){
            ret += data[p];
        }
        return ret;
    }
};

Fenwick fenwick;

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    
	int n,m;
    std::cin>>n>>m;
    //数组长度，查询数
    fenwick.init(n);
    for(int i=1;i<=n;i++){
        LL tmp;
        std::cin>>tmp;
        fenwick.update(i,tmp);
    }

    for(int i=1;i<=m;i++){
        int op;
        int x,y;
        LL k;
        std::cin>>op>>x;
        if(op==1){
            std::cin>>k;
            //将单点增加k，如果想要改成修改，则可以update(x,k-查询x位置上的数)
            fenwick.update(x,k);
        }
        else{
            std::cin>>y;
            //输出[x,y]的数组和
            std::cout<<fenwick.query(y)-fenwick.query(x-1)<<"\n";
        }
    }
    
    return 0;
}
```

### 树状数组求逆序对

```cpp
//复杂度 nlogn
//Luogu P1908
//逆序对<i,j>即，符合i<j且ai>aj的<i,j>的个数
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

### 二维树状数组

```cpp
//二维树状数组 支持单点修改和区间查询
//loj 133
#include <iostream>

int const MAXN = 5005;
using LL = long long;

class BIT2D{
public:
    int N,M;
    LL data[MAXN][MAXN];
    
    void init(int n,int m){
        N = n, M = m;
    }
    
    inline int lowbit(int x){
        return x&(-x);
    }
    
    void add(int x, int y, LL v){//把(x,y)这个点加上v
        for(int i=x;i<=N;i+=lowbit(i)){
            for(int j=y;j<=M;j+=lowbit(j)){
                data[i][j] += v;
            }
        }
    }
    
    LL sum(int x, int y){
        LL ret = 0;
        for(int i=x;i>0;i-=lowbit(i)){
            for(int j=y;j>0;j-=lowbit(j)){
                ret += data[i][j];
            }
        }
        return ret;
    }
    
    LL query(int x1, int y1, int x2, int y2){//查询(x1,y1)-(x2,y2)这个矩形的区间和
        return sum(x2,y2) - sum(x2,y1-1) - sum(x1-1, y2) + sum(x1-1, y1-1);
    }
};

BIT2D bit2d;
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
//luogu p3372
//线段树维护的数据要求满足结合律，比如区间和，区间最大区间最小，区间gcd
//区间修改一般支持加、乘、赋值

#include <iostream>

int const MAXN = 100005;
using LL = long long;

struct Node
{
    int s,t;//该端点的起点和终点下标
    LL tag, v;
};

Node st[MAXN*4+2];
LL arr[MAXN];

void build(int s, int t, int p=1){
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

void update(int l, int r, LL k, int p=1){
    int s = st[p].s, t = st[p].t;
    if(l<=s && t<=r){
        st[p].v   += (t-s+1) * k;
        st[p].tag += k;
        return;
    }
    spreadTag(p);
    
    int m = s+((t-s)>>1);
    if(l<=m) update(l, r, k, p*2);
    if(r>m)  update(l, r, k, p*2+1);
    st[p].v = st[p*2].v + st[p*2+1].v;
}

LL query(int l, int r, int p=1){
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

## 动态开点线段树

```cpp
//动态开点线段树
//luogu p3372，由于并没有找到合适的习题，我把题中的查询范围全部加了一个偏移值-5e4

//动态开点线段树并不是动态分配内存，只是在范围很大，但查询不多的时候，可以用
//普通线段树的空间复杂度是O(n)，单次操作的时间复杂度是O(logn)
//动态开点，设查询为m次，时间复杂度仍然为O(logn)，但是空间复杂度变成O(mlogn)
//动态开点还可以处理查询范围为负数的情况，比如查询[-5,6]这一段上的和
//动态开点假设初始数组全部为0，输入一个数组时直接add修改线段树即可

using LL = long long;
int const MAXN = 8e6+5;//能开多大开多大，128M可以开到800万

struct Node{
    LL val, tag;
    int ls,  rs;
}st[MAXN];

inline int& ls(int x){return st[x].ls;}
inline int& rs(int x){return st[x].rs;}
inline LL& val(int x){return st[x].val;}
inline LL& tag(int x){return st[x].tag;}

inline int getMid(int s, int t){
    //处理负数边界时，需要强行向下取整，而不是向零取整
    if(s+t==0) return 0;
    if(s+t>0) return (s+t)/2;
    return -((-s-t+1)/2);
}

int stcnt = 1;
int L = -(1e6+5), R = 1e6+5;//这里根据题目信息选择区间范围

void upd(int &p, LL k, int len){
    if(!p) p = ++stcnt;
    val(p) += k * len;
    tag(p) += k;
}

void spreadTag(int p, int len){
    if(len<=1) return;
    upd(ls(p), tag(p), len-len/2);
    upd(rs(p), tag(p), len/2);
    tag(p) = 0;
}

LL query(int l, int r, int p = 1, int s = L, int t = R){
    if(s>=l && t<=r) return val(p);
    spreadTag(p, t-s+1);
    int mid = getMid(s,t);
    LL ret = 0;
    if(mid >= l) ret += query(l, r, ls(p), s, mid);
    if(mid < r)  ret += query(l, r, rs(p), mid+1, t);
    return ret;
}

void add(int l, int r, LL k, int p = 1, int s = L, int t = R){
    if(s>=l && t<=r){
        val(p) += k * (t-s+1);
        tag(p) += k;
        return;
    }
    spreadTag(p, t-s+1);
    int mid = getMid(s,t);
    if(mid >= l) add(l, r, k, ls(p), s, mid);
    if(mid < r)  add(l, r, k, rs(p), mid+1, t);
    val(p) = val(ls(p)) + val(rs(p));
}

void solve(){
    int n,m;
    std::cin>>n>>m;
    int const offset = -5e4;
    for(int i=1;i<=n;i++){
        int a;
        std::cin>>a;
        add(i+offset,i+offset,a);
    }
    while(m--){
        int ope, l, r, x;
        std::cin>>ope>>l>>r;
        l += offset;
        r += offset;
        if(ope==1){
            std::cin>>x;
            add(l,r,x);
        }
        else{
            std::cout<<query(l,r)<<"\n";
        }
    }
}
```

## 权值线段树

```cpp
//权值线段树
//loj 104

//权值线段树大部分时候是用来代替平衡树使用的
//和树状数组求逆序对很像，他把每一个[x,x]范围上的节点视作一个桶，插入数据时，add(x,x,1)，删除一个时add(x,x,-1)。
//查询复杂度为O(logv)，v为值域

/*这里是动态开点线段树的核心代码*/

void insert(int v){//插入一个数
    add(v,v,1);
}

void remove(int v){//删除一个数，相同的只删一个
    add(v,v,-1);
}

int countL(int v){//计算小于v的数的个数
    return query(L, v-1);
}

int countG(int v){//计算大于v的数的个数
    return query(v+1, R);
}

int rank(int v){//求v的排名，即小于v的数的个数+1
    return countL(v)+1;
}

int kth(int k, int p=1, int s=L, int t=R){//查询排名第k的数
    if(s==t) return s;
    int mid = getMid(s,t);
    if(val(ls(p)) >= k)
        return kth(k, ls(p), s, mid);
    else
        return kth(k-val(ls(p)), rs(p), mid+1, t);
}

int pre(int v){//查询v的前驱，即第一个比v小的数，可能需要保证一定存在
    int r = countL(v);
    return kth(r);
}

int suc(int v){//查询v的后继
    int r = val(1) - countG(v) + 1;
    return kth(r);
}
```

## 珂朵莉树

```CPP
//珂朵莉树，区间推平问题
//方便给某个区间赋值，区间加数，维护区间第k大值，区间和等等
//数据随机的情况下，复杂度为nloglogn
//珂朵莉树的每一个节点都是一个区间，这个区间内的值相同。
//cf896c
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
//luogu p3372 和线段树区间加，维护区间和一样
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

## 01字典树

```cpp
//01字典树，复杂度线性
//HDU4825
int const MAXN = 3500005;
int const MAXBIT = 35;//注意题目给的数据范围，这里2^32以下可以处理

class Trie{
public:
    int nxt[MAXN][2];
    int cnt;
    LL num[MAXN];
    
    void init(){
        for(int i=0;i<=cnt;i++) for(int j=0;j<2;j++) nxt[i][j] = 0;
        for(int i=0;i<=cnt;i++) num[i] = 0;
        cnt = 0;
    }
    
    void insert(LL n){
        //插入一个自然数
        int cur = 0;
        for(LL i=MAXBIT;i>=0;i--){
            LL bit = (n>>i)&1;
            if(!nxt[cur][bit]){
                nxt[cur][bit] = ++cnt;
            }
            cur = nxt[cur][bit];
        }
        num[cur] = n;
    }
    
    LL find_max(LL x){
        //查询x与数组内的所有数的异或的最大值
        int cur=0;
        for(int i=MAXBIT;i>=0;i--){
            LL bit = (x>>i)&1;
            if(nxt[cur][bit^1])//尽量走与当前位不同的路径，最小值应改为走相同的
                cur = nxt[cur][bit^1];
            else
                cur = nxt[cur][bit];
        }
        return x^num[cur];
    }
};

Trie trie;
```

可以通过01Trie来计算连续区间的异或最大值。要用到一个性质：

$$
a\oplus b\oplus b = a
$$

也就是说，我们可以把异或前缀全部插入到Trie里，然后以第i个数为结尾的区间的最大异或值就是find_max(pre[i])。注意特殊处理长度为1的区间。

```cpp
LL ans = 0;
arr[0] = 0;

for(int i=1;i<=n;i++){
	std::cin>>arr[i];
	ans = std::max(ans,arr[i]);
	arr[i] ^= arr[i-1]; 
}

trie.insert(0);//注意要先插入一个0
for(int i=1;i<=n;i++){
	ans = std::max(ans,trie.find_max(arr[i]));
	trie.insert(arr[i]);
}

std::cout<<ans<<"\n";
```

## 对顶堆

```cpp
//动态维护一个集合的第k大数，每次操作logn
//spoj RMID2
//维护第k小只要维护第n-k大即可
//另外这个k是可以变化的，不需要固定，复杂度确实是logn

#include <iostream>
#include <algorithm>
#include <queue>
#include <vector>

template<typename T>
class KthLargest{
private:
    std::priority_queue<T,std::vector<T>,std::less<T> > big{};
    std::priority_queue<T,std::vector<T>,std::greater<T> > small{};
    size_t kth{};
    size_t size{};
    
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
    
    T findK(size_t k){
		//找到第k大的数字
        kth = k;
        update();
        return small.top();
    }
    
    void eraseK(size_t k){
		//移除第k大的数字
        kth = k;
        update();
        small.pop();
        size--;
        update();
    }
    
    void insert(T x){
		//插入一个数字
        size++;
        if(small.empty() || x>=small.top()){
            small.push(x);
        }
        else{
            big.push(x);
        }
        update();
    }
    
    size_t getSize(){
        return size;
    }
};

KthLargest<int> ddd;

```

## 单调栈

```cpp
//单调栈 luogu p5788
//本题定义f[i]为数列中第i个元素之后第一个大于a[i]的元素的下标（不存在则为0）
//很显然我们可以维护一个单调不增的栈
//当push的元素x大于栈顶t时，第一个大于t的元素就是x。反复出栈直到栈顶t小于等于x或栈空，入栈。
//复杂度 n
int arr[MAXN];
int ans[MAXN];
int stk[MAXN];

int main(){
    int n;
    std::cin>>n;

    for(int i=1;i<=n;i++){
        std::cin>>arr[i];
    }

    int top = 0;

    for(int i=1;i<=n;i++){
        while(top&&arr[stk[top]]<arr[i]){
            ans[stk[top]] = i;//这一行是具体的操作，因题而异；而其他行在这个for循环里都是固定的
            top--;
        }

        stk[++top] = i;
    }

    for(int i=1;i<=n;i++){
        std::cout<<ans[i]<<" ";
    }

    return 0;
}
```

## 单调队列

```cpp
//单调队列，luogu1886
//本题是滑动窗口，即在长度为n的数组中，给出一个长度为k的连续区间，从左向右滑动，求每个区间中的最大值和最小值
//求最小值时，我们可以维护一个单增的双端队列。x加入队尾时，如果队尾元素b>=x，则把队尾弹出，直到b<x或者栈空时把x入队。
//因为我们的区间长度有限，每次我们的区间左端点向右枚举+1时，判断队首元素的下标，如果小于区间左端点，就出队。
//之后留在队首的元素就是区间最小值
//具体可见代码。最大值维护同理。

//复杂度n
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
        std::cout<<arr[dq.front()]<<" ";//输出最小值
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
        std::cout<<arr[dq.front()]<<" ";//输出最大值
    }

    return 0;
}
```

## ST表

对于经典的RMQ（即给定一个数组，求区间内的最大值）问题，有如下代码

```cpp
//复杂度 单次查询 logn 预处理 nlogn
//luogu P3865
//查询区间最大值
//也可以查询其他可重复贡献问题的信息
//可重复贡献指对于运算op，满足x op x = x。这样的运算有最大最小、gcd等。但显然求和不是。
//另外op还必须满足结合律。

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
        //完全不用担心这两个范围重叠，因为是求max
        int s = logn[b-a+1];
        printf("%d\n",std::max(fmax[a][s],fmax[b-(1<<s)+1][s]));
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

当验证一个情况是否能满足题目的复杂度小于等于$O(n)$，而且这些情况具有单调性（即例如若x>y，x不能满足，则y一定不能满足）时，就可以通过二分去得到最符合题意的答案。二分这些情况的复杂度为$O(\log n)$，再乘上验证情况的复杂度得到总的复杂度。

judge函数应该根据题意写出。

如果是浮点数的二分，则不推荐使用EPS进行精度判断（有可能会丢精度）。而是使用计数器，一般迭代100次就能保证符合题目要求。

## 二分查找

通常是在排好序上的数组中，查找第一个大于（或大于等于）x的元素。见STL用法中的lower_bound和upper_bound。每次查找的复杂度是logn

## 二分求单调函数零点

设函数$f$在$[l,r]$上严格单调，$mid=(l+r)/2$，显然有$f(l)f(r)<0$。迭代中，若$f(l)f(mid)<0$，则$r=mid$，否则$l=mid$。直到$f(mid)=0$或者$r-l<EPS$或者迭代次数达到要求。收敛速度是线性收敛。

# 三分法

## 三分法求单峰函数的极值点

用二分求函数的导数的零点也可以，但是并不是每次都可以方便的求出导数。三分法可以不用求出导数。

设函数$f$在$[l,r]$上单峰，意味着有且只有一个极大值$x$，$f$在$[l,x]$上严格单增，在$[x,r]$上严格单减。单谷函数则为极小值$x$。

```cpp
//三分法求单峰函数的极值点 luogu p3382
//收敛速度是线性收敛
//用二分求函数的导数的零点也可以，但是并不是每次都可以方便的求出导数。三分法可以不用求出导数。
//设函数f在[l,r]上单峰，意味着有且只有一个极大值x，f在[l,x]上严格单增，在[x,r]上严格单减。单谷函数则为极小值x。

//在[l,r]上取两个不等的点，设靠近l的是l1，靠近r的是r1。如果f(l1)<f(r1)，说明极大值一定在[l1,r]，令l=l1
//如果f(l1)>f(r1)，极大值一定在[l,r1]，令r=r1
//持续下去直到r-l<EPS或者迭代次数足够

//取l1和r1时，可以直接取三等分点，也可以取黄金分割点(l1=l+(r-l)(1-0.618),r1=r-(r-l)*(1-0.618))
//还可以让l1=mid-EPS, r1=mid-EPS，但是要令l=mid而不是l=l1，防止死循环

using DB = double;
DB const EPS = 1e-8;
DB l,r;
std::cin>>l>>r;

while(r-l>EPS){
    DB mid = (l+r)/2;
    DB f1 = func(mid-EPS), f2 = func(mid+EPS);//func根据题目要求定义，是一元函数
    if(f1<f2)
        l = mid;
    else
        r = mid;
}
```

## 三分套三分

例如Luogu P2571，这是一个二元函数要求最小值。我们发现这个函数在固定$x$的时候$y$是单谷的，固定$y$的时候$x$是单谷的。所以我们可以先三分一个变量，再固定这个变量三分另一个变量，最后得出答案。

注意，三分套三分是指不能先假设一个$y$的定值，再三分$x$，然后拿着计算出的$x$再去三分$y$。应当在三分$x$的过程中，把$x$当作参数，传入三分$y$的函数中。它们不是先后关系，而是嵌套关系。

```cpp
auto func = [&](DB x, DB y){
    //这里是函数定义
};

auto sfy = [&](DB x){//固定x，三分y
    DB ret = 0.0;
    DB l = 0.0, r = 1.0;
    while(r-l>EPS){
        DB delta = (r-l)/3.0;
        DB f1 = func(x,l+delta), f2 = func(x,r-delta);
        if(f1>f2)
            l = l+delta;
        else
            r = r-delta;
    }
    return func(x,l);
};

DB l=0.0,r=1.0;
while(r-l>EPS){//三分x
    DB delta = (r-l)/3.0;
    DB f1 = sfy(l+delta), f2 = sfy(r-delta);
    if(f1>f2)
        l = l+delta;
    else
        r = r-delta;
}
//最后的答案是sfy(l)
```

## 三分答案 TODO

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
//即每种物品有ki个
//我们可以简单转化为01背包，但是复杂度太高；采用二进制分组的思想
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

## 分组背包

```cpp
//分组背包 复杂度=组数*背包容量*组内个数最大值
//luogu p1757
//有g个组，每组物品有group[i].size()个，每个物品有价值v和体积w，总共n个物品，背包体积为m。每个组最多只能拿一个物品出来，求最大价值。
#include <iostream>
#include <algorithm>
#include <cstring>
#include <vector>

int const MAXN = 1005;

int dp[MAXN];
std::vector<int> group[MAXN];//存储每一组的物品序号
int v[MAXN],w[MAXN];//物品价值和体积

void solve(){
    int n,m;
    std::cin>>m>>n;//背包容量;n件物品
    
    for(int i=1;i<=n;i++){
        int a,b,c;
        std::cin>>a>>b>>c;
        w[i] = a, v[i] = b,group[c].push_back(i);
    }
    
    int const g = 100;//本题中只说了有g个组，没有给出具体有多少个以及是否连续，采取遍历的方法
    for(int i=1;i<=g;i++){
        if(group[i].size()==0) continue;
        for(int j=m;j>=0;j--){
            for(auto k:group[i]){//注意这三个循环的顺序不能改变
                if(j>=w[k]) dp[j] = std::max(dp[j], dp[j-w[k]]+v[k]);
            }
        }
    }
    std::cout<<dp[m]<<"\n";
    
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);

	int T;
	T=1;
	while(T--){
	    solve();
	}

    return 0;
}

```

## 最长上升子序列

```cpp
//最长上升子序列 复杂度nlogn
//luogu B3637
#include <iostream>
#include <algorithm>
#include <cstring>

const int MAXN = 100005;

int arr[MAXN];
int dp[MAXN];
//dp[i]表示长度为i的上升子序列的最后一个元素的最小值
//例如1 2 5 3 4 1。
//最开始dp[1]=1，arr[2]=2>dp[1]，所以插入dp[2]=2;arr[3]同理，得到dp={1,2,5};到arr[4]=3时，我们找到第一个大于等于3的元素，即dp[3]，替换他，得到dp={1,2,3};接下来到arr[5]=4，现在4>3，可以插入末尾得到dp={1,2,3,4}，显然我们刚刚的操作把5换成3，让后面的数更有可能直接加入到数组末尾了。最后arr[6]=1，由于我们求的是最长上升子序列，而不是最长不下降，所以替换不影响。
//注意dp里面的数字并不是最长的序列，例如我们加一个0进去，dp={0,2,3,4}，但是0是在最后的，不存在0,2,3,4这个序列。我们只能计算长度。

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    
    int n;
    std::cin>>n;
    for(int i=1;i<=n;i++){
        std::cin>>arr[i];
    }
    
    std::memset(dp,0x3f,sizeof(dp));
    
    int maxv = dp[0];
    for(int i=1;i<=n;i++){
        *std::lower_bound(dp,dp+n,arr[i]) = arr[i];
        //换成最长不下降子序列时，用upper_bound
    }
    int ans = 0;
    while(dp[ans]!=maxv) ans++;
    std::cout<<ans<<"\n";

    return 0;
}

```

## Dilworth定理

把一个序列分为若干不上升子序列，其序列总数的最小值等于最长上升子序列的长度

## 最长公共子序列

```cpp
//最长公共子序列，复杂度nm
//hdu 1159
//luogu p1439因为是排列，可以转化为LIS问题，复杂度是nlogn。但hdu1159没有这种性质，复杂度到不了nlogn
#include <iostream>
#include <cstring>
#include <string>
#define MAXN 505

int dp[MAXN][MAXN];

int lcs(std::string const & s1, std::string const & s2){
    int n1=s1.size(),n2=s2.size();
    std::memset(dp,0,sizeof(dp));
    
    for(int i=1;i<=n1;i++){
        for(int j=1;j<=n2;j++){
            if(s1[i-1]==s2[j-1])
                dp[i][j] = dp[i-1][j-1]+1;
            else
                dp[i][j] = std::max(dp[i][j-1],dp[i-1][j]);
        }
    }
    return dp[n1][n2];
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);
    
    std::string s1,s2;
    
    while(std::cin>>s1>>s2){
        std::cout<<lcs(s1,s2)<<"\n";
    }
    
    return 0;
}
```

## 斜率优化TODO

## 四边形不等式TODO

## 悬线法

```cpp
//luogu p1387
//悬线法求符合条件的最大矩形/正方形，复杂度 nm

int grid[MAXN][MAXN];
int l[MAXN][MAXN], r[MAXN][MAXN], u[MAXN][MAXN];
//l,r分别表示从当前格向上的悬线最多能向左向右扩展多少格（含自己）
//u表示当前格向上能扩展多少格（含自己），即悬线

void dp(int n, int m){
    for(int i=1;i<=n;i++){
        for(int j=1;j<=m;j++){
            if(grid[i][j]) l[i][j] = l[i][j-1]+1;//若(i,j-1)可选，当然首先要(i,j)可选，(i,j-1)不可选时l[i][j-1]会等于0，l[i][j]就会等于1
            //这里的if条件看情况选择，下面同理
        }
    }
    for(int i=1;i<=n;i++){
        for(int j=m;j>=1;j--){
            if(grid[i][j]) r[i][j] = r[i][j+1]+1;//若(i,j+1)可选
        }
    }
    for(int i=1;i<=n;i++){
        for(int j=1;j<=m;j++){
            if(grid[i][j]){
                u[i][j] = u[i-1][j]+1;
                if(grid[i-1][j]){//若(i-1,j)可选
                    l[i][j] = std::min(l[i][j], l[i-1][j]);
                    r[i][j] = std::min(r[i][j], r[i-1][j]);
                }
                //然后在这里对ans进行该有的操作，因题而异
                //对于(i,j)这一格来说，它对应的悬线向左右拓展能得到的最大矩形面积为
                //u[i][j]*(l[i][j]+r[i][j]-1)
                //最大正方形为
                //min(u[i][j],l[i][j]+r[i][j]-1)的平方
            }
        }
    }
}
```

## 数位DP

数位DP是对有多少数符合特性的计数问题。通常他的题目数据范围会很大，比如$10^{18}$。题目通常也会要求我们的数字要在某个范围内，还有可能会要求符合题意的一对、一组数。

我们通常会用记忆化搜索来实现。

```cpp
//数位dp模板题，常用记忆化搜索实现
//hdu 2089
//本题要求，[n,m]之间的所有整数，不含4，不含62（连续的）的数字有多少个
#include <iostream>
#include <algorithm>
#include <cmath>
#include <cstring>
#include <vector>

#define pb push_back

std::vector<int> digit;//用于存储getSum中x的每一位，最高位从0下标开始
int dp[8][12][2];//dp[pos][last][limit]
//pos代表搜索到第几位，如五位数，pos==0说明搜索到高位第一位，pos==4说明搜索到个位
//last代表上一位搜索的数字是多少，如果last==11，10这样的数字则代表目前在搜索pos==0。设置为11还是10还是别的什么要看题目对只有一位数时的要求
//limit代表本位数字的取值有没有限制，例如上限是12345，现在搜到了12???，要搜第三位，显然第三位只能取0,1,2,3，limit==true。又如搜到了11???，第三位就可以取0-9，limit==false。
//limit==true当且仅当上一位limit也为true且取得最大值（第一位特判）

int dfs(int pos, int last, bool limit){
    int ret = 0;
    if(pos==digit.size()) return 1;//搜索终点，由于不是非法状态所以返回1
    if(dp[pos][last][limit] != -1) return dp[pos][last][limit];
    for(int v=0;v<=(limit ? digit[pos] : 9);v++){
        if((last==6 && v==2) || v==4) continue;//非法状态
        ret += dfs(pos+1, v, limit && v==digit[pos]);
    }
    dp[pos][last][limit] = ret;
    return ret;
}

int getSum(int x){
    digit.clear();
    std::memset(dp,-1,sizeof(dp));
    while(x){//注意如果某些题0也在范围内要特判
        digit.pb(x%10);
        x/=10;
    }
    std::reverse(digit.begin(),digit.end());//高位到低位存
    return dfs(0,10,true);
}

void solve(int n, int m){
    std::cout<<getSum(m)-getSum(n-1)<<"\n";
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);

	int n,m;
	while(std::cin>>n>>m){
	    if(n==0 && m==0) break;
	    solve(n,m);
	}

    return 0;
}
```

下面再给出一例，求一对数字的、与位运算有关的，拆分成二进制的题。

```cpp
//数位dp另一例
//atcoder abc317_f
//本题给定n,a1,a2,a3。要求求三元组<x1,x2,x3>的个数，满足
//1. 1<=xi<=n，对所有i
//2. xi是ai的任意倍数，对所有i
//3. x1^x2^x3=0
//其中n取值[1,1e18]
//ai取值[1,10]

#define pb push_back
using LL = long long;
LL const MOD = 998244353;

std::vector<int> digit;
int dp[80][12][12][12][2][2][2][2][2][2];
int a1,a2,a3;

int dfs(int pos, int r1, int r2, int r3,
        bool l1, bool l2, bool l3,
        bool z1, bool z2, bool z3){
    //分布代表着，pos位数字，上一位搜索到的x1除以a1的余数,...,x1的limit,...,x1是否前面全是前导0
    int ret = 0;
    if(pos==-1){
        return !z1 && !z2 && !z3 && !r1 && !r2 && !r3;
        //每个数都没有前导零（即填入了至少一个数字），以及余数都是0（即xi已经是ai的倍数了）
    }
    if(dp[pos][r1][r2][r3][l1][l2][l3][z1][z2][z3] != -1) return dp[pos][r1][r2][r3][l1][l2][l3][z1][z2][z3];
    
    int m1 = l1 ? digit[pos] : 1;
    int m2 = l2 ? digit[pos] : 1;
    int m3 = l3 ? digit[pos] : 1;
    
    for(LL i=0;i<=m1;i++){
        for(LL j=0;j<=m2;j++){
            for(LL k=0;k<=m3;k++){
                if((i^j^k)!=0) continue;
                
                int newr1 = ((LL)r1+(i<<pos))%a1;//计算新的余数
                int newr2 = ((LL)r2+(j<<pos))%a2;
                int newr3 = ((LL)r3+(k<<pos))%a3;
                
                ret = (ret + dfs(pos-1,newr1,newr2,newr3,
                l1&&i==digit[pos], l2&&j==digit[pos],l3&&k==digit[pos],
                z1&&i==0,z2&&j==0,z3&&k==0))%MOD;
            }
        }
    }
    
    dp[pos][r1][r2][r3][l1][l2][l3][z1][z2][z3] = ret;
    return ret;
}

int getSum(LL x){
    digit.clear();
    std::memset(dp,-1,sizeof(dp));
    while(x){
        digit.pb(x%2);
        x/=2;
    }//本题低位到高位存更方便，方便移位运算
    return dfs(digit.size()-1,0,0,0,1,1,1,1,1,1);
}

void solve(){
    LL n;
    std::cin>>n>>a1>>a2>>a3;
    std::cout<<getSum(n)<<"\n";
}
```

选择低位到高位还是高位到低位应该根据题目的不同来选择。

另外如果满足limit为真的情况出现的概率远低于为假的，那么可以把limit这一维省略掉，dp数组里面只记录limit为假的情况。

低位到高位存还有一个好处。我们知道高位到低位存时，由于数字长度不一样，比如a有5位，b有10位，把a的dp数组求完后，求b时就必须再次清空成-1。原因是，对于a的pos为1，也就是从高到低第二位，其实对应这b的pos为6，我们就不能用a中求出的dp带进去b里面计算了。

而低位到高位就没有这个问题，大家的最低位都是平等的从0下标开始的，可以进行复用。清零为-1只需要程序最开始的时候清零一次即可。有时候程序有多个case会避免TLE。

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
//快速幂
//luogu P1226
using LL = long long;

LL qPow(LL x, LL p){
    //x^p
    LL res = 1;
    while(p){
        if(p&1){
            res = res * x;
        }
        x *= x;
        p>>=1;
    }
    return res;
}

LL qPowMod(LL x, LL p, LL m){
    //x^p % m
    LL res = 1;
    while(p){
        if(p&1){
            res = (res * x)%m;
        }
        x = (x*x)%m;
        p>>=1;
    }
    return res;
}
```

## 离散化

有两种，一种是unique函数版，一种是树状数组求逆序对里使用的，都可以，区别是，那个对于相同的数字根据先后顺序确定大小，这个则是一样大

```cpp
//复杂度nlogn
//离散化 例如将1,500,40,1000保持相对大小不变，离散化为1,3,2,4
//luogu B3694
#include <iostream>
#include <vector>
#include <algorithm>

std::vector<int> arr,assi;

void solve(){
    int n;
    std::cin>>n;
    arr.clear();
    assi.clear();
    for(int i=1;i<=n;i++){
        int a;
        std::cin>>a;
        arr.push_back(a);
        assi.push_back(a);
    }
    std::sort(assi.begin(),assi.end());
    assi.erase(std::unique(assi.begin(),assi.end()),assi.end());

    for(int i=0;i<n;i++){
        arr[i] = std::upper_bound(assi.begin(),assi.end(),arr[i])-assi.begin();
    }

    for(int i=0;i<n;i++){
        std::cout<<arr[i]<<" ";
    }
    std::cout<<"\n";
}
```

## 莫队算法

```cpp
//对于序列上的区域离线询问问题，如果[l,r]的答案能够O(1)拓展得到
//[l-1,r],[l+1,r],[l,r-1],[l,r+1]的答案，那么就可以在O(n sqrt(n))中解决所有询问
//SPOJ DQUERY
//本题是给定若干个区间[l,r]，查询这个范围内有多少个不同的数
int arr[MAXN];
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

int ans[MAXQ], cnt[MAXA], cur;
int l=1,r=0;//初始化询问区间

inline void add(int p){
    if(cnt[arr[p]]==0)//新增一种数
        cur++;
    cnt[arr[p]]++;
}

inline void del(int p){
    cnt[arr[p]]--;
    if(cnt[arr[p]]==0)//把一种数全部删完
        cur--;
}

int main(){
    std::ios::sync_with_stdio(false);
    std::cin.tie(0);

	int n;
	std::cin>>n;
	sq = std::sqrt(n);
	for(int i=1;i<=n;i++) std::cin>>arr[i];
	int q;
	std::cin>>q;
	for(int i=0;i<q;i++){
	    std::cin>>Q[i].l>>Q[i].r;
	    Q[i].id = i;//把询问离线
	}
	std::sort(Q,Q+q);
	
	for(int i=0;i<q;i++){
	    while(l>Q[i].l)
	        add(--l);//当前区间l大于查询的l，要把左边的数加进来
	    while(r<Q[i].r)
	        add(++r);//当前区间r小于查询的r，要把右边的数加进来
	    while(l<Q[i].l)
	        del(l++);//当前区间l小于查询的l，要把左边的数删掉
	    while(r>Q[i].r)
	        del(r--);//当前区间r大于查询的r，要把右边的数删掉
	    //注意上述顺序，扩张区间是先移动再更新，缩减区间是先更新再移动
	    //四个操作的具体实现可能会有不一样，具体题目具体讨论
	    ans[Q[i].id] = cur;
	}
	
	for(int i=0;i<q;i++){
	    std::cout<<ans[i]<<"\n";
	}

    return 0;
}
```

## 0/1分数规划

0/1分数规划的目的是如下列式子取值最大化

$$
\dfrac{\sum^n_{i=1}a_ix_i}{\sum^n_{i=1}b_ix_i}
$$

其中$\{a_i\}$和$\{b_i\}$是给定的数列，而$\{x_i\}$是要求的一组解，其取值只能是$0$或$1$。或者说，给定$n$对数$a_i,b_i$，从中选出若干对（通常题目要求恰好选$k$对），使选出的数对的$a$之和和$b$之和的商最大。

我们转化成二分答案，验证一个值$m$，上式的取值能否大于等于$m$。转化一下就可以得到，是否存在一组解$x_1,\cdots,x_n$，满足

$$
\sum^n_{i=1}(a_i-m\times b_i)\times x_i\geq 0
$$

如果存在则说明$m$比最大值要小，否则$m$比最大值要大，满足二分性。

我们可以计算每一个$(a_i-m\times b_i)$，如果题目说可以任意选择若干对，则只要有一个非负数，就能满足条件。如果要求恰好选$k$对，那么我们全部算出来然后排序，选择最大的$k$个，其和非负就能满足条件。

```cpp
//0/1分数规划，复杂度n log^2 n
//nowcoder NC14662
//介绍见markdown
LL a[MAXN],b[MAXN];
int n,k;

bool judge(DB mid){
    std::vector<DB> vec;
    for(int i=1;i<=n;i++){
        vec.pb(a[i]-mid*b[i]);
    }
    std::sort(vec.begin(),vec.end());
    DB sum = 0;
    for(int i=n-1;i>=0 && n-1-i+1<=k;i--) sum+=vec[i];
    if(sum>=0) return true;
    return false;
}

void solve(){
    std::cin>>n>>k;
    for(int i=1;i<=n;i++){
        std::cin>>b[i]>>a[i];
    }
    
    DB l=0, r=1e13;
    for(int i=1;i<=100;i++){
        DB mid = (l+r)/2;
        if(judge(mid)) l = mid;
        else r = mid;
    }
    std::cout<<(LL)r<<"\n";
}
```

## 表达式求值 可能需要进一步完善TODO

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

## 艾弗森括号

艾弗森括号常表示为$[A]$，其中$A$是一个二值表达式。用三元运算符可以等价为$[A]\leftrightarrow A?1:0$。

例如$[\gcd(a,b)==1]$，就意味着，如果$a,b$互质，式子的值为$1$，否则为$0$。

## 向上、向下取整

在C++中，如果我们对一个double或者float用强制类型转换

```
double x = 1.1;
std::cout<<(int)x<<"\n";
```

这其实看起来像是向下取整，实际上却不是。它是省略小数部分，实为向$0$取整。C++的整数除法也是向零取整的，而Python则是向下取整。具体表现为C++中$-7/2$为$-3$，Python中$-7//2$为$-4$。

C++本身也有std::floor()和std::ceil()，但是只能针对浮点数使用。

我们一般会在结果非负的时候，用以下方式计算向上向下取整

```cpp
int x = a/b;//a/b向下取整
int y = (a+b-1)/b;//a/b向上取整
```

而对于负数结果，我们有一个性质

$$
-\lfloor x\rfloor= \lceil -x\rceil
$$

$$
-\lceil x\rceil= \lfloor -x\rfloor
$$

转换即可。

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

同样，两个参数类型相同。自定义比较方法同sort。如果要比较三四个元素，可以使用initializer_list，例如std::max({1,2,3,4,5});

## std::max_element,std::min_element

返回一个范围内的最大（最小元素）的迭代器（指针）。复杂度，准确比较max(N-1,0)次

```cpp
std::vector<int> v{3,1,-14,9};
std::cout<<*std::max_element(v.begin(),v.end())<<"\n";
```

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

## std::copy

将一个范围的值复制给另一个范围，复杂度：准确赋值 (last - first) 次

```cpp
int a[10],b[10];
std::copy(a,a+10,b);//源起点，源终点，目的地起点
```

需要注意的点是，不要数组越界，包括源不要越界，目标的大小也要足够复制。类型要一致。以及，目标起点的地址不能在源之内，否则行为是未定义的。

## std::fill

将给定值填写到整个范围中，复杂度：准确赋值 std::distance(first, last) 次。

```cpp
std::vector<int> v{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
std::fill(v.begin(), v.end(), -1);
```

之后v变成全-1的vector。

可以对C数组使用指针，和memset不一样的是，memset是字节赋值，fill并不会把4字节的int每一个字节都赋值一次。

注意，如果你对结构体数组使用，你不能直接像memset一样全部memset为0。你要创建一个你认为的初始化应该有的值，再fill进去。

```cpp
struct A{
    int a,b;
}a[100];

std::memset(a,0,sizeof(a));
//std::fill(a,a+100,0);//出错
A tmp = {0,0};
std::fill(a,a+100,tmp);
```

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

这里的bucket是指哈希桶，也就是用拉链法实现的hash表。

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

### ::resize

重新指定vector的大小，会改变size()，resize有两个参数(new_size, value)，第一个为要分配的大小，第二个为指定的初始值（可以忽略，此时调用默认构造函数）。注意只有加大size时，新增的元素会被赋值。

### ::reserve

给vector预留空间，但不会改变size()的大小。参数只有一个，为预留的元素个数。

如果dfs中参数引用的vector会导致RE，可以试试预留足够的大小。

### std::vector\<bool\>

这是一个特化的vector，它每一个元素所占的空间是一位，而不是sizeof(bool)（通常是一字节）。

不建议使用，除非非常了解会发生什么。

operator[]返回的不是bool类型，返回的是一个proxy reference。

```cpp
std::vector<bool> c{false,true,true};
bool a = c[0];//经过了强制类型转换
auto b = c[0];//没有转换，本身b就是引用了
a = true;//c是0 1 1
b = true;//c是1 1 1
```

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

## scanf TODO

### 输入十六进制、八进制、二进制

## printf TODO

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
