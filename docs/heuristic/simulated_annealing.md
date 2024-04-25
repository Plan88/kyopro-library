## Rust
```rust
fn main() {
    let input = problem_io::Input::new();
    let state = domain::State::new(input);
    let scheduler = simulated_annealing::Scheduler::new(T_START, T_FINAL, T_LIMIT);

    let best_state = simulated_annealing::simulated_annealing(state, scheduler);
}

pub mod problem_io {
    use proconio::input;

    pub struct Input {}

    impl Input {
        pub fn new() -> Self {
            todo!();
        }
    }
}

pub mod domain {
    use super::problem_io::Input;

    pub type Score = u128;

    #[derive(Clone)]
    pub struct State {
        score: Score,
        last_op: Vec<()>,
    }

    impl State {
        pub fn new(input: Input) -> Self {
            todo!();
        }

        pub fn move_to_neighbor(&mut self) {}

        pub fn eval(&self) -> Score {
            self.score
        }

        pub fn rollback(&mut self) {}
    }
}

pub mod simulated_annealing {
    use super::domain::{Score, State};
    use super::rand_generator::RandomGenerator;
    use super::utils::Timer;

    type Probability = f64;
    type Temperature = f64;
    type Time = f64;

    pub struct Scheduler {
        t_start: Temperature,
        t_final: Temperature,
        time_limit: Time,
        timer: Timer,
        rng: RandomGenerator,
        complete: bool,
    }

    impl Scheduler {
        pub fn new(t_start: Temperature, t_final: Temperature, time_limit: Time) -> Self {
            let timer = Timer::new();
            let rng = RandomGenerator::new(998244353);
            let complete: bool = false;

            Self {
                t_start,
                t_final,
                time_limit,
                timer,
                rng,
                complete,
            }
        }

        /// 時間計測開始
        pub fn start(&mut self) {
            self.timer.reset();
        }

        /// 経過時間取得
        pub fn get_elapsed_time(&mut self) -> Time {
            self.timer.get_epalsed_time()
        }

        /// time_elapsed秒経過したときの温度を計算する
        fn get_temperature(&self, time_elapsed: Time) -> Temperature {
            let power: f64 = time_elapsed / self.time_limit;
            let temp: Temperature = self.t_start * (self.t_final / self.t_start).powf(power);
            temp
        }

        /// 遷移確率を計算する
        fn probability(
            &self,
            temp: Temperature,
            cur_score: Score,
            new_score: Score,
        ) -> Probability {
            // 最小化
            Probability::exp((cur_score as Temperature - new_score as Temperature) / temp).min(1.0)
            // 最大化
            // Probability::exp((new_score as Temperature - cur_score as Temperature) / temp).min(1.0)
        }

        /// 遷移するかどうかの判定
        fn acceptance(&mut self, prob: Probability) -> bool {
            // dbg!(prob);
            self.rng.gen_bool(prob)
        }

        /// time_limit秒経過したかどうか
        fn is_complete(&self) -> bool {
            self.complete
        }
    }

    pub fn simulated_annealing(mut state: State, mut scheduler: Scheduler) -> State {
        let mut num_loops: usize = 0;
        let mut num_updates: usize = 0;
        let mut time_elapsed: Time;
        let mut temp: Temperature = scheduler.t_start;
        let mut best_score: Score = state.eval();
        let mut best_state: State = state.clone();

        scheduler.start();
        loop {
            if num_loops % 100 == 0 {
                time_elapsed = scheduler.get_elapsed_time();
                temp = scheduler.get_temperature(time_elapsed);

                if scheduler.is_complete() {
                    break;
                }
            }

            state.move_to_neighbor();
            let new_score: Score = state.eval();
            let prob: Probability = scheduler.probability(temp, best_score, new_score);
            if scheduler.acceptance(prob) {
                best_score = new_score;
                best_state = state.clone();
                num_updates += 1;
            } else {
                state.rollback();
            }
            num_loops += 1;
        }

        dbg!(num_loops);
        dbg!(num_updates);
        dbg!(best_score);
        best_state
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

pub mod utils {
    pub struct Timer {
        start_time: f64,
    }

    impl Timer {
        pub fn new() -> Self {
            let start_time = Self::get_current_time();
            Self { start_time }
        }

        /// 現在の時刻[sec]を取得する
        fn get_current_time() -> f64 {
            let t = std::time::SystemTime::now()
                .duration_since(std::time::UNIX_EPOCH)
                .unwrap(); // 1970-01-01 00:00:00 からの時間が取得できる (単位不明)
            t.as_secs() as f64 + t.subsec_nanos() as f64 * 1e-9
        }

        /// 現在の経過時間[sec]を取得する
        pub fn get_epalsed_time(&self) -> f64 {
            let current_time = Self::get_current_time();
            current_time - self.start_time
        }

        /// timer をリセットする
        pub fn reset(&mut self) {
            self.start_time = Self::get_current_time();
        }
    }
}
```
