## Rust
```rust
mod math {
    use std::ops;

    /// Lagrange interpolation
    pub struct Interpolation<T> {
        /// Degree of polynomial
        n: usize,
        /// Coefficients
        coeff: Vec<T>,
    }

    impl<T> Interpolation<T>
    where
        T: Copy
            + ops::Neg<Output = T>
            + ops::Add<Output = T>
            + ops::Sub<Output = T>
            + ops::Mul<Output = T>
            + ops::Div<Output = T>
            + num::Zero
            + num::One
            + num::FromPrimitive,
    {
        /// Given n+1 points (0, y0), (1, y1), ..., (n, yn), construct the interpolation polynomial
        pub fn new(y: &[T]) -> Self {
            let n = y.len() - 1;
            let mut coeff = Vec::with_capacity(n + 1);
            let mut fact = T::one();
            for i in 1..=n {
                fact = fact * T::from_usize(i).unwrap();
            }
            fact = T::one() / fact;
            let mut inv = vec![fact; n + 1];
            for i in (0..n).rev() {
                inv[i] = inv[i + 1] * T::from_usize(i + 1).unwrap();
            }
            let mut neg = T::one();
            if (n % 2) == 1 {
                neg = -neg;
            }

            for i in 0..=n {
                let left = inv[i];
                let right = inv[n - i] * neg;
                coeff.push(left * right * y[i]);
                neg = -neg;
            }
            Self { n, coeff }
        }

        /// Evaluate the polynomial at x
        pub fn eval(&self, x: T) -> T {
            let mut left = T::one();
            let mut right = vec![T::one(); self.n + 1];
            for i in (0..self.n).rev() {
                right[i] = right[i + 1] * (x - T::from_usize(i + 1).unwrap());
            }
            let mut ans = T::zero();
            for i in 0..=self.n {
                ans = ans + self.coeff[i] * left * right[i];
                left = left * (x - T::from_usize(i).unwrap());
            }
            ans
        }
    }
}
```
