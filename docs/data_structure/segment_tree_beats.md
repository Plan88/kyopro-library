## c++
```c++
struct SegmentTreeBeats{
    const long long LINF = 1LL<<60;
    int n;
    vector<long long> sum;
    vector<long long> f_max, s_max, n_max;
    vector<long long> f_min, s_min, n_min;
    vector<long long> len, ladd, lval;

    SegmentTreeBeats(vector<long long> a){
        int sz = a.size();
        n = 1;
        while(n < sz) n <<= 1;

        sum.resize(2*n-1, 0);

        f_max.resize(2*n-1, -LINF);
        s_max.resize(2*n-1, -LINF);
        n_max.resize(2*n-1, 0);

        f_min.resize(2*n-1, LINF);
        s_min.resize(2*n-1, LINF);
        n_min.resize(2*n-1, 0);

        ladd.resize(2*n-1, 0);
        lval.resize(2*n-1, LINF);
        len.resize(2*n-1);

        len[0] = n;
        for(int i=0; i<n-1; ++i) len[2*i+1] = len[2*i+2] = (len[i]>>1);

        for(int i=0; i<sz; ++i){
            f_max[i+n-1] = f_min[i+n-1] = sum[i+n-1] = a[i];
            n_max[i+n-1] = n_min[i+n-1] = 1;
        }

        for(int i=n-2; i>=0; --i) _update(i);
    }

    void _update_node_max(int k, long long x){
        sum[k] += (x-f_max[k]) * n_max[k];

        if(f_max[k] == f_min[k]) f_max[k] = f_min[k] = x;
        else if(f_max[k] == s_min[k]) f_max[k] = s_min[k] = x;
        else f_max[k] = x;

        if(lval[k] != LINF && x < lval[k]) lval[k] = x;

        return;
    }

    void _update_node_min(int k, long long x) {
        sum[k] += (x-f_min[k]) * n_min[k];

        if(f_max[k] == f_min[k]) f_max[k] = f_min[k] = x;
        else if(s_max[k] == f_min[k]) s_max[k] = f_min[k] = x;
        else f_min[k] = x;

        if(lval[k] != LINF && lval[k] < x) lval[k] = x;

        return;
    }

    void _update_node_add(int k, long long x){
        f_max[k] += x;
        if(s_max[k] != -LINF) s_max[k] += x;

        f_min[k] += x;
        if(s_min[k] != LINF) s_min[k] += x;

        sum[k] += len[k] * x;
        if(lval[k] != LINF) lval[k] += x;
        else ladd[k] += x;

        return;
    }

    void _update_node_assign(int k, long long x){
        f_max[k] = x; s_max[k] = -LINF;
        f_min[k] = x; s_min[k] = LINF;
        n_max[k] = n_min[k] = len[k];

        sum[k] = x * len[k];
        lval[k] = x; ladd[k] = 0;

        return;
    }

    void _pushdown(int k){
        if(n-1 <= k) return;

        if(lval[k] != LINF){
            _update_node_assign(2*k+1, lval[k]);
            _update_node_assign(2*k+2, lval[k]);
            lval[k] = LINF;
            return;
        }

        if(ladd[k] != 0){
            _update_node_add(2*k+1, ladd[k]);
            _update_node_add(2*k+2, ladd[k]);
            ladd[k] = 0;
        }

        if(f_max[k] < f_max[2*k+1]) _update_node_max(2*k+1, f_max[k]);
        if(f_max[k] < f_max[2*k+2]) _update_node_max(2*k+2, f_max[k]);

        if(f_min[2*k+1] < f_min[k]) _update_node_min(2*k+1, f_min[k]);
        if(f_min[2*k+2] < f_min[k]) _update_node_min(2*k+2, f_min[k]);

        return;
    }

    void _update(int k){
        if(n-1 <= k) return;

        sum[k] = sum[2*k+1] + sum[2*k+2];

        if(f_max[2*k+1] < f_max[2*k+2]){
            f_max[k] = f_max[2*k+2];
            n_max[k] = n_max[2*k+2];
            s_max[k] = max(f_max[2*k+1], s_max[2*k+2]);
        }
        else if(f_max[2*k+1] > f_max[2*k+2]){
            f_max[k] = f_max[2*k+1];
            n_max[k] = n_max[2*k+1];
            s_max[k] = max(s_max[2*k+1], f_max[2*k+2]);
        }
        else{
            f_max[k] = f_max[2*k+1];
            n_max[k] = n_max[2*k+1] + n_max[2*k+2];
            s_max[k] = max(s_max[2*k+1], s_max[2*k+2]);
        }

        if(f_min[2*k+1] < f_min[2*k+2]){
            f_min[k] = f_min[2*k+1];
            n_min[k] = n_min[2*k+1];
            s_min[k] = min(s_min[2*k+1], f_min[2*k+2]);
        }
        else if(f_min[2*k+1] > f_min[2*k+2]){
            f_min[k] = f_min[2*k+2];
            n_min[k] = n_min[2*k+2];
            s_min[k] = min(f_min[2*k+1], s_min[2*k+2]);
        }
        else{
            f_min[k] = f_min[2*k+1];
            n_min[k] = n_min[2*k+1] + n_min[2*k+2];
            s_min[k] = min(s_min[2*k+1], s_min[2*k+2]);
        }

        return;
    }

    void update_min(int a, int b, long long x, int k=0, int l=0, int r=-1){
        if(r<0) r = n;

        // break condition
        if(b<=l || r<=a || f_max[k]<=x) return;
        // tag condition
        if(a<=l && r<=b && s_max[k]<x){
            _update_node_max(k, x);
            return;
        }

        _pushdown(k);
        update_min(a, b, x, 2*k+1, l, (l+r)/2);
        update_min(a, b, x, 2*k+2, (l+r)/2, r);
        _update(k);

        return;
    }

    void update_max(int a, int b, long long x, int k=0, int l=0, int r=-1) {
        if(r<0) r = n;

        if(b<=l || r<=a || x<=f_min[k]) return;
        if(a<=l && r<=b && x<s_min[k]){
            _update_node_min(k, x);
            return;
        }

        _pushdown(k);
        update_max(a, b, x, 2*k+1, l, (l+r)/2);
        update_max(a, b, x, 2*k+2, (l+r)/2, r);
        _update(k);
    }

    void update_add(int a, int b, long long x, int k=0, int l=0, int r=-1){
        if(r<0) r = n;

        if(b<=l || r<=a) return;
        if(a<=l && r<=b){
            _update_node_add(k, x);
            return;
        }

        _pushdown(k);
        update_add(a, b, x, 2*k+1, l, (l+r)/2);
        update_add(a, b, x, 2*k+2, (l+r)/2, r);
        _update(k);
    }

    void update_assign(int a, int b, long long x, int k=0, int l=0, int r=-1){
        if(r<0) r = n;

        if(b<=l || r<=a) return;
        if(a<=l && r<=b){
            _update_node_assign(k, x);
            return;
        }

        _pushdown(k);
        update_assign(a, b, x, 2*k+1, l, (l+r)/2);
        update_assign(a, b, x, 2*k+2, (l+r)/2, r);
        _update(k);
    }

    long long query_max(int a, int b, int k=0, int l=0, int r=-1){
        if(r<0) r = n;

        if(b<=l || r<=a) return 0;
        if(a<=l && r<=b) return f_max[k];

        _pushdown(k);
        long long lv = query_max(a, b, 2*k+1, l, (l+r)/2);
        long long rv = query_max(a, b, 2*k+2, (l+r)/2, r);
        return max(lv, rv);
    }

    long long query_min(int a, int b, int k=0, int l=0, int r=-1){
        if(r<0) r = n;

        if(b<=l || r<=a) return LINF;
        if(a<=l && r<=b) return f_min[k];

        _pushdown(k);
        long long lv = query_min(a, b, 2*k+1, l, (l+r)/2);
        long long rv = query_min(a, b, 2*k+2, (l+r)/2, r);
        return min(lv, rv);
    }

    long long query_sum(int a, int b, int k=0, int l=0, int r=-1){
        if(r<0) r = n;

        if(b<=l || r<=a) return 0;
        if(a<=l && r<=b) return sum[k];

        _pushdown(k);
        long long lv = query_sum(a, b, 2*k+1, l, (l+r)/2);
        long long rv = query_sum(a, b, 2*k+2, (l+r)/2, r);
        return lv+rv;
    }
};
```
