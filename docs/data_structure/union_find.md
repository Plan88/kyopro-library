## C++
```c++
struct UnionFind{
    vector<int> par; // par[i]:iの親の番号
    vector<int> s; // s[i]:iの属する木のノード数
    int n_group; // 木の数
    
    UnionFind(int N) : par(N) {
        for(int i=0; i<N; i++) par[i] = i;
        s.assign(N, 1);
        n_group = N;
    }
    
    int root(int x){ // xが属する木の根
        if(par[x] == x) return x;
        return par[x] = root(par[x]);
    }
    
    void unite(int x, int y){ // xとyの木を併合
        int rx = root(x);
        int ry = root(y);
        if(rx == ry) return;
        par[rx] = ry;
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
};
```

## Rust
```rust
pub struct UnionFind {
    par: Vec<usize>,
}

impl UnionFind {
    pub fn new(n: usize) -> Self {
        let par = (0..n).collect::<Vec<usize>>();
        Self { par }
    }

    pub fn unite(&mut self, x: usize, y: usize) {
        let rx = self.root(x);
        let ry = self.root(y);

        self.par[rx] = ry;
    }

    pub fn root(&mut self, x: usize) -> usize {
        let r = self.par[x];

        if r == x {
            return r;
        }

        self.par[x] = self.root(r);

        return self.par[x];
    }

    pub fn is_same(&mut self, x: usize, y: usize) -> bool {
        self.root(x) == self.root(y)
    }
}
```
