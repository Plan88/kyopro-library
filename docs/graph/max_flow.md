## C++
```c++
struct MaxFlow{
    int n;
    struct edge{
        int to, cap, rev;
        edge(int to, int cap, int rev):to(to), cap(cap), rev(rev) {}
    };
    vector<vector<edge>> G;
    vector<int> level, iter;

    MaxFlow(int N): n(N){
        G.resize(N);
        level.resize(N);
        iter.resize(N);
    }

    void add_edge(int from, int to, int cap){
        G[from].emplace_back(to, cap, G[to].size());
        G[to].emplace_back(from, 0, G[from].size()-1);
    }

    void bfs(int s){
        level.assign(n, -1);
        queue<int> q;
        level[s] = 0;
        q.push(s);
        while(!q.empty()){
            int v = q.front(); q.pop();
            for(int i=0; i<G[v].size(); i++){
                edge& e = G[v][i];
                if(0 < e.cap && level[e.to] < 0){
                    level[e.to] = level[v] + 1;
                    q.push(e.to);
                }
            }
        }
    }

    int dfs(int v, int t, int f){
        if(v == t) return f;
        for(int& i = iter[v]; i<G[v].size(); i++){
            edge& e = G[v][i];
            if(0 < e.cap && level[v] < level[e.to]){
                int d = dfs(e.to, t, min(f, e.cap));
                if(d > 0){
                    e.cap -= d;
                    G[e.to][e.rev].cap += d;
                    return d;
                }
            }
        }
        return 0;
    }

    int max_flow(int s, int t){
        // O(|E||V|^2)
        int flow = 0;
        for(;;){
            bfs(s);
            if(level[t] < 0) return flow;
            iter.assign(n, 0);
            int f;
            while((f = dfs(s, t, INF)) > 0){
                flow += f;
            }
        }
    }
};
```

## Rust
maybe contained some bug
```rust
#[allow(dead_code)]
mod graph {
    use std::collections::VecDeque;

    use itertools::Itertools;
    use num::{Bounded, Num};
    use num_traits::NumAssign;

    #[derive(Clone, Copy, Debug)]
    struct InnerEdge<T: Ord + Copy + Num + NumAssign + Bounded> {
        from: usize,
        to: usize,
        cap: T,
        flow: T,
        rev: usize, // 逆辺が edges[to] の何番目にあるか
        is_rev: bool,
        id: usize,
    }

    impl<T: Ord + Copy + Num + NumAssign + Bounded> InnerEdge<T> {
        fn get_residual(&self) -> T {
            self.cap - self.flow
        }
        fn to_edge(&self) -> Edge<T> {
            Edge {
                from: self.from,
                to: self.to,
                cap: self.cap,
                flow: self.cap,
            }
        }
    }

    pub struct Edge<T> {
        from: usize,
        to: usize,
        cap: T,
        flow: T,
    }

    pub struct MaxFlow<T: Ord + Copy + Num + NumAssign + Bounded> {
        n: usize,
        edges: Vec<Vec<InnerEdge<T>>>,
        iter: Vec<usize>,
        dist: Vec<u32>,
        counter: usize,
    }

    impl<T: Ord + Copy + Num + NumAssign + Bounded> MaxFlow<T> {
        pub fn new(n: usize) -> Self {
            let edges = vec![vec![]; n];
            let iter = vec![0; n];
            let dist = vec![0; n];
            let counter = 0;
            Self {
                n,
                edges,
                iter,
                dist,
                counter,
            }
        }

        pub fn add_edge(&mut self, from: usize, to: usize, cap: T) {
            let rev = self.edges[to].len();
            let edge = InnerEdge {
                from,
                to,
                cap,
                flow: T::zero(),
                rev,
                is_rev: false,
                id: self.counter,
            };
            let rev = self.edges[from].len();
            let rev_edge = InnerEdge {
                from: to,
                to: from,
                cap,
                flow: cap,
                rev,
                is_rev: true,
                id: self.counter,
            };
            self.counter += 1;

            self.edges[from].push(edge);
            self.edges[to].push(rev_edge);
        }

        pub fn flow(&mut self, source: usize, sink: usize) -> T {
            let mut max_flow = T::zero();

            loop {
                self.dist = self.bfs(source);
                if self.dist[sink] == u32::MAX {
                    break;
                }
                self.iter = vec![0; self.n];
                loop {
                    let flow = self.dfs(source, sink, T::max_value());
                    if flow == T::zero() {
                        break;
                    }
                    max_flow += flow;
                }
            }
            max_flow
        }

        pub fn get_edges(&self) -> Vec<Edge<T>> {
            let mut edges = vec![];
            for i in 0..self.n {
                for &edge in self.edges[i].iter() {
                    if !edge.is_rev {
                        edges.push(edge);
                    }
                }
            }
            edges.sort_by_key(|edge| edge.id);
            edges.into_iter().map(|edge| edge.to_edge()).collect_vec()
        }

        /// 残余グラフで source からの最短距離を bfs で求める
        fn bfs(&self, source: usize) -> Vec<u32> {
            let mut dist = vec![u32::MAX; self.n];

            let mut q = VecDeque::new();
            dist[source] = 0;
            q.push_back(source);

            while let Some(v) = q.pop_front() {
                for edge in self.edges[v].iter() {
                    if edge.get_residual() == T::zero() {
                        continue;
                    }
                    let cost = dist[v] + 1;
                    if cost < dist[edge.to] {
                        dist[edge.to] = cost;
                        q.push_back(edge.to);
                    }
                }
            }

            dist
        }

        /// 残余グラフでの source -> sink のパスを求めて流せる最大の流量を返す
        fn dfs(&mut self, source: usize, sink: usize, flow: T) -> T {
            if source == sink {
                return flow;
            }

            let m = self.edges[source].len();
            for i in self.iter[source]..m {
                let edge = self.edges[source][i];
                let cap = edge.get_residual();
                if self.dist[source] >= self.dist[edge.to]
                    || self.dist[edge.to] == u32::MAX
                    || cap == T::zero()
                {
                    continue;
                }
                let f = self.dfs(edge.to, sink, flow.min(cap));
                if f > T::zero() {
                    let rev = self.get_rev_edge(edge);
                    self.edges[edge.from][rev.rev].flow += f;
                    self.edges[rev.from][edge.rev].flow -= f;
                    return f;
                }
                self.iter[source] = i + 1;
            }
            T::zero()
        }

        /// 逆辺を取得する
        fn get_rev_edge(&self, edge: InnerEdge<T>) -> InnerEdge<T> {
            self.edges[edge.to][edge.rev]
        }
    }
}
```
