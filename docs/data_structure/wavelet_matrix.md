## c++
```c++
struct BitArray{
    uint64_t n;
    static const uint64_t b = 64;
    static const uint64_t l = 512;
    static const uint64_t s = 64;
    static const uint64_t MASK = (1LLU<<63)-1 + (1LLU<<63);

    vector<uint64_t> B;
    vector<uint64_t> L, S;

    BitArray() {}
    BitArray(vector<bool>& a){
        uint64_t sz = a.size();
        uint64_t szB,szL,szS;
        szB = (sz + b-1) / b;
        szL = (sz + l-1) / l;
        szS = l/s*szL;

        B.assign(szB,0);
        L.assign(szL,0);
        S.assign(szS,0);

        for(uint64_t i=0; i<sz; ++i){
            if(!a[i]) continue;
            uint64_t bpos = i/b, offset = b - i%b - 1;
            B[bpos] |= 1LLU<<offset;
        }

        for(uint64_t i=0; i<(szL-1)*l; ++i){
            uint64_t lpos = i/l+1;
            L[lpos] += a[i];
        }

        uint64_t num = 0;
        for(uint64_t i=0; i<szS*s; ++i){
            if(i%s == 0){
                uint64_t spos = i/s, lpos = i/l;
                S[spos] = num - L[lpos];
            }
            if(i<sz) num += a[i];
        }
    }

    uint64_t pop_count(uint64_t x) {
        x = (x & 0x5555555555555555ULL) + ((x >> 1) & 0x5555555555555555ULL);
        x = (x & 0x3333333333333333ULL) + ((x >> 2) & 0x3333333333333333ULL);
        x = (x + (x >> 4)) & 0x0f0f0f0f0f0f0f0fULL;
        x = x + (x >>  8);
        x = x + (x >> 16);
        x = x + (x >> 32);
        return x & 0x7FLLU;
    }

    uint64_t access(int pos){
        int bpos = pos/b, bit = b-1-pos%b;
        uint64_t ret = (B[bpos]>>bit) & 1;
        return ret;
    }

    uint64_t rank(int pos, int bit){
        int bpos = pos/b, lpos = pos/l, spos = pos/s;
        int nbit = b-pos%b -1;
        uint64_t mask = MASK - ((1LLU<<nbit)-1 + (1LLU<<nbit));
        uint64_t n = L[lpos] + S[spos] + pop_count(B[bpos] & mask);
        uint64_t ret = bit ? n : pos-n;
        return ret;
    }

    uint64_t select(int n, int bit){

        // L内で二分探索
        int ok=0, ng=L.size();
        while(ng-ok>1){
            int pos = (ok+ng)/2;
            int k = bit ? L[pos] : l*pos - L[pos];
            if(k < n) ok = pos;
            else ng = pos;
        }
        int lpos = ok;

        // S内で二分探索
        ok=l*lpos/s, ng=ok+l/s;
        while(ng-ok>1){
            int pos = (ok+ng)/2;
            int k = bit ? L[lpos]+S[pos] : l*lpos+s*pos - (L[lpos]+S[pos]);
            if(k < n) ok = pos;
            else ng = pos;
        }
        int spos = ok;

        // B内で二分探索
        int bpos = s*spos/b;
        int tmp = bit ? L[lpos]+S[spos] : l*lpos+s*spos - (L[lpos]+S[spos]);
        ok=0, ng=b;
        while(ng-ok>1){
            int nbit = (ok+ng)/2;
            uint64_t mask = MASK - (1LLU<<nbit) + 1;
            int k = tmp + (bit ? pop_count(B[bpos] & mask) : b-pop_count(B[bpos] & mask)-nbit);
            if(k < n) ng = nbit;
            else ok = nbit;
        }
        int nbit = b-ok;

        uint64_t ret = l*lpos + s*spos + nbit;
        return ret;
    }
};

struct WaveletMatlix{
    static const uint64_t MASK = (1LLU<<63)-1 + (1LLU<<63);
    int N, LOGA;
    vector<BitArray> ba;
    vector<int> nzero;
    map<int,int> begin;

    WaveletMatlix(vector<int> A){
        N = A.size();
        uint64_t mx = 0; for(int i=0; i<N; ++i) mx = max(mx, (uint64_t)A[i]);

        LOGA = floor(log2(mx));
        ba.resize(LOGA+1);
        nzero.resize(LOGA+1,0);

        for(int i=LOGA; i>=0; --i){
            vector<bool> a(N);
            for(int j=0; j<N; ++j) a[j] = (A[j]>>i)&1;
            ba[i] = BitArray(a);

            vector<int> o,z;
            for(int j=0; j<N; j++){
                if((A[j]>>i) & 1) o.push_back(A[j]);
                else z.push_back(A[j]);
            }

            nzero[i] = z.size();
            z.insert(z.end(), o.begin(), o.end());
            swap(A,z);
        }

        for(int i=0; i<N; ++i){
            if(begin.find(A[i]) != begin.end()) continue;
            begin[A[i]] = i;
        }
    }

    uint64_t access(int pos){
        uint64_t ret = 0;
        for(int i=LOGA; i>=0; --i){
            int b = ba[i].access(pos);
            if(b){
                pos = nzero[i] + ba[i].rank(pos,1);
            }
            else{
                pos = ba[i].rank(pos,0);
            }
            ret = (ret<<1) + b;
        }
        return ret;
    }

    uint64_t rank(int pos, int x){
        // pos: 0-indexed
        // [0,pos)にxがいくつあるか
        if(begin.find(x) == begin.end()) return 0;

        for(int i=LOGA; i>=0; --i){
            int o,z;
            if((x>>i) & 1){
                o = ba[i].rank(pos,1); z = nzero[i];
                pos = o+z;
            }
            else{
                z = ba[i].rank(pos,0);
                pos = z;
            }
        }

        uint64_t ret = max(0, pos-begin[x]);
        return ret;
    }
    
    uint64_t rank(int l, int r, uint64_t x){
        // pos: 0-indexed
        // [l,r)にxがいくつあるか
        return rank(r,x) - rank(l,x);
    }

    int select(int i, int x){
        // i(>0)番目のxの次の位置(0-indexed)を求める
        if(begin.find(x) == begin.end()) return -1;
        if(N<=begin[x]+i-1) return -1;

        int pos = begin[x]+i;
        for(int i=0; i<=LOGA; ++i){
            if((x>>i) & 1){
                pos = ba[i].select(pos-nzero[i],1);
            }
            else{
                pos = ba[i].select(pos,0);
            }
        }

        return pos;
    }

    uint64_t quantile(int l, int r, int k){
        // [l,r)でk番目に小さい値を求める
        int ret = 0;
        for(int i=LOGA; i>=0; --i){
            int o,z;
            o = ba[i].rank(r,1) - ba[i].rank(l,1);
            z = r-l-o;
            if(k <= z){
                l = ba[i].rank(l,0);
                r = l + z;
                ret = ret<<1;
            }
            else{
                l = nzero[i] + ba[i].rank(l,1);
                r = l + o;
                k -= z;
                ret = (ret<<1) + 1;
            }
        }
        return ret;
    }

    vector<pair<uint64_t,int>> topk(int l, int r, int k){
        // [l,r)で出現頻度が大きいk個を(値,頻度)で返す
        // 頻度が同じものは値が小さいほうを優先
        auto c = [](tuple<int,int,int,uint64_t> a, tuple<int,int,int,uint64_t> b){
            int width_a = get<1>(a) - get<0>(a);
            int width_b = get<1>(b) - get<0>(b);
            if(width_a != width_b) return width_a < width_b;
            else{
                uint64_t val_a = get<3>(a), val_b = get<3>(b);
                return val_a > val_b;
            }
        };

        vector<pair<uint64_t,int>> ret;
        priority_queue<tuple<int,int,int,uint64_t>,vector<tuple<int,int,int,uint64_t>>,decltype(c)> q(c);
        q.emplace(l,r,LOGA,0);
        while(!q.empty() && (int)ret.size()<k){
            tuple<int,int,int,uint64_t> t = q.top(); q.pop();

            int depth = get<2>(t);
            uint64_t val = get<3>(t);
            if(depth < 0){
                int freq = get<1>(t) - get<0>(t);
                ret.emplace_back(val,freq);
            }
            else{
                int l = get<0>(t), r = get<1>(t), i = depth;
                int z = ba[i].rank(r,0) - ba[i].rank(l,0);

                int l1 = ba[i].rank(l,0), r1 = l1+z;
                int l2 = nzero[i] + ba[i].rank(l,1), r2 = l2+r-l-z;
                if(r1-l1>0) q.emplace(l1,r1,depth-1,val);
                if(r2-l2>0) q.emplace(l2,r2,depth-1,val | (1LLU<<i));
            }
        }

        return ret;
    }

    uint64_t sum(int l, int r){
        // [l,r)の総和を計算
        uint64_t ret = 0;
        queue<tuple<int,int,int>> q;
        q.emplace(l,r,LOGA);
        while(!q.empty()){
            tuple<int,int,int> t = q.front(); q.pop();

            int i = get<2>(t);
            if(0 <= i){
                int l = get<0>(t), r = get<1>(t);
                int z = ba[i].rank(r,0) - ba[i].rank(l,0);
                uint64_t o = r-l-z;
                ret += o * (1LLU<<i);

                int l1 = ba[i].rank(l,0), r1 = l1+z;
                int l2 = nzero[i] + ba[i].rank(l,1), r2 = l2+r-l-z;
                if(r1-l1>0) q.emplace(l1,r1,i-1);
                if(r2-l2>0) q.emplace(l2,r2,i-1);
            }
        }

        return ret;
    }

    uint64_t rangeLess(int l, int r, uint64_t x){
        // [l,r)でx未満の要素数
        int LOGX = floor(log2(max(x,(uint64_t)1)));
        if(LOGA < LOGX) return r-l;

        uint64_t ret = 0;
        for(int i=LOGA; i>=0; --i){
            int o = ba[i].rank(r,1) - ba[i].rank(l,1);
            int z = r-l-o;

            if((x>>i) & 1){
                ret += z;
                l = nzero[i] + ba[i].rank(l,1);
                r = l+o;
            }
            else{
                l = ba[i].rank(l,0);
                r = l+z;
            }
        }
        return ret;
    }

    uint64_t rangeFreq(int l, int r, uint64_t x, uint64_t y){
        // [l,r)でx<=c<yとなるcの個数
        return rangeLess(l,r,y) - rangeLess(l,r,x);
    }

    vector<pair<uint64_t,int>> rangeList(int l, int r, uint64_t x, uint64_t y){
        // [l,r)でx<=c<yとなるcを頻度と共に返す
        vector<pair<uint64_t,int>> ret;
        stack<tuple<int,int,int,uint64_t>> stk;
        stk.emplace(l,r,LOGA,0);
        while(!stk.empty()){
            tuple<int,int,int,uint64_t> t = stk.top(); stk.pop();
            int i = get<2>(t);
            uint64_t val = get<3>(t);

            if(i<0){
                int freq = get<1>(t) - get<0>(t);
                ret.emplace_back(val,freq);
            }
            else{
                uint64_t val0 = val, val1 = val | (1LLU<<i);
                uint64_t mask = MASK - (1LLU<<i) + 1;

                int l = get<0>(t), r = get<1>(t);
                if((x&mask)<=val1 && val1<y){
                    int o = ba[i].rank(r,1) - ba[i].rank(l,1);
                    int l1 = nzero[i] + ba[i].rank(l,1), r1 = l1+o;
                    if(r1-l1>0) stk.emplace(l1,r1,i-1,val1);
                }
                if((x&mask)<=val0 && val0<y){
                    int z = ba[i].rank(r,0) - ba[i].rank(l,0);
                    int l0 = ba[i].rank(l,0), r0 = l0+z;
                    if(r0-l0>0) stk.emplace(l0,r0,i-1,val0);
                }
            }
        }

        return ret;
    }

    vector<pair<uint64_t,int>> rangeMaxK(int l, int r, uint64_t x, uint64_t y, int k){
        // [l,r)でx<=c<yとなるcを大きい順にk個, 頻度と共に返す
        vector<pair<uint64_t,int>> ret;
        stack<tuple<int,int,int,uint64_t>> stk;
        stk.emplace(l,r,LOGA,0);
        while(!stk.empty() && (int)ret.size()<k){
            tuple<int,int,int,uint64_t> t = stk.top(); stk.pop();
            int i = get<2>(t);
            uint64_t val = get<3>(t);

            if(i<0){
                int freq = get<1>(t) - get<0>(t);
                ret.emplace_back(val,freq);
            }
            else{
                uint64_t val0 = val, val1 = val | (1LLU<<i);
                uint64_t mask = MASK - (1LLU<<i) + 1;

                int l = get<0>(t), r = get<1>(t);
                if((x&mask)<=val0 && val0<y){
                    int z = ba[i].rank(r,0) - ba[i].rank(l,0);
                    int l0 = ba[i].rank(l,0), r0 = l0+z;
                    if(r0-l0>0) stk.emplace(l0,r0,i-1,val0);
                }
                if((x&mask)<=val1 && val1<y){
                    int o = ba[i].rank(r,1) - ba[i].rank(l,1);
                    int l1 = nzero[i] + ba[i].rank(l,1), r1 = l1+o;
                    if(r1-l1>0) stk.emplace(l1,r1,i-1,val1);
                }
            }
        }

        return ret;
    }

    long long rangeMax(int l, int r, uint64_t x, uint64_t y){
        vector<pair<uint64_t,int>> v = rangeMaxK(l,r,x,y,1);
        return v.empty() ? -1 : v[0].first;
    }

    long long rangeMax(int l, int r){
        return rangeMax(l,r,0,MASK);
    }

    vector<pair<uint64_t,int>> rangeMinK(int l, int r, uint64_t x, uint64_t y, int k){
        // [l,r)でx<=c<yとなるcを小さい順にk個, 頻度と共に返す
        vector<pair<uint64_t,int>> ret;
        stack<tuple<int,int,int,uint64_t>> stk;
        stk.emplace(l,r,LOGA,0);
        while(!stk.empty() && (int)ret.size()<k){
            tuple<int,int,int,uint64_t> t = stk.top(); stk.pop();
            int i = get<2>(t);
            uint64_t val = get<3>(t);

            if(i<0){
                int freq = get<1>(t) - get<0>(t);
                ret.emplace_back(val,freq);
            }
            else{
                uint64_t val0 = val, val1 = val | (1LLU<<i);
                uint64_t mask = MASK - (1LLU<<i) + 1;

                int l = get<0>(t), r = get<1>(t);
                if((x&mask)<=val1 && val1<y){
                    int o = ba[i].rank(r,1) - ba[i].rank(l,1);
                    int l1 = nzero[i] + ba[i].rank(l,1), r1 = l1+o;
                    if(r1-l1>0) stk.emplace(l1,r1,i-1,val1);
                }
                if((x&mask)<=val0 && val0<y){
                    int z = ba[i].rank(r,0) - ba[i].rank(l,0);
                    int l0 = ba[i].rank(l,0), r0 = l0+z;
                    if(r0-l0>0) stk.emplace(l0,r0,i-1,val0);
                }
            }
        }

        return ret;
    }

    long long rangeMin(int l, int r, uint64_t x, uint64_t y){
        vector<pair<uint64_t,int>> v = rangeMinK(l,r,x,y,1);
        return v.empty() ? -1 : v[0].first;
    }

    long long rangeMin(int l, int r){
        return rangeMin(l,r,0,MASK);
    }

    vector<tuple<uint64_t, uint64_t, uint64_t>> intersect(int l1, int r1, int l2, int r2) {
		// [l1,r1), [l2,r2)に共通して現れる要素を頻度と共に返す
        // (値, [l1,r1)での頻度, [l2,r2)での頻度)
        vector<tuple<uint64_t,uint64_t,uint64_t>> ret;

        queue<tuple<uint64_t,uint64_t,uint64_t,uint64_t,int,uint64_t>> que; // s1, e1, s2, e2, depth, value
        que.push(make_tuple(l1, r1, l2, r2, LOGA, 0));
        while (!que.empty()) {
            auto e = que.front(); que.pop();
            uint64_t s1, e1, s2, e2, val; int i;
            tie(s1, e1, s2, e2, i, val) = e;

            if(i < 0){
                ret.emplace_back(val, r1 - l1, r2 - l2);
                continue;
            }

            // 0
            int l10 = ba[i].rank(l1, 0);
            int r10 = ba[i].rank(r1, 0);
            int l20 = ba[i].rank(l2, 0);
            int r20 = ba[i].rank(r2, 0);
            if(l10<r10 && l20<r20) que.emplace(l10, r10, l20, r20, i-1, val);

            // 1
            int l11 = nzero[i] + ba[i].rank(l1, 1);
            int r11 = nzero[i] + ba[i].rank(r1, 1);
            int l21 = nzero[i] + ba[i].rank(l2, 1);
            int r21 = nzero[i] + ba[i].rank(r2, 1);
            if(l11<r11 && l21<r21) que.emplace(l11, r11, l21, r21, i-1, val | (1LLU<<i));
        }

        return ret;
    }
};
```
