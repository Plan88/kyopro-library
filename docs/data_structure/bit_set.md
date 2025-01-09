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

        /// bytes から生成
        /// bytes の i 番目の要素は下から i ビット目になる
        pub fn from_bytes(bytes: Vec<u8>) -> Self {
            assert!(is_01bytes(&bytes));
            let n = bytes.len();
            let mut internal = vec![];
            for i in (0..n).step_by(128) {
                let string = bytes[i..(i + 128).min(n)]
                    .iter()
                    .rev()
                    .map(|&byte| byte - b'0')
                    .join("");
                internal.push(u128::from_str_radix(&string, 2).unwrap());
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

        pub fn count(&self) -> usize {
            let mut count = 0;
            for &integer in self.internal.iter() {
                count += integer.count_ones();
            }
            count as usize
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

        pub fn and_inplace(&mut self, other: &Self) {
            let n = self.internal.len().min(other.internal.len());
            for i in 0..n {
                self.internal[i] &= other.internal[i];
            }
            while self.internal.len() > n {
                self.internal.pop();
            }
        }
        pub fn and(&self, other: &Self) -> Self {
            let mut and = Self {
                internal: self.internal.clone(),
            };
            and.and_inplace(other);
            and
        }

        pub fn or_inplace(&mut self, other: &Self) {
            let n = self.internal.len().min(other.internal.len());
            for i in 0..n {
                self.internal[i] |= other.internal[i];
            }
            for i in n..other.internal.len() {
                self.internal.push(other.internal[i]);
            }
        }
        pub fn or(&self, other: &Self) -> Self {
            let mut or = Self {
                internal: self.internal.clone(),
            };
            or.or_inplace(other);
            or
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
        bytes.iter().all(|bi| bi == &b'0' || bi == &b'1')
    }
    fn get_pos_and_bit(i: usize) -> (usize, usize) {
        (i / 128, i % 128)
    }
}
```

## Examples

- [ABC258 G - Triangle](https://atcoder.jp/contests/abc258/submissions/61501928)
