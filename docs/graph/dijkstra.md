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
