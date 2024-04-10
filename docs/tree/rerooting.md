## C++
```c++
template<typename T>
struct ReRooting{
    typedef function<T(T,T)> F1;
    typedef function<T(T,int)> F2;

    struct Node{
        int to, cost;
        Node(int t, int c): to(t), cost(c) {}
    };

    int N;
    F1 merge;
    F2 addNode;
    T e;
    vector<vector<Node>> adj;

    vector<vector<int>> i4adj;
    vector<vector<T>> dp; // dp[i][j] := iを親, j番目の隣接ノードを根とする部分木の値
    vector<T> ret;

    ReRooting(int n, F1 f, F2 g, T identity): N(n), merge(f), addNode(g), e(identity){
        i4adj.resize(n);
        adj.resize(n);
    }

    void add_edge(int i, int j, int cost=1){
        i4adj[i].push_back((int)adj[j].size());
        i4adj[j].push_back((int)adj[i].size());
        adj[i].emplace_back(Node(j, cost)); // iから見てk番目のノードはj
        adj[j].emplace_back(Node(i, cost)); // jから見てl番目のノードはi
    }

    void initialize(){
        dp.resize(N);
        for(int i=0; i<N; i++)
            dp[i].resize(adj[i].size());
        ret.resize(N);

        if(N == 1){
            ret[0] = addNode(e, 0);
            return;
        }

        vector<int> par(N), order(N); // par[i] := 0を根としたときの親 order[i] := dfsでのi番目の訪問ノード
        par[0] = -1;
        int index = 0;
        stack<int> stk; stk.push(0);
        while(!stk.empty()){
            int node = stk.top(); stk.pop();
            order[index++] = node;

            for(Node& e : adj[node]){
                if(e.to == par[node]) continue;
                stk.push(e.to);
                par[e.to] = node;
            }
        }

        // 0を根とする木について解を求める
        for(int i=N-1; i>=1; i--){
            int node = order[i];
            int parent = par[node];

            T result = e;
            int parIndex = -1;
            for(int j=0; j<adj[node].size(); j++){
                Node& e = adj[node][j];
                if(e.to == parent){
                    parIndex = j;
                    continue;
                }
                result = merge(result, dp[node][j]);
            }
            dp[parent][i4adj[node][parIndex]] = addNode(result, node);
        }

        // 全ての頂点について解を求める
        for(int i=0; i<N; i++){
            int node = order[i];
            T accum = e;
            vector<T> accumsFromTail(adj[node].size());
            accumsFromTail[accumsFromTail.size() - 1] = e;

            for (int j=accumsFromTail.size()-1; j >= 1; j--)
                    accumsFromTail[j - 1] = merge(dp[node][j], accumsFromTail[j]);

            for (int j = 0; j < accumsFromTail.size(); j++) {
                dp[adj[node][j].to][i4adj[node][j]] = addNode(merge(accum, accumsFromTail[j]), node);
                accum = merge(accum, dp[node][j]);
            }
            ret[node] = addNode(accum, node);
        }
    }

    T query(int node){
        return ret[node];
    }

};
```
