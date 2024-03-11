## c++
```c++
template<typename T>
struct BIT{
    private:
        using Func = function<T(T, T)>;

        int n;
        vector<T> node;

        T init_v;
        Func func;

    public:

        BIT(int N, Func _func, T _init_v) {
            n = N + 1;
            init_v = _init_v;
            func = _func;

            node.resize(n, init_v);
            for(int i=1; i<n; i++) if((i+(i&-i)) < n) node[i+(i&-i)] = func(node[i+(i&-i)], node[i]);
        }

        BIT() {}

        void add(int pos, T v) {
            // 0-indexed
            pos++;
            while(pos < n){
                node[pos] = func(node[pos], v);
                pos += pos & -pos;
            }
        }

        T sum(int pos){
            // 0-indexed
            // [0,pos]の総和
            pos++;
            T res = init_v;
            while(pos > 0){
                res = func(res, node[pos]);
                pos = pos & (pos-1);
            }
            return res;
        }

    	T sum(int l, int r){
            // 0-indexed
            // [l,r)の総和
            return sum(r-1) - sum(l-1);
        }
};
```
