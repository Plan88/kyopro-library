## C++
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

# Rust
```rust
pub mod data_structure {
    #[derive(Debug)]
    pub struct HeavyLightDecomposition {
        pre_order: Vec<usize>,      // pre_order[i] = 頂点 i の訪問時刻
        parent: Vec<Option<usize>>, // parent[i] = 頂点 i の親の頂点番号
        head: Vec<usize>,           // head[i] = 頂点 i と同じ heavy path における最も根側の頂点番号
        pub num_childs: Vec<usize>, // num_childs[i] = 頂点 i を根とする部分木の頂点数
        edges: Vec<Vec<usize>>,
    }

    impl HeavyLightDecomposition {
        pub fn new(n: usize) -> Self {
            assert!(n > 0);
            Self {
                pre_order: vec![usize::MAX; n],
                parent: vec![None; n],
                head: vec![usize::MAX; n],
                num_childs: vec![1; n],
                edges: vec![vec![]; n],
            }
        }

        /// u -> v と v -> u 方向に辺を追加する
        pub fn add_edge(&mut self, u: usize, v: usize) {
            self.edges[u].push(v);
            self.edges[v].push(u);
        }

        pub fn build(&mut self) {
            self.dfs_for_num_childs(0, None);
            let mut order = 0;
            self.dfs(0, None, 0, &mut order);
        }

        /// 頂点 v の訪問時刻を返す
        pub fn index(&self, v: usize) -> usize {
            self.pre_order[v]
        }

        /// 頂点 u と頂点 v の最小共通祖先を返す
        pub fn lca(&self, mut u: usize, mut v: usize) -> usize {
            loop {
                if self.pre_order[u] > self.pre_order[v] {
                    std::mem::swap(&mut u, &mut v);
                }
                if self.head[u] == self.head[v] {
                    return u;
                }
                v = self.parent[self.head[v]].unwrap();
            }
        }

        /// 頂点 from から頂点 to へのパスをいくつかの heavy path に分解し、各 heavy path の端点を返すイテレータを返す。
        pub fn iter_heavy_path(&self, from: usize, to: usize) -> PathSegmentsIterator {
            PathSegmentsIterator {
                hld: self,
                from,
                to,
                done: false,
            }
        }

        /// num_childs を求めるための dfs
        fn dfs_for_num_childs(&mut self, v: usize, parent: Option<usize>) {
            let n = self.edges[v].len();
            for i in 0..n {
                let to = self.edges[v][i];
                if Some(to) == parent {
                    continue;
                }
                self.dfs_for_num_childs(to, Some(v));
                self.num_childs[v] += self.num_childs[to];
            }
        }

        /// pre_order, parent, head を求めるための dfs
        /// Self::dfs_for_num_childs の後に実行する
        fn dfs(&mut self, v: usize, parent: Option<usize>, head: usize, order: &mut usize) {
            self.parent[v] = parent;
            self.pre_order[v] = *order;
            *order += 1;
            self.head[v] = head;
            let next_id = {
                let mut max = 0;
                let mut max_id = None;
                for &to in &self.edges[v] {
                    if Some(to) == parent {
                        continue;
                    }
                    if max < self.num_childs[to] {
                        max = self.num_childs[to];
                        max_id = Some(to);
                    }
                }
                max_id
            };
            if let Some(to) = next_id {
                self.dfs(to, Some(v), head, order);
            }
            let n = self.edges[v].len();
            for i in 0..n {
                let to = self.edges[v][i];
                if Some(to) == next_id || Some(to) == parent {
                    continue;
                }
                self.dfs(to, Some(v), to, order);
            }
        }
    }

    pub struct PathSegmentsIterator<'a> {
        hld: &'a HeavyLightDecomposition,
        from: usize,
        to: usize,
        done: bool,
    }

    impl Iterator for PathSegmentsIterator<'_> {
        /// (根側の頂点番号, 葉側の頂点番号 (端点を含む), lca を含む path か, 元のパスの u -> v の向きと同じか)
        type Item = (usize, usize, bool, bool);
        fn next(&mut self) -> Option<Self::Item> {
            let Self {
                hld,
                from,
                to,
                done,
            } = *self;
            (!done).then_some(())?;
            let HeavyLightDecomposition {
                pre_order,
                parent,
                head,
                ..
            } = hld;

            if head[from] == head[to] {
                self.done = true;
                if pre_order[from] < pre_order[to] {
                    Some((from, to, true, false))
                } else {
                    Some((to, from, true, true))
                }
            } else if pre_order[from] < pre_order[to] {
                self.to = parent[head[to]].unwrap();
                Some((head[to], to, false, false))
            } else {
                self.from = parent[head[from]].unwrap();
                Some((head[from], from, false, true))
            }
        }
    }
}
```
