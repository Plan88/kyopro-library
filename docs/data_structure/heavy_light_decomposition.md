## c++
```c++
struct HeavyLightDecomposition {
    int n,pos;
    vector<vector<int>> edge;
    vector<int> vid, head, sub, hvy, par, dep, inv, type;

    HeavyLightDecomposition(){}
    HeavyLightDecomposition(int sz):
        n(sz),pos(0),edge(n),
        vid(n,-1),head(n),sub(n,1),hvy(n,-1),
        par(n),dep(n),inv(n),type(n) {}

    void add_edge(int u, int v){
        edge[u].push_back(v);
        edge[v].push_back(u);
    }

    void build(int r=0){
        int c = 0;
        dfs(r);
        bfs(r, c++);
    }

    void dfs(int rt){
        stack<pair<int,int>> stk;
        par[rt] = -1; dep[rt] = 0;
        stk.emplace(rt,0);
        while(!stk.empty()){
            int v = stk.top().first;
            int &i = stk.top().second;
            if(i < (int)edge[v].size()){
                int u = edge[v][i++];
                if(u == par[v]) continue;
                par[u] = v;
                dep[u] = dep[v]+1;
                stk.emplace(u,0);
            }
            else{
                stk.pop();
                int res = 0;
                for(int u : edge[v]){
                    if(u == par[v]) continue;
                    sub[v] += sub[u];
                    if(res<sub[u]) res = sub[u], hvy[v] = u;
                }
            }
        }
    }

    void bfs(int r, int c){
        int &k = pos;
        queue<int> que({r});
        while(!que.empty()){
            int h=que.front();que.pop();
            for(int i=h;i!=-1;i=hvy[i]){
                type[i] = c;
                vid[i] = k++;
                inv[vid[i]] = i;
                head[i] = h;
                for(int j : edge[i]){
                    if(j==par[i] || j==hvy[i]) continue;
                    que.push(j);
                }
            }
        }
    }

    // for_each(vertex)
    // [l,r] <- attention!!
    void for_each(int u, int v, const function<void(int, int)>& f){
        while(true){
            if(vid[u] > vid[v]) swap(u,v);
            f(max(vid[head[v]],vid[u]), vid[v]);
            if(head[u] != head[v]) v = par[head[v]];
            else break;
        }
    }

    // for_each(edge)
    // [l,r] <- attention!!
    void for_each_edge(int u, int v, const function<void(int, int)>& f){
        while(true){
            if(vid[u] > vid[v]) swap(u,v);
            if(head[u] != head[v]){
                f(vid[head[v]], vid[v]);
                v = par[head[v]];
            }
            else{
                if(u!=v) f(vid[u]+1, vid[v]);
                break;
            }
        }
    }

    int lca(int u, int v){
        while(true){
            if(vid[u] > vid[v]) swap(u,v);
            if(head[u] == head[v]) return u;
            v = par[head[v]];
        }
    }

    int distance(int u, int v){
        return dep[u] + dep[v] - 2*dep[lca(u,v)];
    }
};
```
