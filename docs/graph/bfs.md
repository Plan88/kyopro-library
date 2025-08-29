## Rust
```rust
mod graph {
    use std::{
        cmp::Reverse,
        collections::{BinaryHeap, VecDeque},
    };

    use num::Zero;

    #[derive(Clone, Copy)]
    struct Edge<T> {
        from: usize,
        to: usize,
        cost: T,
    }

    pub struct Graph<T: Copy> {
        n: usize,
        edges: Vec<Vec<Edge<T>>>,
    }

    impl<T: Copy> Graph<T> {
        pub fn new(n: usize) -> Self {
            let edges = vec![vec![]; n];
            Self { n, edges }
        }

        pub fn add_directed_edge(&mut self, from: usize, to: usize, cost: T) {
            self.edges[from].push(Edge { from, to, cost })
        }

        pub fn add_edge(&mut self, from: usize, to: usize, cost: T) {
            self.add_directed_edge(from, to, cost);
            self.add_directed_edge(to, from, cost);
        }

        /// 頂点 s から各頂点への最短距離を bfs で計算する
        /// 全ての辺の長さは 1 と仮定する
        pub fn bfs(&self, s: usize) -> Vec<Option<u32>> {
            let mut dist = vec![None; self.n];
            let mut q = VecDeque::with_capacity(self.n);

            dist[s] = Some(0);
            q.push_back(s);
            while let Some(v) = q.pop_front() {
                for edge in &self.edges[v] {
                    if dist[edge.to].is_some_and(|c| c <= dist[edge.from].unwrap() + 1) {
                        continue;
                    }
                    dist[edge.to] = Some(dist[edge.from].unwrap() + 1);
                    q.push_back(edge.to);
                }
            }

            dist
        }
    }

    impl<T: Ord + Copy + Zero> Graph<T> {
        /// 頂点 s から各頂点への最短距離をダイクストラ法で計算する
        pub fn dijkstra(&self, s: usize) -> Vec<Option<T>> {
            let mut dist = vec![None; self.n];
            let mut q = BinaryHeap::<Reverse<(T, usize)>>::new();

            dist[s] = Some(T::zero());
            q.push(Reverse((T::zero(), s)));
            while let Some(Reverse((cost, v))) = q.pop() {
                if dist[v].is_some_and(|c| c < cost) {
                    continue;
                }

                for edge in &self.edges[v] {
                    let new_cost = cost + edge.cost;
                    if dist[edge.to].is_some_and(|c| c <= new_cost) {
                        continue;
                    }
                    dist[edge.to] = Some(new_cost);
                    q.push(Reverse((new_cost, edge.to)));
                }
            }

            dist
        }
    }
}
```
