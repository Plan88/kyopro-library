## C++
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

## Rust
```rust
pub mod data_structure {
    use std::ops::{Add, Sub};

    use num::Zero;

    pub trait FenwickTreeTrait: Add<Output = Self> + Copy {
        fn add_fenwick(&self, rhs: Self) -> Self {
            *self + rhs
        }
    }
    impl FenwickTreeTrait for i64 {}

    pub struct FenwickTree<T: FenwickTreeTrait> {
        n: usize,
        data: Vec<T>,
    }

    impl<T: FenwickTreeTrait> FenwickTree<T> {
        pub fn from_iterable<I: IntoIterator<Item = T>>(iter: I) -> Self {
            let data: Vec<T> = iter.into_iter().collect();
            let n = data.len();
            Self { n, data }
        }

        /// add x at pos (0-indexed)
        pub fn add(&mut self, mut pos: usize, x: T) {
            pos += 1;
            while pos <= self.n {
                self.data[pos] = self.data[pos].add_fenwick(x);
                pos += lsb(pos);
            }
        }
    }

    impl<T: FenwickTreeTrait + Zero> FenwickTree<T> {
        pub fn new(n: usize) -> Self {
            Self {
                n,
                data: vec![T::zero(); n + 1],
            }
        }

        /// return sum of [0, pos) (0-indexed)
        pub fn cumsum(&self, mut pos: usize) -> T {
            let mut sum = T::zero();
            while pos > 0 {
                sum = sum.add_fenwick(self.data[pos]);
                pos -= lsb(pos);
            }
            sum
        }
    }

    impl<T: FenwickTreeTrait + Zero + Sub<Output = T>> FenwickTree<T> {
        /// return sum of [left, right) (0-indexed)
        pub fn sum(&self, left: usize, right: usize) -> T {
            self.cumsum(right) - self.cumsum(left)
        }
    }

    fn lsb(x: usize) -> usize {
        let x_i32 = x as i32;
        (x_i32 & -x_i32) as usize
    }
}
```
