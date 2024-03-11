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
```
