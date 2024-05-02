## C++
```c++
struct LowestCommonAncestor {
    struct Edge {
        int from, to, id;
        Edge(int from, int to, int id = -1) : from(from), to(to), id(id) {}
        Edge rev() {
            return Edge(to, from, id);
        }
    };

    LowestCommonAncestor() : LowestCommonAncestor(0) {}
    LowestCommonAncestor(int N) : N(N) {
        edges.resize(N);
        depth.resize(N, 0);
        LOGN = std::bit_width(uint32_t(N));
        tab.resize(N, std::vector<int>(LOGN + 1, -1));
    }

    void add_edge(int from, int to, int id = -1) {
        edges[from].emplace_back(from, to, id);
    }

    void build(int r = 0) {
        std::queue<std::pair<int, int>> q;
        q.emplace(r, r);
        while (!q.empty()) {
            auto &[v, par] = q.front();
            q.pop();
            for (Edge &e : edges[v]) {
                if (e.to == par)
                    continue;
                depth[e.to] = depth[v] + 1;
                q.emplace(e.to, v);
            }
        }

        for (int i = 0; i < N; i++) {
            for (Edge &e : edges[i]) {
                if (depth[e.from] + 1 == depth[e.to])
                    continue;
                tab[e.from][0] = e.to;
            }
        }

        for (int k = 0; k < LOGN; k++)
            for (int i = 0; i < N; i++) {
                if (tab[i][k] == -1)
                    continue;
                tab[i][k + 1] = tab[tab[i][k]][k];
            }
    }

    int lca(int u, int v) {
        if (depth[u] < depth[v])
            std::swap(u, v);
        int k = depth[u] - depth[v];
        for (int i = 0; i <= LOGN; i++) {
            if (((k >> i) & 1) == 0)
                continue;
            u = tab[u][i];
        }
        if (u == v)
            return u;

        for (int i = LOGN; i >= 0; i--) {
            if (tab[u][i] != tab[v][i]) {
                u = tab[u][i];
                v = tab[v][i];
            }
        }
        return tab[u][0];
    }

  private:
    int N, LOGN = 1;
    std::vector<vector<Edge>> edges;
    std::vector<int> depth;
    std::vector<std::vector<int>> tab;
};

struct AuxiliaryTree {
    AuxiliaryTree(std::vector<int> attributes) : attributes(attributes) {
        N = attributes.size();
        M = *std::max_element(attributes.begin(), attributes.end()) + 1;

        lca = LowestCommonAncestor(N);
        edges.resize(N);
    }

    void add_edge(int from, int to) {
        lca.add_edge(from, to);
        edges[from].emplace_back(from, to);
    }

    void build(int r = 0) {
        preorder.resize(N, -1);
        subtree_vertices.resize(M);

        lca.build();
        dfs(r);
        build_trees();
    }

    int solve_tree(int tree_id) {
        return trees[tree_id].solve();
    }

  private:
    struct Edge {
        int from, to, id;
        Edge(int from, int to, int id = -1) : from(from), to(to), id(id) {}
        Edge rev() {
            return Edge(to, from, id);
        }
    };

    struct Tree {
        Tree() {}
        Tree(std::vector<int> attributes) : attributes(attributes) {
            N = attributes.size();
            edges.resize(N);
        }

        void add_edge(int from, int to, int id = -1) {
            edges[from].emplace_back(from, to, id);
        }

        int solve(int v = 0, int par = -1) {
            if (N == 0)
                return 0;
            return 0;
        }

      private:
        int N;
        std::vector<std::vector<Edge>> edges;
        std::vector<int> attributes;
    };

    int N, M;
    std::vector<int> attributes, preorder;
    std::vector<std::vector<int>> subtree_vertices;
    std::vector<std::vector<Edge>> edges;
    std::vector<Tree> trees;
    LowestCommonAncestor lca;

    void dfs(int r) {
        int order = 0;
        std::stack<std::pair<int, int>> stk;
        stk.emplace(r, 0);

        while (!stk.empty()) {
            auto [v, pos] = stk.top();
            stk.pop();

            if (pos == 0) {
                preorder[v] = order++;
                subtree_vertices[attributes[v]].push_back(v);
            }

            for (int i = pos; i < edges[v].size(); i++) {
                Edge &e = edges[v][i];
                if (preorder[e.to] > -1)
                    continue;
                stk.emplace(v, i + 1);
                stk.emplace(e.to, 0);
                break;
            }
        }
    }

    void build_trees() {
        trees.resize(M);
        std::vector<int> to_subtree_id(N, 0);

        for (int i = 0; i < M; i++) {
            int n = subtree_vertices[i].size();

            for (int j = 0; j < n - 1; j++) {
                int r = lca.lca(subtree_vertices[i][j], subtree_vertices[i][j + 1]);
                if (attributes[r] != i)
                    subtree_vertices[i].push_back(r);
            }

            std::sort(subtree_vertices[i].begin(),
                      subtree_vertices[i].end(),
                      [&](int i, int j) { return preorder[i] < preorder[j]; });
            subtree_vertices[i].erase(
                std::unique(subtree_vertices[i].begin(), subtree_vertices[i].end()),
                subtree_vertices[i].end());

            n = subtree_vertices[i].size();
            for (int j = 0; j < n; j++)
                to_subtree_id[subtree_vertices[i][j]] = j;

            std::vector<int> sub_attributes(n, 0);
            for (int j = 0; j < n; j++)
                sub_attributes[j] = attributes[subtree_vertices[i][j]] == i;

            Tree tree(sub_attributes);
            for (int j = 0; j < n - 1; j++) {
                int r = lca.lca(subtree_vertices[i][j], subtree_vertices[i][j + 1]);
                int from = to_subtree_id[r], to = to_subtree_id[subtree_vertices[i][j + 1]];

                tree.add_edge(from, to);
                tree.add_edge(to, from);
            }
            trees[i] = tree;
        }
    }
};
```

## Example

- [ABC340 G - Leaf Color (C++)](https://atcoder.jp/contests/abc340/submissions/52977073)
