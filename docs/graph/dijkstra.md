## C++
```c++
template <class T = int>
struct Graph {
    struct Edge {
        int from, to, id = -1;
        T cost = 1;

        Edge(int from, int to, T cost = 1, int id = -1) : from(from), to(to), cost(cost), id(id) {}
    };

    Graph(int N) : N(N) {
        edges.resize(N);
    }

    void add_directed_edge(int from, int to, T cost = 1, int id = -1) {
        Edge e(from, to, cost, id);
        edges[from].push_back(e);
    }

    void add_edge(int u, int v, T cost = 1, int id = -1) {
        add_directed_edge(u, v, cost, id);
        add_directed_edge(v, u, cost, id);
    }

    std::vector<T> dijkstra(int s = 0) {
        std::vector<T> dist(N, -1);
        std::priority_queue<std::pair<T, int>, std::vector<std::pair<T, int>>, std::greater<std::pair<T, int>>> q;

        auto push = [&](int v, T cost) {
            if (dist[v] < 0 or cost < dist[v]) {
                dist[v] = cost;
                q.emplace(cost, v);
            }
        };

        push(s, 0);
        while (!q.empty()) {
            auto [cost, v] = q.top();
            q.pop();

            if (dist[v] < cost)
                continue;

            for (Edge &e : edges[v]) {
                push(e.to, cost + e.cost);
            }
        }

        return dist;
    }

  private:
    int N;
    std::vector<std::vector<Edge>> edges;
};
```

## Rust
```rust
mod graph {
    use std::{cmp::Reverse, collections::BinaryHeap};

    use num::{Bounded, Num};

    #[derive(Clone, Copy)]
    struct Edge<T: Ord + Copy + Num + Bounded> {
        from: usize,
        to: usize,
        cost: T,
    }

    pub struct Graph<T: Ord + Copy + Num + Bounded> {
        n: usize,
        edges: Vec<Vec<Edge<T>>>,
    }

    impl<T: Ord + Copy + Num + Bounded> Graph<T> {
        pub fn new(n: usize) -> Self {
            let edges = vec![vec![]; n];
            Self { n, edges }
        }

        pub fn add_directed_edge(&mut self, from: usize, to: usize, cost: T) {
            let edge = Edge { from, to, cost };
            self.edges[from].push(edge);
        }

        pub fn add_edge(&mut self, from: usize, to: usize, cost: T) {
            self.add_directed_edge(from, to, cost);
            self.add_directed_edge(to, from, cost);
        }

        /// 頂点 s から各頂点への最短距離を計算する
        pub fn dijkstra(&self, s: usize) -> Vec<T> {
            let mut dist = vec![T::max_value(); self.n];
            let mut q = BinaryHeap::<Reverse<(T, usize)>>::new();

            dist[s] = T::zero();
            q.push(Reverse((T::zero(), s)));
            while let Some(Reverse((cost, v))) = q.pop() {
                if dist[v] < cost {
                    continue;
                }

                for edge in self.edges[v].iter() {
                    let new_cost = cost + edge.cost;
                    if dist[edge.to] > new_cost {
                        dist[edge.to] = new_cost;
                        q.push(Reverse((new_cost, edge.to)));
                    }
                }
            }

            dist
        }
    }
}
```

## Examples

- [ABC164 E - Two Currencies (C++)](https://atcoder.jp/contests/abc164/submissions/57343801)
