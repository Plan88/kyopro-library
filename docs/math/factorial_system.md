## C++
```c++
template <class T = int> struct FenwickTree {
  public:
    FenwickTree() : _n(0) {}
    explicit FenwickTree(int n) : _n(n), data(n) {}

    void add(int p, T x) {
        p++;
        while (p <= _n) {
            data[p - 1] += x;
            p += p & -p;
        }
    }

    T sum(int l, int r) {
        return sum(r) - sum(l);
    }

  private:
    int _n;
    std::vector<T> data;

    T sum(int r) {
        T s = 0;
        while (r > 0) {
            s += data[r - 1];
            r -= r & -r;
        }
        return s;
    }
};

struct FactorialSystem {
    std::vector<int> a;

    FactorialSystem(std::vector<int> &a) {
        build(a);
    }
    FactorialSystem(int N, bool zero = true) {
        // if zero then 0 の長さ N の階乗進法表記 else N! の階乗進法表記
        if (zero) {
            std::vector<int> a(N, 0);
            build(a);
        } else {
            std::vector<int> a(N + 1, 0);
            a[N] = 1;
            build(a);
        }
    }
    FactorialSystem(uint64_t X) {
        // X の階乗進法表記
        std::vector<int> a;
        std::uint64_t p = 1;

        while(X) {
            int ai = X % p;
            X /= p++;
            a.push_back(ai);
        }

        build(a);
    }

    std::vector<int> to_permutation(bool remove_leading_zero = false) {
        int N = size(), threshold = -1;
        std::vector<int> p(N);
        FenwickTree fenwick_tree(N);

        for(int i = N - 1; i >= 0; i--) {
            int ok = 0, ng = N;
            while(ng - ok > 1) {
                int m = (ok + ng) / 2;
                if(m - fenwick_tree.sum(0, m) <= a[i])
                    ok = m;
                else
                    ng = m;
            }

            p[N - 1 - i] = ok;
            fenwick_tree.add(ok, 1);
        }
        return p;
    }

    FactorialSystem& operator+=(FactorialSystem &fs) {
        int N = fs.size(), kuriage = 0;
        for(int i = 0; i < N; i++) {
            if(a.size() < i + 1) {
                a.push_back(0);
            }
            a[i] += fs.a[i] + kuriage;
            kuriage = a[i] / (i + 1);
            a[i] %= i + 1;
        }

        int n = N;
        while(kuriage > 0) {
            if(a.size() < n + 1) {
                a.push_back(0);
            }
            a[n] += kuriage;
            kuriage = a[n] / (n + 1);
            a[n] %= n + 1;
            n++;
        }
        return *this;
    }
    FactorialSystem& operator+=(const std::uint64_t x) {
        FactorialSystem fs(x);
        *this += fs;
        return *this;
    }
    FactorialSystem operator+(FactorialSystem &fs) {
        FactorialSystem res(*this);
        return res += fs;
    }
    FactorialSystem operator+(const std::uint64_t x) {
        FactorialSystem res(*this);
        return res += x;
    }

    FactorialSystem& operator-=(FactorialSystem &fs) {
        // assume this >= fs
        int N = fs.size();
        for(int i = N - 1; i >= 0; i--) {
            if (a[i] < fs.a[i]) {
                int pos = i + 1;
                while(a[pos] == 0)
                    pos++;
                for(int j = pos; j > i; j--) {
                    a[j]--;
                    a[j - 1] += j;
                }
            }
            a[i] -= fs.a[i];
        }

        return *this;
    }
    FactorialSystem& operator-=(const std::uint64_t x) {
        // assume this >= x
        FactorialSystem fs(x);
        *this -= fs;
        return *this;
    }
    FactorialSystem operator-(FactorialSystem &fs) {
        FactorialSystem res(*this);
        return res -= fs;
    }
    FactorialSystem operator-(const std::uint64_t x) {
        FactorialSystem res(*this);
        return res -= x;
    }

    FactorialSystem& operator*=(const int x) {
        int N = size();
        std::uint64_t kuriage = 0;
        for(int i = 0; i < N; i++) {
            std::uint64_t sum = 1ULL * a[i] * x + kuriage;
            a[i] = sum % (i + 1);
            kuriage = sum / (i + 1);
        }

        int n = N;
        while(kuriage > 0) {
            a.push_back(0);
            a[n] = kuriage % (n + 1);
            kuriage /= (n + 1);
            n++;
        }

        return *this;
    }
    FactorialSystem operator*(const int x) {
        FactorialSystem res(*this);
        return res *= x;
    }

    FactorialSystem& operator/=(const int x) {
        // 階乗進法の割り算 O(N)
        int N = size();
        std::uint64_t kuriage = 0;
        for(int i = N - 1; i >= 0; i--) {
            std::uint64_t sum = kuriage + a[i];
            if(kuriage + a[i] < x) {
                kuriage = sum * i;
                a[i] = 0;
                continue;
            }
            a[i] = sum / x;
            kuriage = (sum % x) * i;
        }

        return *this;
    }
    FactorialSystem operator/(const int x) {
        FactorialSystem res(*this);
        return res /= x;
    }

    size_t size() {
        return a.size();
    }

    void truncate(int N) {
        while(a.size() > N and a.back() == 0)
            a.pop_back();
    }

    void print() {
        for(int i = 0; i < size(); i ++) {
            std::cout << a[i] << " \n"[i == size() - 1];
        }
    }

  private:
    void build(std::vector<int> &a_) {
        a = a_;
    }
};
```

## Example

- [ARC047 C - N!÷K番目の単語 (C++)](https://atcoder.jp/contests/arc047/submissions/53031226)
