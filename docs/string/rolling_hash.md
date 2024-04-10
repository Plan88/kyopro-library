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

struct RollingHash {
    const long long MASK30 = (1LL << 30) - 1;
    const long long MASK31 = (1LL << 31) - 1;
    const long long MOD = (1LL << 61) - 1;
    const long long MASK61 = MOD;

    int N;
    std::vector<long long> hs, b;
    long long base;

    template <typename T>
    RollingHash(std::vector<T> &a, long long base_ = -1) {
        _build(a, base_);
    }

    RollingHash(std::string &s, long long base_ = -1) {
        int n = s.size();
        std::vector<char> a(n);
        for (int i = 0; i < n; i++)
            a[i] = s[i];
        _build(a, base_);
    }

    long long get(int l, int r) {
        long long ret = hs[r] + MOD - Mul(hs[l], b[r - l]);
        if (ret >= MOD)
            ret -= MOD;
        return ret;
    }

  private:
    template <typename T>
    void _build(std::vector<T> &a, long long base_ = -1) {
        if (base_ == -1) {
            RNG rng(base_);
            base = rng.gen(1LL, 1LL << 60);
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

- [Codeforces Round 934 (Div. 1) B. Non-Palindromic Substring (C++)](https://codeforces.com/contest/1943/submission/254546503)
