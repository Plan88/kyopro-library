## Rust
```rust
use proconio::input;
use rand::Rng;
use rand_chacha::ChaCha20Rng;
use rand_mod::get_rng;

fn main() {
    let input = Input::new();
    let state = State::new(input);
    let scheduler = Scheduler::new(1_000_000_000_000.0, 100_000_000.0, 1.9);

    let best_state = simulated_annealing(state, scheduler);
}

type Probability = f64;
type Temperature = f64;
type Time = f64;
type Score = u128;

struct Input {}

impl Input {
    fn new() -> Self {
        input! {}

        Self {}
    }
}

#[derive(Clone)]
struct State {
    score: Score,
    last_op: Vec<()>,
    rng: ChaCha20Rng,
}

impl State {
    fn new(input: Input) -> Self {}
    fn move_to_neighbor(&mut self) {}

    fn eval(&self) -> Score {
        self.score
    }

    fn undo(&mut self) {}
}

struct Scheduler {
    t_start: Temperature,
    t_final: Temperature,
    time_limit: Time,
    time_current: Time,
    rng: ChaCha20Rng,
    complete: bool,
}

impl Scheduler {
    fn new(t_start: Temperature, t_final: Temperature, time_limit: Time) -> Self {
        let time_current: Time = -1.0;
        let rng = rand_mod::get_rng(998244353);
        let complete: bool = false;

        Scheduler {
            t_start,
            t_final,
            time_limit,
            time_current,
            rng,
            complete,
        }
    }

    fn get_elapsed_time(&mut self) -> Time {
        let t = std::time::SystemTime::now()
            .duration_since(std::time::UNIX_EPOCH)
            .unwrap();
        let ms: Time = t.as_secs() as f64 + t.subsec_nanos() as f64 * 1e-9;

        if self.time_current < 0.0 {
            self.time_current = ms;
        }
        let t_elapsed: Time = ms - self.time_current;

        if t_elapsed > self.time_limit {
            self.complete = true;
        }

        t_elapsed
    }

    /// time_elapsed秒経過したときの温度を計算する
    fn get_temperature(&self, time_elapsed: Time) -> Temperature {
        let power: f64 = time_elapsed / self.time_limit;
        let temp: Temperature = self.t_start * (self.t_final / self.t_start).powf(power);
        temp
    }

    /// 遷移確率を計算する
    fn probability(&self, temp: Temperature, cur_score: Score, new_score: Score) -> Probability {
        // 最小化
        Probability::exp((cur_score as Temperature - new_score as Temperature) / temp).min(1.0)
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

fn simulated_annealing(mut state: State, mut scheduler: Scheduler) -> State {
    let mut num_loops: usize = 0;
    let mut num_updates: usize = 0;
    let mut time_elapsed: Time;
    let mut temp: Temperature = scheduler.t_start;
    let mut best_score: Score = state.eval();
    let mut best_state: State = state.clone();
    dbg!(best_score);

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
            state.undo();
        }
        num_loops += 1;
    }

    dbg!(num_loops);
    dbg!(num_updates);
    dbg!(best_score);
    best_state
}

mod rand_mod {
    use rand_chacha::ChaCha20Rng;
    use rand_core::SeedableRng;

    pub fn get_rng(seed: u64) -> ChaCha20Rng {
        ChaCha20Rng::seed_from_u64(seed)
    }
}
```
