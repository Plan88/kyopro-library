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

    std::vector<T> bellman_ford(int s = 0, T inf = 1 << 30) {
        // inf := コストの初期値
        std::vector<T> dist(N, inf);
        dist[s] = 0;

        // 高々 N-1 回のループで全頂点の最短距離が求まる
        for (int i = 0; i < N - 1; i++) {
            for (int j = 0; j < N; j++) {
                for (Edge &e : edges[j]) {
                    dist[e.to] = std::min(dist[e.to], dist[e.from] + e.cost);
                }
            }
        }

        return dist;
    }

    bool exists_negative_cycle(int s = 0, T inf = 1 << 30) {
        // 負閉路存在判定
        // inf := コストの初期値
        std::vector<T> dist = bellman_ford(s, inf);

        // N 回目のループで最短距離の更新があれば負閉路が存在
        for (int i = 0; i < N; i++) {
            for (Edge &e : edges[i]) {
                if (dist[e.from] + e.cost < dist[e.to])
                    return true;
            }
        }

        return false;
    }

    bool exists_reachable_negative_cycle(int s = 0, T inf = 1 << 30) {
        // 頂点 s から到達可能な負閉路の存在判定
        // inf := コストの初期値
        std::vector<T> dist(N, inf);
        dist[s] = 0;
        // 到達可能フラグ
        std::vector<int> reachable(N, 0);
        reachable[s] = 1;

        // 高々 N-1 回のループで全頂点の最短距離が求まる
        for (int i = 0; i < N - 1; i++) {
            for (int j = 0; i < N; j++) {
                for (Edge &e : edges[j]) {
                    if (dist[e.from] + e.cost < dist[e.to]) {
                        dist[e.to] = dist[e.from] + e.cost;
                        if (reachable[e.from] == 1) {
                            reachable[e.to] = 1;
                        }
                    }
                }
            }
        }

        // N 回目のループで到達可能な頂点の最短距離の更新があれば負閉路が存在
        for (int i = 0; i < N; i++) {
            for (Edge &e : edges[i]) {
                if (reachable[e.from] == 1 and dist[e.from] + e.cost < dist[e.to])
                    return true;
            }
        }

        return false;
    }

  private:
    int N;
    std::vector<std::vector<Edge>> edges;
};
```

## Rust
```rust

    use num::{Bounded, Zero};
    use num_traits::SaturatingAdd;

    #[derive(Clone, Copy, Debug)]
    struct Edge<T> {
        from: usize,
        to: usize,
        cost: T,
    }

    pub struct Graph<T> {
        n: usize,
        edges: Vec<Vec<Edge<T>>>,
    }

    impl<T: Ord + Copy + SaturatingAdd + Bounded + Zero> Graph<T> {
        pub fn new(n: usize) -> Self {
            Self {
                n,
                edges: vec![vec![]; n],
            }
        }

        /// from -> to のコスト cost の有向辺を追加する。
        pub fn add_edge(&mut self, from: usize, to: usize, cost: T) {
            self.edges[from].push(Edge { from, to, cost });
        }

        /// ベルマンフォード法で頂点 s を始点とする各頂点への最短経路を求める。
        pub fn bellman_ford(&self, s: usize) -> Vec<T> {
            let mut dist = vec![T::max_value(); self.n];
            dist[s] = T::zero();

            for _ in 0..self.n - 1 {
                for i in 0..self.n {
                    for edge in &self.edges[i] {
                        dist[edge.to] =
                            dist[edge.to].min(dist[edge.from].saturating_add(&edge.cost));
                    }
                }
            }
            dist
        }

        /// 負閉路が存在するかどうかを判定する。
        pub fn exists_negative_cycle(&self) -> bool {
            let dist = self.bellman_ford(0);

            for i in 0..self.n {
                for edge in &self.edges[i] {
                    if dist[edge.from].saturating_add(&edge.cost) < dist[edge.to] {
                        return true;
                    }
                }
            }
            false
        }

        /// 頂点 s から到達可能な負閉路が存在するかどうかを判定する。
        pub fn exists_reachable_negative_cycle(&self, s: usize) -> bool {
            let dist = self.bellman_ford(s);
            let reachable = (0..self.n)
                .map(|i| dist[i] < T::max_value())
                .collect::<Vec<_>>();

            for i in 0..self.n {
                for edge in &self.edges[i] {
                    if reachable[edge.from]
                        && dist[edge.from].saturating_add(&edge.cost) < dist[edge.to]
                    {
                        return true;
                    }
                }
            }
            false
        }
    }
}
```

## Example

- [ARC173 D - Bracket Walk (C++)](https://atcoder.jp/contests/arc173/submissions/52252759)
- [ABC404 G - Specified Range Sums (Rust)](https://atcoder.jp/contests/abc404/submissions/67182837)
