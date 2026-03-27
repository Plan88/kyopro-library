## Rust
```rust
mod tree {
    use std::collections::VecDeque;

    type Cost = u64;

    struct Edge {
        from: usize,
        to: usize,
        cost: Cost,
    }

    #[derive(Clone, Copy)]
    struct SubTreeInfo {
        n_child: usize,
        depth: usize,
        par: Option<usize>,
    }

    pub struct Tree {
        n: usize,
        edges: Vec<Vec<Edge>>,
        subtree: Vec<SubTreeInfo>,
    }

    impl Tree {
        pub fn new(n: usize) -> Self {
            let edges = (0..n).map(|_| vec![]).collect();
            let subtree = vec![
                SubTreeInfo {
                    n_child: 0,
                    depth: 0,
                    par: None,
                };
                n
            ];

            Self { n, edges, subtree }
        }

        pub fn add_edge(&mut self, from: usize, to: usize, cost: Cost) {
            self.add_directed_edge(from, to, cost);
            self.add_directed_edge(to, from, cost);
        }

        fn add_directed_edge(&mut self, from: usize, to: usize, cost: Cost) {
            let edge = Edge { from, to, cost };
            self.edges[from].push(edge);
        }

        pub fn build(&mut self) {
            self.dfs(0, 0, 0);
        }

        fn dfs(&mut self, v: usize, par: usize, depth: usize) -> SubTreeInfo {
            let mut subtree = SubTreeInfo {
                n_child: 1,
                depth,
                par: Some(par),
            };
            let m = self.edges[v].len();
            for i in 0..m {
                let edge = &self.edges[v][i];
                if edge.to == par {
                    continue;
                }
                let info = self.dfs(edge.to, v, depth + 1);
                subtree.n_child += info.n_child;
            }

            self.subtree[v] = subtree;
            subtree
        }

        /// 木の重心を求める
        pub fn get_centroid(&self) -> Vec<usize> {
            let mut c = vec![];
            for i in 0..self.n {
                let mut flag = true;
                if self.n - self.subtree[i].n_child > self.n / 2 {
                    flag = false;
                }
                for edge in self.edges[i].iter() {
                    if let Some(par) = self.subtree[i].par
                        && par == edge.to
                    {
                        continue;
                    }
                    if self.subtree[edge.to].n_child > self.n / 2 {
                        flag = false;
                        break;
                    }
                }
                if flag {
                    c.push(i);
                }
            }
            c
        }

        /// 木の直径を求める
        /// 木の直径とその両端の頂点を返す
        pub fn calc_diameter(&self) -> (Cost, usize, usize) {
            let r = self
                .subtree
                .iter()
                .enumerate()
                .fold((0, 0), |cur, (i, &tree)| {
                    if cur.1 < tree.depth {
                        (i, tree.depth)
                    } else {
                        cur
                    }
                })
                .0;

            let mut dist = vec![Cost::MAX; self.n];
            let mut q = VecDeque::new();
            q.push_back(r);
            dist[r] = 0;

            while let Some(v) = q.pop_front() {
                for edge in self.edges[v].iter() {
                    let cost = dist[v] + edge.cost;
                    if cost < dist[edge.to] {
                        dist[edge.to] = cost;
                        q.push_back(edge.to);
                    }
                }
            }

            let v = dist
                .iter()
                .enumerate()
                .fold(
                    (0, 0),
                    |cur, (i, &cost)| {
                        if cur.1 < cost { (i, cost) } else { cur }
                    },
                )
                .0;
            (dist[v], r, v)
        }
    }
}
```
