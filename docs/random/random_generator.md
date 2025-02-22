## Rust
```rust
pub mod rand_generator {
    use rand::{seq::SliceRandom, Rng};
    use rand_chacha::{rand_core::SeedableRng, ChaCha20Rng};
    use rand_distr::{
        uniform::{SampleRange, SampleUniform},
        Distribution, Normal,
    };

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

        pub fn gen_range<T: SampleUniform, U: SampleRange<T>>(&mut self, range: U) -> T {
            self.rng.gen_range(range)
        }

        pub fn gen_bool(&mut self, prob: f64) -> bool {
            self.rng.gen_bool(prob.clamp(0.0, 1.0))
        }

        pub fn gen_permutation(&mut self, n: usize) -> Vec<usize> {
            let mut permutation: Vec<usize> = (0..n).collect();
            permutation.shuffle(&mut self.rng);
            permutation
        }

        pub fn shuffle<T>(&mut self, v: &mut [T]) {
            v.shuffle(&mut self.rng);
        }
    }

    fn get_rng(seed: u64) -> ChaCha20Rng {
        ChaCha20Rng::seed_from_u64(seed)
    }
}
```
