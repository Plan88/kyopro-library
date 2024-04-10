## C++
```c++
struct LowLink {
    struct Edge {
        int from, to, id = -1;
        Edge(int from, int to, int id = -1) : from(from), to(to), id(id) {}
    };

    LowLink(int N) : N(N) {
        edges.resize(N);
        ord.resize(N, N);
        low.resize(N, N);
    }

    void add_directed_edge(int u, int v, int id = -1) {
        // assume 0-indexed
        edges[u].emplace_back(u, v, id);
        while (int(back_edge.size()) <= id)
            back_edge.push_back(0);
    }

    void add_edge(int u, int v, int id = -1) {
        // assume 0-indexed
        add_directed_edge(u, v, id);
        add_directed_edge(v, u, id);
    }

    void build(int r = 0) {
        low[r] = dfs(r);
    }

    bool is_bridge(int u, int v) {
        // assume existing edge (u, v)
        if (ord[u] > ord[v])
            std::swap(u, v);
        return low[v] > ord[u];
    }

    bool is_articulation_point(int v) {
        if (ord[v] == 0) {
            return int(edges[v].size()) > 1;
        }
        for (Edge &edge : edges[v]) {
            int to = edge.to;
            if (ord[to] < ord[v])
                continue;
            if (is_back_edge(edge))
                continue;
            if (low[to] >= ord[v])
                return true;
        }
        return false;
    }

  private:
    int N, cnt = 0;
    std::vector<std::vector<Edge>> edges;
    std::vector<int> ord, low, back_edge;

    int dfs(int v = 0, int par = -1) {
        if (low[v] < N)
            return low[v];

        ord[v] = cnt++;
        low[v] = ord[v];

        for (Edge &edge : edges[v]) {
            int to = edge.to;
            if (to == par)
                continue;
            if (ord[v] > ord[to]) {
                low[v] = std::min(low[v], ord[to]);
                back_edge[edge.id] = 1;
                continue;
            }
            low[v] = std::min(low[v], dfs(to, v));
        }

        return low[v];
    }

    bool is_back_edge(Edge &edge) {
        return back_edge[edge.id] == 1;
    }
};
```

## Example

- [ABC334 G - Christmas Color Grid 2 (C++)](https://atcoder.jp/contests/abc334/submissions/52009538)
