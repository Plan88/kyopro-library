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
