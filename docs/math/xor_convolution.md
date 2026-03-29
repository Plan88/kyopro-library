## Rust
```rust
mod math {
    use num_traits::Zero;
    type Int = u32;

    pub fn xor_convolution(a: &Vec<Int>, b: &Vec<Int>) -> Vec<Int> {
        let n = 1 << next_power_of_2(a.len().max(b.len()));
        let mut fa = a.clone();
        let mut fb = b.clone();
        fa.resize(n, Int::zero());
        fb.resize(n, Int::zero());

        fast_walsh_hadamard_transform(&mut fa);
        fast_walsh_hadamard_transform(&mut fb);
        for i in 0..n {
            fa[i] *= fb[i];
        }
        fast_walsh_hadamard_transform(&mut fa);
        for x in &mut fa {
            *x /= n as Int; // Int = modint の場合は逆元を掛ける
        }
        fa
    }

    pub fn fast_walsh_hadamard_transform(x: &mut Vec<Int>) {
        let k = x.len().trailing_zeros() as usize;
        for p in 0..k {
            let step = 1 << p;
            for i in (0..x.len()).step_by(step * 2) {
                for j in i..i + step {
                    let u = x[j];
                    let v = x[j + step];
                    x[j] = u + v;
                    x[j + step] = u - v;
                }
            }
        }
    }

    ///  x <= 2^k を満たす最小の非負整数 k を求める。
    fn next_power_of_2(mut x: usize) -> usize {
        if x == 0 {
            return 0;
        }
        let mut k = 0;
        while (1 << k) < x {
            k += 1;
        }
        k
    }
}
```
