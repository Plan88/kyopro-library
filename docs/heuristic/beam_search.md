## Rust
```rust
use beam_search::beam_search;
use domain::State;
use problem_io::Input;

const TURN: usize = 49;
const BEAM_WIDTH: usize = 200;

fn main() {
    let input = Input::new();
    let state = State::new(input.clone());

    let mut solution = beam_search(state, TURN, BEAM_WIDTH);
}

pub mod problem_io {
    use proconio::input;

    #[derive(Clone)]
    pub struct Input {}

    impl Input {
        pub fn new() -> Self {
            todo!();
        }
    }
}

pub mod domain {
    pub type Score = f64;
    pub type Action = Option<()>;

    #[derive(Clone)]
    pub struct State {}

    impl State {
        /// 初期状態の生成
        pub fn new(input: Input) -> State {
            todo!();
        }

        pub fn eval(&self) -> Score {
            todo!();
        }
        pub fn hash(&self) -> u64 {
            todo!();
        }

        /// スコアとハッシュの差分計算
        /// 状態は更新しない
        pub fn try_apply(&mut self, op: Action, _score: Score, _hash: u64) -> (Score, u64) {
            todo!();
        }

        /// 状態を更新する
        /// 元の状態に戻すための情報を返す
        pub fn apply(&mut self, op: Action) -> Action {
            todo!();
        }

        /// applyから返された情報をもとに状態を元に戻す
        pub fn rollback(&mut self, backup: Action) {
            todo!();
        }

        /// 可能な操作の候補を生成する
        pub fn generate_op(&self) -> Vec<Action> {
            todo!();
        }
    }
}

pub mod beam_search {
    use super::domain::{Action, Score, State};
    use std::cell::UnsafeCell;
    use std::rc::*;

    struct Candidate {
        op: Action,
        parent: Rc<Node>,
        score: Score,
        hash: u64,
        p: usize, // 優先度(複数もたせたほうが良い場合があるかもしれない。)
    }

    struct Node {
        parent: Option<(Action, Rc<Node>)>, // 操作、親への参照
        // 速度のためにUnsafeCellを使っているがRefCellのほうが安全
        child: UnsafeCell<Vec<(Action, Weak<Node>)>>, // 操作、子への参照
        score: Score,
        hash: u64,
    }

    // 多スタート用に構造体にまとめておくと楽
    struct Tree {
        state: State,
        node: Rc<Node>,
    }

    impl Tree {
        // 注意: depthは深くなっていくごとに-1されていく
        fn dfs(&mut self, next_states: &mut Vec<Candidate>, p: &mut usize, depth: usize) {
            if depth == 0 {
                let score = self.node.score;
                let hash = self.node.hash;

                // 検算
                // assert_eq!(score, self.state.eval());
                // assert_eq!(hash, self.state.hash());

                // 次の操作を列挙
                for op in self.state.generate_op() {
                    let (next_score, next_hash) = self.state.try_apply(op, score, hash);
                    next_states.push(Candidate {
                        op,
                        parent: self.node.clone(),
                        score: next_score,
                        hash: next_hash,
                        p: *p,
                    });
                    *p += 1;
                }
            } else {
                let node = self.node.clone();
                let child = unsafe { &mut *node.child.get() };
                // 有効な子だけにする
                child.retain(|(_, x)| x.upgrade().is_some());

                for (op, ptr) in child {
                    self.node = ptr.upgrade().unwrap();
                    let backup = self.state.apply(*op);
                    self.dfs(next_states, p, depth - 1);

                    self.state.rollback(backup);
                }

                self.node = node.clone();
            }
        }
    }

    pub fn beam_search(init_state: State, turn: usize, beam_width: usize) -> Vec<Action> {
        let mut tree = {
            let score = init_state.eval();
            let hash = init_state.hash();
            Tree {
                state: init_state,
                node: Rc::new(Node {
                    parent: None,
                    child: UnsafeCell::new(vec![]),
                    score,
                    hash,
                }),
            }
        };

        let mut cur_beam = vec![];
        let mut next_states = vec![];

        let mut set = rustc_hash::FxHashSet::default();

        for depth in 0..turn {
            next_states.clear();
            tree.dfs(&mut next_states, &mut 0, depth);

            if depth + 1 != turn {
                // 上位M個を残す
                if next_states.len() > beam_width {
                    next_states.select_nth_unstable_by(
                        beam_width,
                        |Candidate {
                             score: score1,
                             p: p1,
                             ..
                         },
                         Candidate {
                             score: score2,
                             p: p2,
                             ..
                         }| {
                            (*score1, *p1)
                                .partial_cmp(&(*score2, *p2))
                                .unwrap()
                                .reverse()
                        },
                    );
                    next_states.truncate(beam_width);
                }

                cur_beam.clear();
                set.clear();
                for Candidate {
                    op,
                    parent,
                    score,
                    hash,
                    ..
                } in &next_states
                {
                    // 重複除去
                    if set.insert(*hash) {
                        let child = unsafe { &mut *parent.child.get() };
                        let child_ptr = Rc::new(Node {
                            parent: Some((*op, parent.clone())),
                            child: UnsafeCell::new(vec![]),
                            hash: *hash,
                            score: *score,
                        });
                        cur_beam.push(child_ptr.clone());
                        child.push((*op, Rc::downgrade(&child_ptr)));
                    }
                }
            }
        }

        // 最良の状態を選択
        let Candidate {
            op,
            parent: mut ptr,
            score,
            ..
        } = next_states
            .into_iter()
            .max_by(
                |Candidate { score: score1, .. }, Candidate { score: score2, .. }| {
                    (*score1).partial_cmp(score2).unwrap()
                },
            )
            .unwrap();

        let mut ret = vec![op];
        eprintln!("score: {}", score);

        // 操作の復元
        while let Some((op, parent)) = ptr.parent.clone() {
            ret.push(op);
            ptr = parent.clone();
        }

        ret.reverse();
        ret
    }
}

pub mod rand_generator {
    use num_traits::PrimInt;
    use rand::{seq::SliceRandom, Rng};
    use rand_chacha::{rand_core::SeedableRng, ChaCha20Rng};
    use rand_distr::{Distribution, Normal};

    pub struct RandomGenerator {
        rng: ChaCha20Rng,
    }

    impl RandomGenerator {
        pub fn new(seed: u64) -> Self {
            Self { rng: get_rng(seed) }
        }

        pub fn gen_normal(&mut self, mean: f64, std: f64) -> f64 {
            let normal_dist = Normal::<f64>::new(mean, std).unwrap();
            normal_dist.sample(&mut self.rng)
        }

        pub fn gen_range<T: PrimInt>(&mut self, low: i64, high: i64, equal: bool) -> T {
            if equal {
                T::from(self.rng.gen_range(low..=high)).unwrap()
            } else {
                T::from(self.rng.gen_range(low..high)).unwrap()
            }
        }

        pub fn gen_bool(&mut self, prob: f64) -> bool {
            self.rng.gen_bool(prob.clamp(0.0, 1.0))
        }

        pub fn gen_permutation(&mut self, n: usize) -> Vec<usize> {
            let mut permutation: Vec<usize> = (1..=n).collect();
            permutation.shuffle(&mut self.rng);
            permutation
        }

        pub fn shuffle<T>(&mut self, v: &mut Vec<T>) {
            v.shuffle(&mut self.rng);
        }
    }

    fn get_rng(seed: u64) -> ChaCha20Rng {
        ChaCha20Rng::seed_from_u64(seed)
    }
}
```
