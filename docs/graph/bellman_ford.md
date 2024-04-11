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

## Example

- [ARC173 D - Bracket Walk (C++)](https://atcoder.jp/contests/arc173/submissions/52252759)
