## c++
```c++
struct UnionFind{
    vector<int> par; // par[i]:iの親の番号
    vector<int> s; // s[i]:iの属する木のノード数
    int n_group; // 木の数
    VI val;
    
    UnionFind(int N) : par(N) {
        for(int i=0; i<N; i++) par[i] = i;
        s.resize(N, 1);
        n_group = N;
        val.resize(N,0);
    }
    
    int root(int x){ // xが属する木の根
        if(par[x] == x) return x;
        int p = par[x];
        par[x] = root(par[x]);
        if(p != par[x]) val[x] += val[p];
        return par[x];
    }
    
    void unite(int x, int y){ // xとyの木を併合
        int rx = root(x);
        int ry = root(y);
        if(rx == ry) return;
        if(s[rx] > s[ry]) swap(rx, ry);
        par[rx] = ry;
        val[rx] -= val[ry];
        s[ry] += s[rx];
        s[rx] = 0;
        n_group--;
    }
    
    bool same(int x, int y){ // xとyが同じ木に属するかどうか
        return root(x) == root(y);
    }

    int size(int x){ // xが属する木のノード数
        return s[root(x)];
    }

    void add(int pos, int x){
        int r = root(pos);
        val[r] += x;
    }

    int query(int pos){
        int r = root(pos);
        return r == pos ? val[pos] : val[r] + val[pos];
    }
};
```
