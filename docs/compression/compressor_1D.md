## C++
```c++
template<class T>
struct Compressor1D {
    Compressor1D(std::vector<T> v) {
        std::sort(v.begin(), v.end());
        v.erase(std::unique(v.begin(), v.end()), v.end());

        int N = v.size();
        restorator.resize(N);
        for(int i = 0; i < N; ++i) {
            compressor[v[i]] = i;
            restorator[i] = v[i];
        }
    }

    int compress(T x) {
        return compressor[x];
    }

    T restore(int x) {
        return restorator[x];
    }

    int size() {
        return restorator.size();
    }

  private:
    std::unordered_map<T, int> compressor;
    std::vector<T> restorator;
};
```

## Rust
```rust
mod compression {
    use std::collections::BTreeMap;

    pub struct Compressor1D<T: Copy + Ord> {
        compressor: BTreeMap<T, usize>,
        restorator: Vec<T>,
    }

    impl<T: Copy + Ord> Compressor1D<T> {
        pub fn new<I: IntoIterator<Item = T>>(v: I) -> Self {
            Self::build(v)
        }

        fn build<I: IntoIterator<Item = T>>(v: I) -> Self {
            let mut compressor = BTreeMap::new();
            let mut v = v.into_iter().collect::<Vec<T>>();
            v.sort();
            let mut counter = 0;
            let mut restorator = vec![];

            for vi in v {
                if compressor.contains_key(&vi) {
                    continue;
                }
                compressor.insert(vi, counter);
                restorator.push(vi);
                counter += 1;
            }

            Self {
                compressor,
                restorator,
            }
        }

        pub fn compress(&self, v: T) -> Option<usize> {
            if let Some(&x) = self.compressor.get(&v) {
                Some(x)
            } else {
                None
            }
        }

        pub fn restore(&self, v: usize) -> Option<T> {
            if v < self.restorator.len() {
                Some(self.restorator[v])
            } else {
                None
            }
        }

        pub fn len(&self) -> usize {
            self.compressor.len()
        }
    }
}
```
