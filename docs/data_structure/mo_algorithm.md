## Rust
```rust
pub mod mo {
    use itertools::Itertools;
    use num_integer::Roots;

    pub trait Problem<T: Copy> {
        fn add_id(&mut self, i: usize);
        fn erase_id(&mut self, i: usize);
        fn answer(&mut self) -> T;
    }

    #[derive(Clone, Copy)]
    struct Segment {
        l: usize,
        r: usize,
        query_id: usize,
        block_id: usize,
    }

    pub struct Mo {
        n: usize,
        q: usize,
        l: Vec<usize>,
        r: Vec<usize>,
        sqrt_q: usize,
    }

    impl Mo {
        pub fn new(n: usize, q: usize, l: Vec<usize>, r: Vec<usize>) -> Self {
            let sqrt_q = q.sqrt();
            Self { n, q, l, r, sqrt_q }
        }

        fn get_sorted_segment(&self) -> Vec<Segment> {
            let mut segments = self
                .l
                .iter()
                .zip(self.r.iter())
                .enumerate()
                .map(|(i, (&l, &r))| Segment {
                    l,
                    r,
                    query_id: i,
                    block_id: l / self.sqrt_q,
                })
                .collect_vec();

            segments.sort_by_key(|segment| {
                if (segment.block_id & 1) == 1 {
                    (segment.block_id, self.n - segment.r)
                } else {
                    (segment.block_id, segment.r)
                }
            });

            segments
        }

        pub fn solve<T: Copy>(&mut self, p: &mut impl Problem<T>) -> Vec<T> {
            let (mut l, mut r) = (0, 0);
            let segments = self.get_sorted_segment();
            let mut answers = vec![None; self.q];

            for segment in segments.into_iter() {
                let l_next = segment.l;
                let r_next = segment.r;

                if r < r_next {
                    for i in r..r_next {
                        p.add_id(i);
                    }

                    if l < l_next {
                        for i in l..l_next {
                            p.erase_id(i);
                        }
                    } else {
                        for i in (l_next..l).rev() {
                            p.add_id(i);
                        }
                    }
                } else {
                    if l < l_next {
                        for i in l..l_next {
                            p.erase_id(i);
                        }
                    } else {
                        for i in (l_next..l).rev() {
                            p.add_id(i);
                        }
                    }

                    for i in (r_next..r).rev() {
                        p.erase_id(i);
                    }
                }
                answers[segment.query_id] = Some(p.answer());

                l = l_next;
                r = r_next;
            }

            answers.into_iter().map(|ans| ans.unwrap()).collect_vec()
        }
    }
}
```

### examples

- [ABC174 F - Range Set Query](https://atcoder.jp/contests/abc174/submissions/49764340)
- [ABC242 G - Range Pairing Query](https://atcoder.jp/contests/abc242/submissions/49753477?lang=ja)
- [ABC293 G - Triple Index](https://atcoder.jp/contests/abc293/submissions/49753756)
