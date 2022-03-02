### DSU on tree

```c++
int n,q;
vector<int> g[N];

// siz: 子树大小
// big: 重儿子
// col: 结点颜色
// ans: 存答案
// cnt[i]: 颜色为 i 的结点个数
// totColor: 目前出现过的颜色个数

int siz[N],big[N],col[N];
int ans[N],cnt[N],totColor;

//轻重链剖分
void dfs0(int u,int fa){
    siz[u]=1;
    for(int v:g[u]){
        if(v==fa) continue;
        dfs0(v,u);
        siz[u]+=siz[v];
        if(!big[u]||siz[big[u]]<siz[v]) big[u]=v;
    }
}

void update(int u,int fa,int sign,int bigson){
    int c=col[u];
    cnt[c]+=sign;
    if(sign==1&&cnt[c]==1) totColor++;
    if(sign==-1&&!cnt[c]) totColor--;

    for(int v:g[u]){
        if(v!=fa&&v!=bigson) update(v,u,sign,bigson);
    }
}

/*
1.遍历每一个节点
2.递归解决所有的轻儿子，同时消除递归产生的影响
3.递归重儿子，不消除递归的影响
4.统计所有轻儿子对答案的影响
5.更新该节点的答案
6.删除所有轻儿子对答案的影响
*/
void dfs1(int u,int fa,bool keep){
    for(int v:g[u]){
        if(v!=fa&&v!=big[u]) dfs1(v,u,0);
    }
    if(big[u]) dfs1(big[u],u,1);
    
    update(u,fa,1,big[u]);
    ans[u]=totColor;
    
    if(!keep){
        update(u,fa,-1,0);
        totColor=0;
    }
}

void solve(){
	cin>>n;
    rep(i,1,n-1){
        int u,v;cin>>u>>v;
        g[u].pb(v);g[v].pb(u);
    }
    rep(i,1,n) cin>>col[i];
    dfs0(1,0);
    dfs1(1,0,0);
    cin>>q;
    while(q--){
        int x;cin>>x;
        cout<<ans[x]<<"\n";
    }
}
```

##### dfn版
```c++
int n,q;
vector<int> g[N];

// siz: 子树大小
// big: 重儿子
// col: 结点颜色
// L[u]: 结点 u 的 DFS 序
// R[u]: 结点 u 子树中结点的 DFS 序的最大值
// Node[i]: DFS 序为 i 的结点
// ans: 存答案
// cnt[i]: 颜色为 i 的结点个数
// totColor: 目前出现过的颜色个数

int siz[N],big[N],col[N],L[N],R[N],Node[N],totdfn;
int ans[N],cnt[N],totColor;

//先预处理出每个节点子树的大小和它的重儿子
void dfs0(int u,int fa){
    L[u]=++totdfn;
    Node[totdfn]=u;
    siz[u]=1;
    for(int v:g[u]){
        if(v==fa) continue;
        dfs0(v,u);
        siz[u]+=siz[v];
        if(!big[u]||siz[big[u]]<siz[v]) big[u]=v;
    }
    R[u]=totdfn;
}

void add(int u) {
    if(cnt[col[u]]==0) totColor++;
    cnt[col[u]]++;
}
void del(int u) {
    cnt[col[u]]--;
    if (cnt[col[u]]==0) totColor--;
}


// 先遍历 u 的轻（非重）儿子，并计算答案，但不保留遍历后它对 cnt 数组的影响
// 遍历它的重儿子，保留它对 cnt 数组的影响
// 再次遍历 u 的轻儿子的子树结点，加入这些结点的贡献，以得到 u 的答案
void dfs1(int u,int fa,bool keep){
    //计算轻儿子的答案
    for(int v:g[u]){
        if(v!=fa&&v!=big[u]) dfs1(v,u,0);
    }
    //计算重儿子答案并保留计算过程中的数据（用于继承）
    if(big[u]) dfs1(big[u],u,1);
    
    for(int v:g[u]){
        if(v!=fa&&v!=big[u]){
            //子树结点的DFS序构成一段连续区间，可以直接遍历
            for(int i=L[v];i<=R[v];i++){
                add(Node[i]);
            }
        }
    }
    
    add(u);
    ans[u]=totColor;
    if(!keep){
        for(int i=L[u];i<=R[u];i++){
            del(Node[i]);
        }
    }
}

void solve(){
	cin>>n>>q;
    rep(i,1,n) cin>>col[i];
    rep(i,1,n-1){
        int u,v;cin>>u>>v;
        g[u].pb(v);g[v].pb(u);
    }
    dfs0(1,0);
    dfs1(1,0,0);
    while(q--){
        int x;cin>>x;
        cout<<ans[x]<<"\n";
    }
}
```