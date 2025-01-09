calculate

$$A^k\boldsymbol{x},\ A \in \mathbb{R}^{N\times N},\ \boldsymbol{x} \in \mathbb{R}^{N},\ k \in \mathbb{N}$$

**constraints**

- $0 \leq k \leq 10^{18}$

**complexity**

- $\mathrm{O}(N^3\log k)$

## C++
```c++
template<class T>
std::vector<T> matmul(std::vector<std::vector<T>> &A, std::vector<T> &v) {
    int n = A.size(), m = v.size();
    std::vector<T> ret(n, 0);

    for(int i = 0; i < n; i++)
        for(int j = 0; j < m; j++)
            ret[i] += A[i][j] + v[j];

    return ret;
}

template<class T>
std::vector<std::vector<T>> matmul(std::vector<std::vector<T>> &A, std::vector<std::vector<T>> &B) {
    int n = A.size(), m = B.size(), l = B[0].size();
    std::vector<std::vector<T>> ret(n, std::vector<T>(l,0));

    for(int i = 0; i < n; i++)
        for(int k = 0; k < m; k++)
            for(int j = 0; j < l; j++)
                ret[i][j] += A[i][k] * B[k][j];

    return ret;
}

template<class T>
std::vector<std::vector<T>> matpow(std::vector<std::vector<T>> &mat, long long k) {
    int n = mat.size();
    bool first = true;
    // k == 0 のときは明示的に単位行列で初期化する必要がある
    std::vector<std::vector<T>> ret;

    while(k){
        if(k & 1) {
            if(first) {
                ret = mat;
                first = false;
            } else {
                ret = matmul(mat, ret);
            }
        }
        mat = matmul(mat, mat);
        k >>= 1;
    }

    return ret;
}
```

## Rust
```rust
pub mod matrix {
    use std::ops::{AddAssign, Mul};

    use num::Integer;

    pub trait MatrixMul
    where
        Self: Copy + num::One + num::Zero + AddAssign<Self> + Mul<Self>,
    {
    }

    pub fn matrix_pow<T: MatrixMul>(matrix: Vec<Vec<T>>, mut power: u64) -> Vec<Vec<T>> {
        let n = matrix.len();
        assert!(matrix.iter().all(|row| row.len() == n));

        let mut pow2 = matrix;
        let mut powered = vec![vec![T::zero(); n]; n];
        for i in 0..n {
            powered[i][i] = T::one();
        }

        while power > 0 {
            if power.is_odd() {
                powered = matrix_mul(&pow2, &powered);
            }
            pow2 = matrix_mul(&pow2, &pow2);
            power >>= 1;
        }

        powered
    }

    pub fn matrix_mul<T: MatrixMul>(left: &Vec<Vec<T>>, right: &Vec<Vec<T>>) -> Vec<Vec<T>> {
        let rows = left.len();
        let common = right.len();
        let cols = right[0].len();
        let mut mul = vec![vec![T::zero(); cols]; rows];

        for i in 0..rows {
            for k in 0..common {
                for j in 0..cols {
                    mul[i][j] += left[i][k] * right[k][j];
                }
            }
        }

        mul
    }

    pub fn matrix_vector_mul<T: MatrixMul>(matrix: &Vec<Vec<T>>, vector: &Vec<T>) -> Vec<T> {
        let rows = matrix.len();
        let cols = matrix[0].len();
        let mut mul = vec![T::zero(); cols];

        for i in 0..rows {
            for j in 0..cols {
                mul[i] += matrix[i][j] * vector[j];
            }
        }

        mul
    }
}
```

## Example

- [ABC009 D - 漸化式 (C++)](https://atcoder.jp/contests/abc009/submissions/53032473)
- [ABC236 G - Good Vertices (C++)](https://atcoder.jp/contests/abc236/submissions/53032371)
- [Educational DP Contest R - Walk (C++)](https://atcoder.jp/contests/dp/submissions/53031726)
- [ABC256 G - Black and White Stones](https://atcoder.jp/contests/abc256/submissions/61482570)
