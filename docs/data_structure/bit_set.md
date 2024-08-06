## Rust
```rust
#[allow(dead_code)]
mod data_structure {
    use itertools::Itertools;
    use num::PrimInt;

    #[derive(Clone, Debug)]
    pub struct BitSet {
        internal: Vec<u128>,
    }

    impl BitSet {
        pub fn new(length: usize) -> Self {
            Self {
                internal: vec![0; length],
            }
        }

        pub fn from_integer<T: PrimInt>(x: T) -> Self {
            assert!(x >= T::zero());
            Self {
                internal: vec![x.to_u128().unwrap()],
            }
        }

        pub fn from_string(s: String) -> Self {
            assert!(is_01string(&s));
            let n = s.len();
            let mut internal = vec![];
            for i in (0..n).step_by(128) {
                let si = &s[i..i + 128];
                internal.push(u128::from_str_radix(si, 2).unwrap());
            }
            Self { internal }
        }

        pub fn from_bytes(bytes: Vec<u8>) -> Self {
            assert!(is_01bytes(&bytes));
            let n = bytes.len();
            let mut internal = vec![];
            for i in (0..n).step_by(128) {
                let si = &bytes[i..i + 128].iter().join("");
                internal.push(u128::from_str_radix(si, 2).unwrap());
            }
            Self { internal }
        }

        pub fn assign(&mut self, i: usize, b: u128) {
            self.extend(i);
            let (i, j) = get_pos_and_bit(i);
            if b == 1 {
                self.internal[i] |= 1 << j;
            } else {
                let mask = u128::MAX - (1 << j);
                self.internal[i] &= mask;
            }
        }

        pub fn is_one_bit(&self, i: usize) -> bool {
            assert!(!self.is_out_of_range(i));
            let (i, j) = get_pos_and_bit(i);
            (self.internal[i] >> j & 1) == 1
        }
        pub fn is_zero_bit(&self, i: usize) -> bool {
            !self.is_one_bit(i)
        }
        pub fn is_zero(&self) -> bool {
            self.internal.iter().all(|x| x == &0)
        }

        pub fn xor_inplace(&mut self, other: &Self) {
            let n = self.internal.len().min(other.internal.len());
            for i in 0..n {
                self.internal[i] ^= other.internal[i];
            }
            for i in n..other.internal.len() {
                self.internal.push(other.internal[i]);
            }
        }

        pub fn xor(&self, other: &Self) -> Self {
            let mut xor = Self {
                internal: self.internal.clone(),
            };
            xor.xor_inplace(other);
            xor
        }

        pub fn min(&self, other: &Self) -> Self {
            let m = self.internal.len().max(other.internal.len());
            for i in (0..m).rev() {
                let x = if i < self.internal.len() {
                    self.internal[i]
                } else {
                    0
                };
                let y = if i < other.internal.len() {
                    other.internal[i]
                } else {
                    0
                };

                if x == y {
                    continue;
                } else if x < y {
                    return Self {
                        internal: self.internal.clone(),
                    };
                } else {
                    return Self {
                        internal: other.internal.clone(),
                    };
                }
            }
            Self {
                internal: self.internal.clone(),
            }
        }

        fn is_out_of_range(&self, i: usize) -> bool {
            i / 128 >= self.internal.len()
        }
        fn extend(&mut self, i: usize) {
            while self.internal.len() <= i / 128 {
                self.internal.push(0);
            }
        }
    }

    fn is_01string(s: &str) -> bool {
        is_01bytes(s.as_bytes())
    }
    fn is_01bytes(bytes: &[u8]) -> bool {
        bytes
            .iter()
            .filter(|bi| bi != &&b'0' || bi != &&b'1')
            .collect::<Vec<_>>()
            .is_empty()
    }
    fn get_pos_and_bit(i: usize) -> (usize, usize) {
        (i / 128, i % 128)
    }
}
```
