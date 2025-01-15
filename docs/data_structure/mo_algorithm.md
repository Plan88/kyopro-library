## C++
```c++
template <class T>
struct Problem {
    Problem() {
    }

    void add_id(int i) {
        // process for adding index i
    }
    void erase_id(int i) {
        // process for erasing index i
    }
    T get_answer() {
        // return current answer
    }
};

struct Segment {
    int L, R, query_id, block_id;
    Segment(int L, int R, int query_id, int block_id) : L(L), R(R), query_id(query_id), block_id(block_id) {}

    std::tuple<int, int, int> get_key() const {
        if (block_id & 1) {
            return std::make_tuple(block_id, -R, L);
        }
        return std::make_tuple(block_id, R, L);
    }
};

bool operator<(const Segment &lhs, const Segment &rhs) {
    return lhs.get_key() < rhs.get_key();
}

template <class T>
struct Mo {
    int N, Q, sqrt_Q;
    std::vector<int> L, R;

    Mo(int N, int Q, std::vector<int> L, std::vector<int> R) : N(N), Q(Q), L(L), R(R) {
        this->sqrt_Q = calc_sqrt(Q);
    }

    std::vector<Segment> get_sorted_segments() {
        std::vector<Segment> segments;
        for (int i = 0; i < Q; i++) {
            segments.push_back(Segment(L[i], R[i], i, L[i] / sqrt_Q));
        }
        std::sort(segments.begin(), segments.end());
        return segments;
    }

    std::vector<T> solve(Problem<T> p) {
        int l = 0, r = 0;
        std::vector<Segment> segments = get_sorted_segments();
        std::vector<T> answers(Q);

        for (Segment &segment : segments) {
            int l_next = segment.L, r_next = segment.R;

            if (r < r_next) {
                for (int i = r; i < r_next; i++) {
                    p.add_id(i);
                }

                if (l < l_next) {
                    for (int i = l; i < l_next; i++) {
                        p.erase_id(i);
                    }
                } else {
                    for (int i = l - 1; i >= l_next; i--) {
                        p.add_id(i);
                    }
                }
            } else {
                if (l < l_next) {
                    for (int i = l; i < l_next; i++) {
                        p.erase_id(i);
                    }
                } else {
                    for (int i = l - 1; i >= l_next; i--) {
                        p.add_id(i);
                    }
                }

                for (int i = r - 1; i >= r_next; i--) {
                    p.erase_id(i);
                }
            }

            answers[segment.query_id] = p.get_answer();
            l = l_next;
            r = r_next;
        }

        return answers;
    }

  private:
    int calc_sqrt(int x) {
        int ng = 0, ok = sqrt(x) + 2;
        while (ok - ng > 1) {
            int m = (ok + ng) / 2;
            if (m * m < x)
                ng = m;
            else
                ok = m;
        }
        return ok;
    }
};
```

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

        pub fn solve<T: Copy, P: Problem<T>>(&mut self, p: &mut P) -> Vec<T> {
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

## Example

- [ABC174 F - Range Set Query (Rust)](https://atcoder.jp/contests/abc174/submissions/49764340)
- [ABC238 G - Cubic? (C++)](https://atcoder.jp/contests/abc238/submissions/51889578)
- [ABC242 G - Range Pairing Query (Rust)](https://atcoder.jp/contests/abc242/submissions/49753477?lang=ja)
- [ABC293 G - Triple Index (Rust)](https://atcoder.jp/contests/abc293/submissions/49753756)
