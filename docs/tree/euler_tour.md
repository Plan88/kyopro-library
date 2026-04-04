## C++
```c++
template<class T>
struct EulerTour {
    int N;
    std::vector<std::vector<Edge>> edges;
    std::vector<int> in, out;

    EulerTour(int N): N(N) {
        edges.resize(N);
        in.resize(N);
        out.resize(N);
    }

    void add_directed_edge(int from, int to, T cost, int id = -1) {
        Edge e(from, to, cost, id);
        edges[from].push_back(e);
    }
    void add_edges(int from, int to, T cost, int id = -1) {
        add_directed_edge(from, to, cost, id);
        add_directed_edge(to, from, cost, id);
    }

    void build(int r) {
        dfs(r, r);
    }

  private:
    struct Edge {
        int from, to, id = -1;
        T cost = 1;
        Edge(int from, int to, T cost = 1, int id = -1) : from(from), to(to), cost(cost), id(id) {}
    };

    int dfs(int v, int par, int order = 0) {
        in[v] = order;
        for(auto e : edges[v]) {
            if(e.to == par) continue;
            order = dfs(e.to, e.from, order + 1);
        }
        out[v] = ++order;
        return order;
    }
};
```

## Rust
'''rust

mod tree {
    pub struct EulerTour {
        edges: Vec<Vec<usize>>,
        tin: Vec<usize>,
        tout: Vec<usize>,
        order: Vec<usize>,
        depth: Vec<usize>,
        parent: Vec<Option<usize>>,
    }

    impl EulerTour {
        pub fn new(n: usize) -> Self {
            Self {
                edges: vec![vec![]; n],
                tin: vec![0; n],
                tout: vec![0; n],
                order: Vec::with_capacity(n),
                depth: vec![0; n],
                parent: vec![None; n],
            }
        }

        pub fn add_edge(&mut self, u: usize, v: usize) {
            self.edges[u].push(v);
            self.edges[v].push(u);
        }

        pub fn build(&mut self, root: usize) {
            let n = self.edges.len();
            self.tin = vec![0; n];
            self.tout = vec![0; n];
            self.order.clear();
            self.depth = vec![0; n];
            self.parent = vec![None; n];

            let mut time = 0;
            // 非再帰 DFS: (頂点, 隣接リストの次のindex, 行きがけ済みか)
            let mut stack: Vec<(usize, usize, bool)> = vec![(root, 0, false)];

            while let Some((v, idx, entered)) = stack.last_mut() {
                if !*entered {
                    self.tin[*v] = time;
                    self.order.push(*v);
                    time += 1;
                    *entered = true;
                }

                let v = *v;
                let idx = *idx;

                if idx < self.edges[v].len() {
                    stack.last_mut().unwrap().1 += 1;
                    let u = self.edges[v][idx];
                    if Some(u) != self.parent[v] {
                        self.parent[u] = Some(v);
                        self.depth[u] = self.depth[v] + 1;
                        stack.push((u, 0, false));
                    }
                } else {
                    self.tout[v] = time;
                    time += 1;
                    stack.pop();
                }
            }
        }

        pub fn subtree_range(&self, v: usize) -> std::ops::Range<usize> {
            self.tin[v]..self.tout[v]
        }

        /// u が v の祖先かどうか
        pub fn is_ancestor(&self, u: usize, v: usize) -> bool {
            self.tin[u] <= self.tin[v] && self.tout[v] <= self.tout[u]
        }

        pub fn tin(&self, v: usize) -> usize {
            self.tin[v]
        }

        pub fn tout(&self, v: usize) -> usize {
            self.tout[v]
        }

        pub fn order(&self, i: usize) -> usize {
            self.order[i]
        }

        pub fn depth(&self, v: usize) -> usize {
            self.depth[v]
        }

        pub fn parent(&self, v: usize) -> Option<usize> {
            self.parent[v]
        }
    }
}
```
