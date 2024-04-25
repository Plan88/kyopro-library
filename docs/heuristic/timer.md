## Rust
```rust
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
