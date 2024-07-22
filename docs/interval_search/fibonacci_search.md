## Rust
```rust
mod interval_search_for_interactive_problem {
    type Output = i64;

    pub struct QueryProcesser {}
    impl QueryProcesser {
        pub fn new() -> Self {
            todo!();
        }
        fn query(&mut self, i: i64) -> Output {
            todo!("ここにクエリの処理を書く");
        }
    }

    pub struct FibonacciSearch {
        ml: i64,     // ng_min から分割点の左までの長さ
        mr: i64,     // ng_min から分割点の右までの長さ
        ng_min: i64, // 定義域の最小値 - 1
        ng_max: i64, // 定義域の最大値 + 1
    }

    impl FibonacciSearch {
        /// low: 定義域の最小値
        /// high: 定義域の最大値
        pub fn new(low: i64, high: i64) -> Self {
            Self::build(low, high)
        }

        fn build(low: i64, high: i64) -> Self {
            let (ng_min, ng_max) = (low - 1, high + 1);
            let (mut ml, mut mr) = (1, 1);

            // 探索区間の長さ以上となる最小のフィボナッチ数を求める
            while ml + mr < ng_max - ng_min {
                ml += mr;
                std::mem::swap(&mut ml, &mut mr);
            }

            Self {
                ml,
                mr,
                ng_min,
                ng_max,
            }
        }

        /// eval_fn: eval_fn(x, y) に対して x の方が評価が高いとき true, そうでないとき false を返すclosure
        pub fn solve<F>(&mut self, mut f: QueryProcesser, eval_fn: F) -> i64
        where
            F: Fn(Output, Output) -> bool,
        {
            let mut lval = f.query(self.ng_min + self.ml);
            let mut rval = f.query(self.ng_min + self.mr);

            while self.ml < self.mr {
                self.shrink_interval();

                if eval_fn(lval, rval) {
                    rval = lval;
                    lval = f.query(self.ng_min + self.ml);
                } else {
                    self.ng_min += self.mr;
                    if self.ng_min + self.mr < self.ng_max {
                        lval = rval;
                        rval = f.query(self.ng_min + self.mr);
                    } else {
                        self.shrink_interval();
                        lval = f.query(self.ng_min + self.ml);
                    }
                }
            }

            if eval_fn(lval, rval) {
                lval
            } else {
                rval
            }
            }

            if eval_fn(lval, rval) {
                self.ml
            } else {
                self.mr
            }
        }

        fn shrink_interval(&mut self) {
            self.mr -= self.ml;
            std::mem::swap(&mut self.ml, &mut self.mr);
        }
    }
}
```

## Example
- [053 - Discrete Dowsing（★7） (Rust)](https://atcoder.jp/contests/typical90/submissions/55886383)
