## C++
```c++
template <class S, S (*op)(S, S), S (*e)()> struct SegmentTree {
  public:
    SegmentTree() : SegmentTree(0) {}
    SegmentTree(int n) : SegmentTree(std::vector<S>(n, e())) {}
    SegmentTree(const std::vector<S>& v) : _n(int(v.size())) {
        log = ceil_pow2(_n);
        size = 1 << log;
        d = std::vector<S>(2 * size, e());
        for (int i = 0; i < _n; i++) d[size + i] = v[i];
        for (int i = size - 1; i >= 1; i--) {
            update(i);
        }
    }

    void set(int p, S x) {
        assert(0 <= p && p < _n);
        p += size;
        d[p] = x;
        for (int i = 1; i <= log; i++) update(p >> i);
    }

    S get(int p) {
        assert(0 <= p && p < _n);
        return d[p + size];
    }

    S prod(int l, int r) {
        assert(0 <= l && r <= _n);
        S sml = e(), smr = e();
        l += size;
        r += size;

        while (l < r) {
            if (l & 1) sml = op(sml, d[l++]);
            if (r & 1) smr = op(d[--r], smr);
            l >>= 1;
            r >>= 1;
        }
        return op(sml, smr);
    }

    S all_prod() { return d[1]; }

    template <class F> int max_right(int l, F f) {
        assert(0 <= l && l <= _n);
        assert(f(e()));
        if (l == _n) return _n;
        l += size;
        S sm = e();
        do {
            while (l % 2 == 0) l >>= 1;
            if (!f(op(sm, d[l]))) {
                while (l < size) {
                    l = (2 * l);
                    if (f(op(sm, d[l]))) {
                        sm = op(sm, d[l]);
                        l++;
                    }
                }
                return l - size;
            }
            sm = op(sm, d[l]);
            l++;
        } while ((l & -l) != l);
        return _n;
    }

    template <class F> int min_left(int r, F f) {
        assert(0 <= r && r <= _n);
        assert(f(e()));
        if (r == 0) return 0;
        r += size;
        S sm = e();
        do {
            r--;
            while (r > 1 && (r % 2)) r >>= 1;
            if (!f(op(d[r], sm))) {
                while (r < size) {
                    r = (2 * r + 1);
                    if (f(op(d[r], sm))) {
                        sm = op(d[r], sm);
                        r--;
                    }
                }
                return r + 1 - size;
            }
            sm = op(d[r], sm);
        } while ((r & -r) != r);
        return 0;
    }

  private:
    int _n, size, log;
    std::vector<S> d;

    void update(int k) { d[k] = op(d[2 * k], d[2 * k + 1]); }

    int ceil_pow2(int n) {
        int x = 0;
        while ((1U << x) < (unsigned int)(n)) x++;
        return x;
    }
};
```

## Rust
```rust
pub mod data_structure {
    use num::Integer;

    pub trait Monoid {
        fn e() -> Self;
        fn op(&self, rhs: Self) -> Self;
    }

    pub struct SegmentTree<T> {
        n: usize, // m <= n となる最大の 2 の冪乗
        m: usize, // 元の配列の長さ
        val: Vec<T>,
    }

    impl<T: Monoid + Copy> SegmentTree<T> {
        pub fn from_length(n: usize) -> Self {
            let val = vec![T::e(); n];
            Self::build(val)
        }

        pub fn from_array(v: &[T]) -> Self {
            let val = v.to_vec();
            Self::build(val)
        }

        fn build(v: Vec<T>) -> Self {
            let m = v.len();
            let mut n = 1;
            while n < m {
                n <<= 1;
            }

            let mut val = vec![T::e(); n]; // n
            val.extend(v); // n + m
            val.extend(vec![T::e(); n - m]); // 2n

            for i in (1..n).rev() {
                let l = i << 1;
                let r = (i << 1) + 1;
                val[i] = val[l].op(val[r]);
            }

            Self { n, m, val }
        }

        pub fn get(&self, pos: usize) -> T {
            self.val[pos + self.n]
        }

        pub fn set(&mut self, pos: usize, x: T) {
            let mut i = pos + self.n;
            self.val[i] = x;
            while i > 1 {
                i >>= 1;
                let l = i << 1;
                let r = (i << 1) + 1;
                self.val[i] = self.val[l].op(self.val[r]);
            }
        }

        pub fn prod<R>(&self, range: R) -> T
        where
            R: std::ops::RangeBounds<usize>,
        {
            let (mut l, mut r) = self.to_half_open_pair(range);
            l += self.n;
            r += self.n;
            let mut v = T::e();
            while l < r {
                if (l & 1) == 1 {
                    v = v.op(self.val[l]);
                    l += 1;
                }
                if (r & 1) == 1 {
                    r -= 1;
                    v = v.op(self.val[r]);
                }
                l >>= 1;
                r >>= 1;
            }

            v
        }

        /// find max r such that f(prod(l, r)) is true
        pub fn max_right<F>(&self, l: usize, f: F) -> usize
        where
            F: Fn(T) -> bool,
        {
            let mut l = l + self.n;
            let mut current_prod = T::e();

            loop {
                while l.is_even() {
                    l >>= 1;
                }
                let next = current_prod.op(self.val[l]);
                if !f(next) {
                    break;
                }
                current_prod = next;
                l += 1;
                if l.is_power_of_two() {
                    return self.m;
                }
            }

            while l < self.n {
                l <<= 1;
                let next = current_prod.op(self.val[l]);
                if f(next) {
                    current_prod = next;
                    l += 1;
                }
            }

            l - self.n
        }

        /// find min l such that f(prod(l, r)) is true
        pub fn min_left<F>(&self, r: usize, f: F) -> usize
        where
            F: Fn(T) -> bool,
        {
            let mut r = r + self.n;
            let mut current_prod = T::e();

            loop {
                r -= 1;
                while r > 1 && r.is_odd() {
                    r >>= 1;
                }
                let next = self.val[r].op(current_prod);
                if !f(next) {
                    break;
                }
                current_prod = next;
                if r.is_power_of_two() {
                    return 0;
                }
            }

            while r < self.n {
                r = (r << 1) | 1;
                let next = self.val[r].op(current_prod);
                if f(next) {
                    current_prod = next;
                    r -= 1;
                }
            }

            r + 1 - self.n
        }

        /// range = [l, r) と表せる (l, r) を返す
        fn to_half_open_pair<R>(&self, range: R) -> (usize, usize)
        where
            R: std::ops::RangeBounds<usize>,
        {
            let l = match range.start_bound() {
                std::ops::Bound::Included(&l) => l,
                std::ops::Bound::Excluded(&l) => l + 1,
                std::ops::Bound::Unbounded => 0,
            };
            let r = match range.end_bound() {
                std::ops::Bound::Included(&r) => r + 1,
                std::ops::Bound::Excluded(&r) => r,
                std::ops::Bound::Unbounded => self.m,
            };
            (l, r)
        }
    }
}
```
