## C++
```c++
template<class T>
struct Set {
    Set(ll none): none(none) {}

    void insert(T v) {
        st.insert(v);
    }

    void erase(T v) {
        st.erase(v);
    }

    bool contains(T v) {
        return st.contains(v);
    }

    bool is_none(T v) {
        return v == none;
    }

    // minimum x such that v <= x
    T ge(T v) {
        auto itr = st.lower_bound(v);
        if(itr == st.end()) return none;
        return *itr;
    }

    // minimum x such that v < x
    T gt(T v) {
        auto itr = st.upper_bound(v);
        if(itr == st.end()) return none;
        return *itr;
    }

    // maximum x such that v >= x
    T le(T v) {
        if(st.empty()) return none;
        auto itr = st.lower_bound(v);
        if(itr == st.begin()) {
            if(*itr <= v) return *itr;
            else return none;
        }

        if(itr == st.end()) itr--;
        else if(*itr > v) itr--;
        return *itr;
    }

    // minimum x such that v > x
    T lt(T v) {
        if(st.empty()) return none;
        auto itr = st.lower_bound(v);
        if(itr == st.begin()) {
            return none;
        }

        itr--;
        return *itr;
    }
  private:
    T none;
    std::set<T> st;
};
```

## Rust
```rust
mod data_structure {
    use std::collections::BTreeSet;
    use std::ops::Bound::{Excluded, Included, Unbounded};

    pub struct Set<T: Ord> {
        set: BTreeSet<T>,
    }

    impl<T: Ord> Set<T> {
        pub fn new() -> Self {
            Self {
                set: BTreeSet::<T>::new(),
            }
        }

        pub fn insert(&mut self, value: T) -> bool {
            self.set.insert(value)
        }

        pub fn erase(&mut self, value: &T) -> bool {
            self.set.remove(value)
        }

        pub fn contains(&self, value: &T) -> bool {
            self.set.contains(value)
        }

        /// maximum value of elements less than given value
        pub fn lt(&self, value: &T) -> Option<&T> {
            self.set.range((Unbounded, Excluded(value))).last()
        }

        /// maximum value of elements less than or equal to given value
        pub fn le(&self, value: &T) -> Option<&T> {
            self.set.range((Unbounded, Included(value))).last()
        }

        /// minimum value of elements more than given value
        pub fn gt(&self, value: &T) -> Option<&T> {
            self.set.range((Excluded(value), Unbounded)).next()
        }

        /// minimum value of elements more than or equal to given value
        pub fn ge(&self, value: &T) -> Option<&T> {
            self.set.range((Included(value), Unbounded)).next()
        }
    }
}
```
