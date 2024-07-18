## Rust
```rust
#[allow(dead_code)]
mod tree {
    use std::collections::VecDeque;

    use num::{Bounded, Num};

    #[derive(Clone, Copy)]
    struct Edge<T: Ord + Copy + Num + Bounded> {
        from: usize,
        to: usize,
        cost: T,
    }

    struct SubTreeInfo {
        n_child: usize,
    }

    pub struct Tree<T: Ord + Copy + Num + Bounded> {
        n: usize,
        edges: Vec<Vec<Edge<T>>>,
        depth: Vec<usize>,
        n_child: Vec<usize>,
        par: Vec<Option<usize>>,
    }

    impl<T: Ord + Copy + Num + Bounded> Tree<T> {
        pub fn new(n: usize) -> Self {
            let edges = vec![vec![]; n];
            let depth = vec![0; n];
            let n_child = vec![1; n];
            let par = vec![None; n];

            Self {
                n,
                edges,
                depth,
                n_child,
                par,
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
            self.dfs(0, 0, 0);
        }

        fn dfs(&mut self, v: usize, par: usize, depth: usize) -> SubTreeInfo {
            self.depth[v] = depth;

            let m = self.edges[v].len();
            for i in 0..m {
                let edge = self.edges[v][i];
                if edge.to == par {
                    continue;
                }
                let info = self.dfs(edge.to, v, depth + 1);
                self.n_child[v] += info.n_child;
                self.par[edge.to] = Some(v);
            }

            SubTreeInfo {
                n_child: self.n_child[v],
            }
        }

        pub fn get_centroid(&self) -> Vec<usize> {
            let mut c = vec![];
            for i in 0..self.n {
                let mut flag = true;
                if self.n - self.n_child[i] > self.n / 2 {
                    flag = false;
                }
                for edge in self.edges[i].iter() {
                    if self.par[i].is_some() && self.par[i].unwrap() == edge.to {
                        continue;
                    }
                    if self.n_child[edge.to] > self.n / 2 {
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

        pub fn calc_diameter(&self) -> T {
            let r = self
                .depth
                .iter()
                .enumerate()
                .fold((0, 0), |cur, (i, &d)| if cur.1 > d { (i, d) } else { cur })
                .0;

            let mut dist = vec![T::max_value(); self.n];
            let mut q = VecDeque::new();
            q.push_back((r, T::zero()));
            dist[r] = T::zero();

            while let Some((v, cost)) = q.pop_front() {
                for edge in self.edges[v].iter() {
                    let cost = cost + edge.cost;
                    if cost < dist[edge.to] {
                        dist[edge.to] = cost;
                        q.push_back((edge.to, cost));
                    }
                }
            }

            dist.into_iter().max().unwrap()
        }
    }
}
```
