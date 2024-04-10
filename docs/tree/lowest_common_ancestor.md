## C++
```c++
template<class T=int>
struct LCA {
    struct Edge {
        int from, to;
        T cost;
        int id;
        Edge(int from, int to, T cost=1, int id=-1):from(from), to(to), cost(cost), id(id) {}
        Edge rev(){
            return Edge(to, from, cost, id);
        }
    };

    int N, LOGN = 1;
    std::vector<vector<Edge>> edges;
    std::vector<int> depth;
    std::vector<std::vector<int>> tab;

    LCA(): LCA(0) {}
    LCA(int N): N(N) {
        edges.resize(N);
        depth.resize(N,0);
        while((1<<LOGN) < N) LOGN++;
        tab.resize(N,std::vector<int>(LOGN+1,-1));
    }

    void add_edge(int from, int to, T cost=1, int id=-1){
        edges[from].emplace_back(from, to, cost, id);
    }

    void build(int r=0){
        std::queue<std::pair<int,int>> q; q.emplace(r, r);
        while(!q.empty()){
            auto& [v, par] = q.front(); q.pop();
            for(Edge& e : edges[v]){
                if(e.to == par) continue;
                depth[e.to] = depth[v] + 1;
                q.emplace(e.to, v);
            }
        }

        for(int i=0; i<N; i++){
            for(Edge& e : edges[i]){
                if(depth[e.from]+1 == depth[e.to]) continue;
                tab[e.from][0] = e.to;
            }
        }

        for(int k=0; k<LOGN; k++)
            for(int i=0; i<N; i++){
                if(tab[i][k] == -1) continue;
                tab[i][k+1] = tab[tab[i][k]][k];
            }
    }

    std::vector<std::vector<T>> build_tab2(const function<T(T,T)>& merge, T id){
        std::vector<std::vector<T>> tab2(N,std::vector<T>(LOGN+1,id));
        for(int i=0; i<N; i++)
            for(Edge& e : edges[i]){
                if(depth[e.from]+1 == depth[e.to]) continue;
                tab2[e.from][0] = e.cost;
            }

        for(int k=0; k<LOGN; k++)
            for(int i=0; i<N; i++){
                if(tab[i][k] == -1) continue;
                tab2[i][k+1] = merge(tab2[i][k], tab2[tab[i][k]][k]);
            }

        return tab2;
    }

    int lca(int u, int v){
        if(depth[u] < depth[v]) swap(u,v);
        int k = depth[u] - depth[v];
        for(int i=0; i<=LOGN; i++){
            if(((k>>i)&1) == 0) continue;
            u = tab[u][i];
        }
        if(u == v) return u;

        for(int i=LOGN; i>=0; i--){
            if(tab[u][i] != tab[v][i]){
                u = tab[u][i];
                v = tab[v][i];
            }
        }
        return tab[u][0];
    }

    T lca_tab2(int u, int v, const function<T(T,T)>& merge, T id, const std::vector<std::vector<T>>& tab2){
		// aggregation cost on path u-v
        T ret = id;
        if(depth[u] < depth[v]) swap(u,v);
        int k = depth[u] - depth[v];
        for(int i=0; i<=LOGN; i++){
            if(((k>>i)&1) == 0) continue;
            ret = merge(ret, tab2[u][i]);
            u = tab[u][i];
        }
        if(u == v) return ret;

        for(int i=LOGN; i>=0; i--){
            if(tab[u][i] != tab[v][i]){
                ret = merge(ret, tab2[u][i]);
                ret = merge(ret, tab2[v][i]);
                u = tab[u][i];
                v = tab[v][i];
            }
        }
        ret = merge(ret, tab2[u][0]);
        ret = merge(ret, tab2[v][0]);
        return ret;
    }
};
```
