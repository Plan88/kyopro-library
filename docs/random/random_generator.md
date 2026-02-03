## Rust
```rust
mod rand_generator {
    use rand::{Rng, seq::SliceRandom};
    use rand_chacha::{ChaCha20Rng, rand_core::SeedableRng};
    use rand_distr::{
        Distribution, Normal,
        uniform::{SampleRange, SampleUniform},
    };

    pub struct RandomGenerator {
        rng: ChaCha20Rng,
    }

    impl RandomGenerator {
        pub fn new() -> Self {
            Self {
                rng: ChaCha20Rng::from_os_rng(),
            }
        }

        pub fn from_seed(seed: u64) -> Self {
            Self {
                rng: ChaCha20Rng::seed_from_u64(seed),
            }
        }

        pub fn random_normal(&mut self, mean: f64, std: f64) -> f64 {
            let normal_dist = Normal::<f64>::new(mean, std).unwrap();
            normal_dist.sample(&mut self.rng)
        }

        pub fn random_range<T: SampleUniform, U: SampleRange<T>>(&mut self, range: U) -> T {
            self.rng.random_range(range)
        }

        pub fn random_bool(&mut self, prob: f64) -> bool {
            self.rng.random_bool(prob.clamp(0.0, 1.0))
        }

        pub fn random_permutation(&mut self, n: usize) -> Vec<usize> {
            let mut permutation: Vec<usize> = (0..n).collect();
            permutation.shuffle(&mut self.rng);
            permutation
        }

        pub fn shuffle<T>(&mut self, v: &mut [T]) {
            v.shuffle(&mut self.rng);
        }
    }
}
```
