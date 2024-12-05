## Rust
```rust
use std::collections::BTreeSet;
use std::ops::Bound::{Excluded, Included, Unbounded};

pub struct Set<T: Ord> {
    set: BTreeSet<T>,
}

#[allow(dead_code)]
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
```
