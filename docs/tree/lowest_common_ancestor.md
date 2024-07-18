## C++
```c++
template<class T=int>
struct LCA {
    struct Edge {
        int from, to;
        T cost;
        int id;
        Edge(int from, int to, T cost=1, int id=-1):from(from), to(to), cost(cost), id(id) {}
        Edge rev(){
            return Edge(to, from, cost, id);
        }
    };

    int N, LOGN = 1;
    std::vector<vector<Edge>> edges;
    std::vector<int> depth;
    std::vector<std::vector<int>> tab;

    LCA(): LCA(0) {}
    LCA(int N): N(N) {
        edges.resize(N);
        depth.resize(N,0);
        while((1<<LOGN) < N) LOGN++;
        tab.resize(N,std::vector<int>(LOGN+1,-1));
    }

    void add_edge(int from, int to, T cost=1, int id=-1){
        edges[from].emplace_back(from, to, cost, id);
    }

    void build(int r=0){
        std::queue<std::pair<int,int>> q; q.emplace(r, r);
        while(!q.empty()){
            auto& [v, par] = q.front(); q.pop();
            for(Edge& e : edges[v]){
                if(e.to == par) continue;
                depth[e.to] = depth[v] + 1;
                q.emplace(e.to, v);
            }
        }

        for(int i=0; i<N; i++){
            for(Edge& e : edges[i]){
                if(depth[e.from]+1 == depth[e.to]) continue;
                tab[e.from][0] = e.to;
            }
        }

        for(int k=0; k<LOGN; k++)
            for(int i=0; i<N; i++){
                if(tab[i][k] == -1) continue;
                tab[i][k+1] = tab[tab[i][k]][k];
            }
    }

    std::vector<std::vector<T>> build_tab2(const function<T(T,T)>& merge, T id){
        std::vector<std::vector<T>> tab2(N,std::vector<T>(LOGN+1,id));
        for(int i=0; i<N; i++)
            for(Edge& e : edges[i]){
                if(depth[e.from]+1 == depth[e.to]) continue;
                tab2[e.from][0] = e.cost;
            }

        for(int k=0; k<LOGN; k++)
            for(int i=0; i<N; i++){
                if(tab[i][k] == -1) continue;
                tab2[i][k+1] = merge(tab2[i][k], tab2[tab[i][k]][k]);
            }

        return tab2;
    }

    int lca(int u, int v){
        if(depth[u] < depth[v]) swap(u,v);
        int k = depth[u] - depth[v];
        for(int i=0; i<=LOGN; i++){
            if(((k>>i)&1) == 0) continue;
            u = tab[u][i];
        }
        if(u == v) return u;

        for(int i=LOGN; i>=0; i--){
            if(tab[u][i] != tab[v][i]){
                u = tab[u][i];
                v = tab[v][i];
            }
        }
        return tab[u][0];
    }

    T lca_tab2(int u, int v, const function<T(T,T)>& merge, T id, const std::vector<std::vector<T>>& tab2){
		// aggregation cost on path u-v
        T ret = id;
        if(depth[u] < depth[v]) swap(u,v);
        int k = depth[u] - depth[v];
        for(int i=0; i<=LOGN; i++){
            if(((k>>i)&1) == 0) continue;
            ret = merge(ret, tab2[u][i]);
            u = tab[u][i];
        }
        if(u == v) return ret;

        for(int i=LOGN; i>=0; i--){
            if(tab[u][i] != tab[v][i]){
                ret = merge(ret, tab2[u][i]);
                ret = merge(ret, tab2[v][i]);
                u = tab[u][i];
                v = tab[v][i];
            }
        }
        ret = merge(ret, tab2[u][0]);
        ret = merge(ret, tab2[v][0]);
        return ret;
    }
};
```

## Rust
```rust
mod tree {
    use std::mem::swap;

    use num::Num;

    #[derive(Clone, Copy)]
    struct Edge<T: Ord + Copy + Num> {
        from: usize,
        to: usize,
        cost: T,
    }

    pub struct LowestCommonAncestor<T: Ord + Copy + Num> {
        n: usize,
        logn: usize,
        edges: Vec<Vec<Edge<T>>>,
        depth: Vec<usize>,
        tab: Vec<Vec<Option<usize>>>,
        order: Vec<usize>,
    }

    #[allow(dead_code)]
    impl<T: Ord + Copy + Num> LowestCommonAncestor<T> {
        pub fn new(n: usize) -> Self {
            let edges = vec![vec![]; n];
            let depth = vec![0; n];
            let logn = bit_width(n) - 1;
            let tab = vec![vec![None; logn + 1]; n];
            let order = vec![0; n];
            Self {
                n,
                logn,
                edges,
                depth,
                tab,
                order,
            }
        }

        pub fn add_edge(&mut self, from: usize, to: usize, cost: T) {
            self.add_directed_edge(from, to, cost);
            self.add_directed_edge(to, from, cost);
        }

        fn add_directed_edge(&mut self, from: usize, to: usize, cost: T) {
            let edge = Edge { from, to, cost };
            self.edges[from].push(edge);
        }

        pub fn build(&mut self) {
            self.build_depth();
            self.build_tab();
        }

        fn build_depth(&mut self) {
            self.dfs(0, 0, 0, 0);
        }

        fn dfs(&mut self, v: usize, par: usize, depth: usize, mut order: usize) -> usize {
            self.depth[v] = depth;
            self.order[v] = order;

            let m = self.edges[v].len();
            for i in 0..m {
                let edge = self.edges[v][i];
                if edge.to == par {
                    continue;
                }
                order = self.dfs(edge.to, v, depth + 1, order + 1);
            }

            order
        }

        fn build_tab(&mut self) {
            for i in 0..self.n {
                for edge in self.edges[i].iter() {
                    if self.depth[edge.from] < self.depth[edge.to] {
                        continue;
                    }
                    self.tab[edge.from][0] = Some(edge.to);
                }
            }

            for k in 0..self.logn {
                for i in 0..self.n {
                    if let Some(to) = self.tab[i][k] {
                        self.tab[i][k + 1] = self.tab[to][k];
                    }
                }
            }
        }

        pub fn get_order(&self, v: usize) -> usize {
            self.order[v]
        }

        pub fn lca(&self, mut u: usize, mut v: usize) -> usize {
            if self.depth[u] < self.depth[v] {
                swap(&mut u, &mut v);
            }

            let diff = self.depth[u] - self.depth[v];
            for i in 0..=self.logn {
                if ((diff >> i) & 1) == 1 {
                    u = self.tab[u][i].unwrap();
                }
            }

            if u == v {
                return u;
            }

            for i in (0..=self.logn).rev() {
                if self.tab[u][i].is_none() || self.tab[v][i].is_none() {
                    continue;
                }
                if self.tab[u][i] != self.tab[v][i] {
                    u = self.tab[u][i].unwrap();
                    v = self.tab[v][i].unwrap();
                }
            }

            self.tab[u][0].unwrap()
        }

        pub fn build_cost_tab<F: Fn(T, T) -> T>(&self, merge: F, e: T) -> Vec<Vec<T>> {
            let mut tab = vec![vec![e; self.logn + 1]; self.n];

            for i in 0..self.n {
                for edge in self.edges[i].iter() {
                    if self.depth[edge.from] < self.depth[edge.to] {
                        continue;
                    }
                    tab[edge.from][0] = edge.cost;
                }
            }

            for k in 0..self.logn {
                for i in 0..self.n {
                    if let Some(to) = self.tab[i][k] {
                        tab[i][k + 1] = merge(tab[i][k], tab[to][k]);
                    }
                }
            }

            tab
        }

        pub fn lca_cost_tab<F: Fn(T, T) -> T>(
            &self,
            mut u: usize,
            mut v: usize,
            merge: F,
            e: T,
            tab: &Vec<Vec<T>>,
        ) -> T {
            let mut ret = e;

            if self.depth[u] < self.depth[v] {
                swap(&mut u, &mut v);
            }

            let diff = self.depth[u] - self.depth[v];
            for i in 0..=self.logn {
                if ((diff >> i) & 1) == 1 {
                    ret = merge(ret, tab[u][i]);
                    u = self.tab[u][i].unwrap();
                }
            }

            if u == v {
                return ret;
            }

            for i in (0..=self.logn).rev() {
                if self.tab[u][i].is_none() || self.tab[v][i].is_none() {
                    continue;
                }
                if self.tab[u][i] != self.tab[v][i] {
                    ret = merge(ret, tab[u][i]);
                    ret = merge(ret, tab[v][i]);
                    u = self.tab[u][i].unwrap();
                    v = self.tab[v][i].unwrap();
                }
            }

            ret = merge(ret, tab[u][0]);
            ret = merge(ret, tab[v][0]);

            ret
        }
    }

    fn bit_width(x: usize) -> usize {
        x.ilog2() as usize + 1
    }
}
```

## Example
- [035 - Preserve Connectivity（★7） (Rust)](https://atcoder.jp/contests/typical90/submissions/55488507)
