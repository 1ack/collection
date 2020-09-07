# basic knowledge
并查集：
```
int find(int x)
{
    if(x!=fa[x])
        fa[x]=find(fa[x]);
    return fa[x];
}
void union(int x,int y)
{
    int a=find(x),b=find(y);
    if(a!=b)
        fa[b]=a;
}
```

}
树状数组：
```
int a[maxn];
int lowbit(int x)
{
    return x&(-x);
}
void add(int x,int y)
{
    while(x<=n)
    {
        a[x]+=y;
        x+=lowbit(x);
    }
}
int sum(int x)
{
    int s=0;
    while(x>0)
    {
        s+=a[x];
        x-=lowbit(x);
    }
    return s;
}

```

DSU（并查集Disjoint Set Union），Heap（堆），BIT（树状数组），SegmentTree（线段树），四分树，字母树，AC自动机，后缀大家族（SA后缀数组，SAM后缀自动机，ST后缀树），二叉搜索树，平衡树大家族（AVL，红黑树，Treap，Splay，SBT，替罪羊树），kd-tree，PQ-tree,B树

Link-cut Tree

hashTree

Skip list