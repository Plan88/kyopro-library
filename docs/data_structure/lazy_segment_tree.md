## C++
```c++
template <class S,
          S (*op)(S, S),
          S (*e)(),
          class F,
          S (*mapping)(F, S),
          F (*composition)(F, F),
          F (*id)()>
struct LazySegmentTree {
  public:
    LazySegmentTree() : LazySegmentTree(0) {}
    LazySegmentTree(int n) : LazySegmentTree(std::vector<S>(n, e())) {}
    LazySegmentTree(const std::vector<S>& v) : _n(int(v.size())) {
        log = ceil_pow2(_n);
        size = 1 << log;
        d = std::vector<S>(2 * size, e());
        lz = std::vector<F>(size, id());
        for (int i = 0; i < _n; i++) d[size + i] = v[i];
        for (int i = size - 1; i >= 1; i--) {
            update(i);
        }
    }

    void set(int p, S x) {
        assert(0 <= p && p < _n);
        p += size;
        for (int i = log; i >= 1; i--) push(p >> i);
        d[p] = x;
        for (int i = 1; i <= log; i++) update(p >> i);
    }

    S get(int p) {
        assert(0 <= p && p < _n);
        p += size;
        for (int i = log; i >= 1; i--) push(p >> i);
        return d[p];
    }

    S prod(int l, int r) {
        assert(0 <= l && r <= _n);
        if (l >= r) return e();

        l += size;
        r += size;

        for (int i = log; i >= 1; i--) {
            if (((l >> i) << i) != l) push(l >> i);
            if (((r >> i) << i) != r) push(r >> i);
        }

        S sml = e(), smr = e();
        while (l < r) {
            if (l & 1) sml = op(sml, d[l++]);
            if (r & 1) smr = op(d[--r], smr);
            l >>= 1;
            r >>= 1;
        }

        return op(sml, smr);
    }

    S all_prod() { return d[1]; }

    void apply(int p, F f) {
        assert(0 <= p && p < _n);
        p += size;
        for (int i = log; i >= 1; i--) push(p >> i);
        d[p] = mapping(f, d[p]);
        for (int i = 1; i <= log; i++) update(p >> i);
    }
    void apply(int l, int r, F f) {
        assert(0 <= l && l <= r && r <= _n);
        if (l == r) return;

        l += size;
        r += size;

        for (int i = log; i >= 1; i--) {
            if (((l >> i) << i) != l) push(l >> i);
            if (((r >> i) << i) != r) push((r - 1) >> i);
        }

        {
            int l2 = l, r2 = r;
            while (l < r) {
                if (l & 1) all_apply(l++, f);
                if (r & 1) all_apply(--r, f);
                l >>= 1;
                r >>= 1;
            }
            l = l2;
            r = r2;
        }

        for (int i = 1; i <= log; i++) {
            if (((l >> i) << i) != l) update(l >> i);
            if (((r >> i) << i) != r) update((r - 1) >> i);
        }
    }

    template <class G> int max_right(int l, G g) {
        assert(0 <= l && l <= _n);
        assert(g(e()));
        if (l == _n) return _n;
        l += size;
        for (int i = log; i >= 1; i--) push(l >> i);
        S sm = e();
        do {
            while (l % 2 == 0) l >>= 1;
            if (!g(op(sm, d[l]))) {
                while (l < size) {
                    push(l);
                    l = (2 * l);
                    if (g(op(sm, d[l]))) {
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

    template <class G> int min_left(int r, G g) {
        assert(0 <= r && r <= _n);
        assert(g(e()));
        if (r == 0) return 0;
        r += size;
        for (int i = log; i >= 1; i--) push((r - 1) >> i);
        S sm = e();
        do {
            r--;
            while (r > 1 && (r % 2)) r >>= 1;
            if (!g(op(d[r], sm))) {
                while (r < size) {
                    push(r);
                    r = (2 * r + 1);
                    if (g(op(d[r], sm))) {
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
    std::vector<F> lz;

    void update(int k) { d[k] = op(d[2 * k], d[2 * k + 1]); }
    void all_apply(int k, F f) {
        d[k] = mapping(f, d[k]);
        if (k < size) lz[k] = composition(f, lz[k]);
    }
    void push(int k) {
        all_apply(2 * k, lz[k]);
        all_apply(2 * k + 1, lz[k]);
        lz[k] = id();
    }
    int ceil_pow2(int n) {
        int x = 0;
        while ((1U << x) < (unsigned int)(n)) x++;
        return x;
    }
};
```

## Rust
```rust
mod data_structure {
    pub trait Monoid {
        fn e() -> Self;
        fn op(&self, rhs: &Self) -> Self;
    }

    pub trait Operator<T: Monoid> {
        fn mapping(&self, x: &T) -> T;
        fn composition(&self, rhs: &Self) -> Self;
        fn id() -> Self;
    }

    pub struct LazySegmentTree<T, U> {
        n: usize,     // length of leaf node
        m: usize,     // length of original array
        depth: usize, // depth of tree, root's depth is 1
        val: Vec<T>,
        lazy: Vec<U>,
    }

    impl<T: Monoid, U: Operator<T>, I> From<I> for LazySegmentTree<T, U>
    where
        I: IntoIterator<Item = T>,
    {
        fn from(iter: I) -> Self {
            Self::build(iter.into_iter().collect())
        }
    }

    impl<T: Monoid, U: Operator<T>> LazySegmentTree<T, U> {
        pub fn new(n: usize) -> Self {
            Self::build(Self::build_identity_array(n))
        }

        fn build_identity_array(n: usize) -> Vec<T> {
            (0..n).map(|_| T::e()).collect()
        }

        fn build(v: Vec<T>) -> Self {
            let m = v.len();
            let n = bit_ceil(m);
            let depth = bit_width(n);

            let mut val = Self::build_identity_array(n); // n
            val.extend(v); // n + m
            val.extend(Self::build_identity_array(n - m)); // 2n

            for i in (1..n).rev() {
                let l = i << 1;
                let r = (i << 1) + 1;
                val[i] = val[l].op(&val[r]);
            }

            let lazy = (0..n).map(|_| U::id()).collect();

            Self {
                n,
                m,
                depth,
                val,
                lazy,
            }
        }

        pub fn get(&mut self, pos: usize) -> &T {
            let i = pos + self.n;
            for d in (1..self.depth).rev() {
                self.propagate(i >> d);
            }
            &self.val[i]
        }

        pub fn set(&mut self, pos: usize, x: T) {
            let i = pos + self.n;
            for d in (1..self.depth).rev() {
                self.propagate(i >> d);
            }
            self.val[i] = x;
            for d in 1..self.depth {
                self.update(i >> d);
            }
        }

        pub fn apply<R>(&mut self, range: R, f: U)
        where
            R: std::ops::RangeBounds<usize>,
        {
            let (mut l, mut r) = self.to_half_open_pair(range);
            l += self.n;
            r += self.n;

            for d in (1..self.depth).rev() {
                if ((l >> d) << d) != l {
                    self.propagate(l >> d);
                }
                if ((r >> d) << d) != r {
                    self.propagate((r - 1) >> d);
                }
            }

            {
                let (l2, r2) = (l, r);
                while l < r {
                    if (l & 1) == 1 {
                        self.apply_to_node(l, &f);
                        l += 1;
                    }
                    if (r & 1) == 1 {
                        r -= 1;
                        self.apply_to_node(r, &f);
                    }
                    l >>= 1;
                    r >>= 1;
                }
                (l, r) = (l2, r2);
            }

            for d in 1..self.depth {
                if ((l >> d) << d) != l {
                    self.update(l >> d);
                }
                if ((r >> d) << d) != r {
                    self.update((r - 1) >> d);
                }
            }
        }

        pub fn prod<R>(&mut self, range: R) -> T
        where
            R: std::ops::RangeBounds<usize>,
        {
            let (mut l, mut r) = self.to_half_open_pair(range);
            l += self.n;
            r += self.n;

            for d in (1..self.depth).rev() {
                if ((l >> d) << d) != l {
                    self.propagate(l >> d);
                }
                if ((r >> d) << d) != r {
                    self.propagate((r - 1) >> d);
                }
            }
            let (mut vl, mut vr) = (T::e(), T::e());
            while l < r {
                if (l & 1) == 1 {
                    vl = vl.op(&self.val[l]);
                    l += 1;
                }
                if (r & 1) == 1 {
                    r -= 1;
                    vr = self.val[r].op(&vr);
                }
                l >>= 1;
                r >>= 1;
            }

            vl.op(&vr)
        }

        /// find max r such that f(prod(l, r)) is true
        pub fn max_right<F>(&mut self, l: usize, f: F) -> usize
        where
            F: Fn(&T) -> bool,
        {
            let mut l = l + self.n;
            let mut current_prod = T::e();

            for d in (1..self.depth).rev() {
                self.propagate(l >> d);
            }

            loop {
                while (l & 1) == 0 {
                    l >>= 1;
                }
                let next = current_prod.op(&self.val[l]);
                if !f(&next) {
                    break;
                }
                current_prod = next;
                l += 1;
                if l.is_power_of_two() {
                    return self.m;
                }
            }

            while l < self.n {
                self.propagate(l);
                l <<= 1;
                let next = current_prod.op(&self.val[l]);
                if f(&next) {
                    current_prod = next;
                    l += 1;
                }
            }

            l - self.n
        }

        /// find min l such that f(prod(l, r)) is true
        pub fn min_left<F>(&mut self, r: usize, f: F) -> usize
        where
            F: Fn(&T) -> bool,
        {
            let mut r = r + self.n;
            let mut current_prod = T::e();

            for d in (1..self.depth).rev() {
                self.propagate((r - 1) >> d);
            }

            loop {
                r -= 1;
                while r > 1 && (r & 1) == 1 {
                    r >>= 1;
                }
                let next = self.val[r].op(&current_prod);
                if !f(&next) {
                    break;
                }
                current_prod = next;
                if r.is_power_of_two() {
                    return 0;
                }
            }

            while r < self.n {
                self.propagate(r);
                r = (r << 1) | 1;
                let next = self.val[r].op(&current_prod);
                if f(&next) {
                    current_prod = next;
                    r -= 1;
                }
            }

            r + 1 - self.n
        }

        fn update(&mut self, k: usize) {
            if k >= self.n {
                return;
            }
            self.val[k] = self.val[k << 1].op(&self.val[(k << 1) + 1]);
        }

        fn apply_to_node(&mut self, k: usize, f: &U) {
            self.val[k] = f.mapping(&self.val[k]);
            if k < self.n {
                self.lazy[k] = f.composition(&self.lazy[k]);
            }
        }

        fn propagate(&mut self, k: usize) {
            let f = std::mem::replace(&mut self.lazy[k], U::id());
            for i in 0..2 {
                let ch = (k << 1) + i;
                self.apply_to_node(ch, &f);
            }
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

    fn bit_ceil(x: usize) -> usize {
        if x.is_power_of_two() {
            x
        } else {
            fill_one(x) + 1
        }
    }

    fn bit_width(x: usize) -> usize {
        x.ilog2() as usize + 1
    }

    fn fill_one(mut x: usize) -> usize {
        for i in 0..=5 {
            x |= x >> (1 << i);
        }
        x
    }
}
```

## Example
- [029 - Long Bricks（★5） (Rust)](https://atcoder.jp/contests/typical90/submissions/55437201)
