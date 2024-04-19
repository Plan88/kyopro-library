## C++
```c++
struct RNG {
    std::mt19937_64 mt;

    RNG(int seed = -1) {
        if (seed == -1) {
            std::random_device rnd; // 非決定的な乱数生成器
            mt = std::mt19937_64(rnd());
        } else {
            mt = std::mt19937_64(seed);
        }
    }

    unsigned long long gen() {
        return mt();
    }

    template <class T>
    T gen(T l, T r) {
        // generate random number in [l, r]
        return l + gen() % (r - l + 1);
    }
};

template <class T>
struct DynamicRollingHash {
    const long long MASK30 = (1LL << 30) - 1;
    const long long MASK31 = (1LL << 31) - 1;
    const long long MOD = (1LL << 61) - 1;
    const long long MASK61 = MOD;

    struct Data {
        long long hs;
        int length;
        Data(long long hs, int length) : hs(hs), length(length) {}
    };

    int N, size;
    std::vector<long long> b;
    std::vector<Data> data;

    DynamicRollingHash(std::vector<T> &a, long long base_ = -1) {
        build(a, base_);
    }

    DynamicRollingHash(std::string &s, long long base_ = -1) {
        int n = s.size();
        std::vector<T> a(n);
        for (int i = 0; i < n; i++)
            a[i] = s[i];
        build(a, base_);
    }

    long long get(int l, int r) {
        l += size;
        r += size;
        Data vl(0, 0), vr(0, 0);
        int length = 1;
        while (l < r) {
            if (l & 1)
                vl = op(vl, data[l++]);
            if (r & 1)
                vr = op(data[--r], vr);
            l >>= 1;
            r >>= 1;
            length <<= 1;
        }
        return op(vl, vr).hs;
    }

    void set(int p, T x) {
        p += size;
        data[p] = Data(1LL * x, 1);
        while (p > 1) {
            p >>= 1;
            update(p);
        }
    }

  private:
    void build(std::vector<T> &a, long long base_ = -1) {
        long long base = -1;
        if (base_ == -1) {
            RNG rng(base_);
            base = rng.gen(2LL, 1LL << 60);
        } else {
            base = base_;
        }

        N = a.size();
        size = std::bit_ceil(uint32_t(N));
        data.resize(size << 1, Data(0, 0));
        b.resize(size + 1);
        b[0] = 1;
        for (int i = 0; i < N; i++)
            data[size + i] = Data(a[i], 1);

        for (int i = 0; i < size; i++)
            b[i + 1] = Mul(b[i], base);

        for (int i = size - 1; i > 0; i--)
            update(i);
    }

    void update(int k) {
        data[k] = op(data[k << 1], data[(k << 1) + 1]);
    }

    Data op(Data vl, Data vr) {
        long long hs = Mul(vl.hs, b[vr.length]) + vr.hs;
        int length = vl.length + vr.length;
        if (hs >= MOD)
            hs -= MOD;
        return Data(hs, length);
    }

    // mod 2^61-1を計算する関数
    long long CalcMod(long long x) {
        long long xu = x >> 61;
        long long xd = x & MASK61;
        long long ret = xu + xd;
        if (ret >= MOD)
            ret -= MOD;
        return ret;
    }

    // a*b mod 2^61-1を返す関数(最後にModを取る)
    long long Mul(long long a, long long b) {
        long long au = a >> 31;
        long long ad = a & MASK31;
        long long bu = b >> 31;
        long long bd = b & MASK31;
        long long mid = ad * bu + au * bd;
        long long midu = mid >> 30;
        long long midd = mid & MASK30;
        return CalcMod(au * bu * 2 + midu + (midd << 31) + ad * bd);
    }
};
```
```c++
struct RNG {
    std::mt19937_64 mt;

    RNG(int seed = -1) {
        if (seed == -1) {
            std::random_device rnd; // 非決定的な乱数生成器
            mt = std::mt19937_64(rnd());
        } else {
            mt = std::mt19937_64(seed);
        }
    }

    unsigned long long gen() {
        return mt();
    }

    template <class T>
    T gen(T l, T r) {
        // generate random number in [l, r]
        return l + gen() % (r - l + 1);
    }
};

template <class T>
struct DynamicRollingHash {
    const long long MASK30 = (1LL << 30) - 1;
    const long long MASK31 = (1LL << 31) - 1;
    const long long MOD = (1LL << 61) - 1;
    const long long MASK61 = MOD;

    // size := #leaves of segment tree
    // depth := depth of segment tree
    int N, size, depth;
    std::vector<long long> hs, b, lazy;
    std::vector<T> v;

    DynamicRollingHash(std::vector<T> &a, long long base_ = -1) {
        build(a, base_);
    }

    DynamicRollingHash(std::string &s, long long base_ = -1) {
        int n = s.size();
        std::vector<T> a(n);
        for (int i = 0; i < n; i++)
            a[i] = s[i];
        build(a, base_);
    }

    long long get(int l, int r) {
        long long ret = get(r - 1) + MOD - Mul(get(l - 1), b[r - l]);
        if (ret >= MOD)
            ret -= MOD;
        return ret;
    }

    void set(int p, T x) {
        T d = x - v[p];
        v[p] = x;
        apply(p, N, d);
    }

  private:
    void build(std::vector<T> &a, long long base_ = -1) {
        long long base = -1;
        if (base_ == -1) {
            RNG rng(base_);
            base = rng.gen(2LL, 1LL << 60);
        } else {
            base = base_;
        }

        N = a.size();
        hs.resize(N + 1);
        b.resize(N + 1);
        b[0] = 1;
        hs[0] = 0;
        for (int i = 0; i < N; i++) {
            b[i + 1] = Mul(b[i], base);
            hs[i + 1] = Mul(hs[i], base) + a[i];
            if (hs[i + 1] >= MOD)
                hs[i + 1] -= MOD;
        }

        build_lazy(a);
    }

    void build_lazy(std::vector<T> &a) {
        N = a.size();
        size = std::bit_ceil(uint32_t(N));
        depth = std::bit_width(uint32_t(size)) - 1;
        v = a;
        lazy.resize(size << 1, 0);
    }

    long long get(int x) {
        if (x == -1)
            return 0;
        x += size;
        int length = size;
        for (int i = depth; i >= 0; i--) {
            push(x >> i, length);
            length >>= 1;
        }
        return hs[x - size + 1];
    }

    void apply(int l, int r, long long x) {
        l += size;
        r += size;
        int l0 = l, i = 0;
        while (l < r) {
            if (l & 1) {
                assign_add(lazy[l], Mul(x, b[(l << i) - l0]));
                l++;
            }
            if (r & 1) {
                r--;
                assign_add(lazy[r], Mul(x, b[(r << i) - l0]));
            }
            l >>= 1;
            r >>= 1;
            i++;
        }
    }

    void push(int k, int segment_length) {
        // lazy[k] を子に伝播させる
        if (k < size) {
            int l = k << 1, r = l + 1;
            assign_add(lazy[l], lazy[k]);
            assign_add(lazy[r], Mul(lazy[k], b[segment_length >> 1]));
        } else {
            assign_add(hs[k - size + 1], lazy[k]);
        }
        lazy[k] = 0;
    }

    // (a += b) %= MOD をする関数
    void assign_add(long long &a, long long b) {
        a += b;
        if (a >= MOD)
            a -= MOD;
    }

    // mod 2^61-1を計算する関数
    long long CalcMod(long long x) {
        long long xu = x >> 61;
        long long xd = x & MASK61;
        long long ret = xu + xd;
        if (ret >= MOD)
            ret -= MOD;
        return ret;
    }

    // a*b mod 2^61-1を返す関数(最後にModを取る)
    long long Mul(long long a, long long b) {
        long long au = a >> 31;
        long long ad = a & MASK31;
        long long bu = b >> 31;
        long long bd = b & MASK31;
        long long mid = ad * bu + au * bd;
        long long midu = mid >> 30;
        long long midd = mid & MASK30;
        return CalcMod(au * bu * 2 + midu + (midd << 31) + ad * bd);
    }
};
```

## Example

- [ABC331 F - Palindrome Query (C++)](https://atcoder.jp/contests/abc331/submissions/52514217)
