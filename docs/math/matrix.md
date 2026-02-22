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

mod math {
    use std::ops::{Add, Index, IndexMut, Mul};

    #[derive(Debug, Clone, PartialEq)]
    pub enum Tensor<T> {
        Scalar(T),
        Vector(Vec<T>),
        Matrix(Vec<Vec<T>>),
    }

    impl<T> Index<usize> for Tensor<T> {
        type Output = Vec<T>;

        fn index(&self, index: usize) -> &Self::Output {
            match self {
                Self::Matrix(m) => &m[index],
                _ => panic!("indexing with usize is only supported for Matrix"),
            }
        }
    }

    impl<T> IndexMut<usize> for Tensor<T> {
        fn index_mut(&mut self, index: usize) -> &mut Self::Output {
            match self {
                Self::Matrix(m) => &mut m[index],
                _ => panic!("indexing with usize is only supported for Matrix"),
            }
        }
    }

    impl<T> Tensor<T>
    where
        T: Copy + num_traits::Zero + num_traits::One + Add<Output = T> + Mul<Output = T>,
    {
        pub fn mul(&self, rhs: &Self) -> Result<Self, String> {
            match (self, rhs) {
                (Self::Scalar(a), Self::Scalar(b)) => Ok(Self::Scalar(*a * *b)),
                (Self::Scalar(a), Self::Vector(v)) => {
                    Ok(Self::Vector(v.iter().map(|x| *a * *x).collect()))
                }
                (Self::Vector(v), Self::Scalar(a)) => {
                    Ok(Self::Vector(v.iter().map(|x| *x * *a).collect()))
                }
                (Self::Scalar(a), Self::Matrix(m)) => Ok(Self::Matrix(
                    m.iter()
                        .map(|row| row.iter().map(|x| *a * *x).collect())
                        .collect(),
                )),
                (Self::Matrix(m), Self::Scalar(a)) => Ok(Self::Matrix(
                    m.iter()
                        .map(|row| row.iter().map(|x| *x * *a).collect())
                        .collect(),
                )),
                (Self::Vector(a), Self::Vector(b)) => {
                    if a.len() != b.len() {
                        return Err(format!(
                            "vector length mismatch: left={}, right={}",
                            a.len(),
                            b.len()
                        ));
                    }
                    let mut sum = T::zero();
                    for i in 0..a.len() {
                        sum = sum + a[i] * b[i];
                    }
                    Ok(Self::Scalar(sum))
                }
                (Self::Vector(v), Self::Matrix(m)) => {
                    let (rows, cols) = Self::shape(m)?;
                    if v.len() != rows {
                        return Err(format!(
                            "dimension mismatch for Vector * Matrix: vector={}, matrix_rows={}",
                            v.len(),
                            rows
                        ));
                    }
                    let mut res = vec![T::zero(); cols];
                    for j in 0..cols {
                        let mut sum = T::zero();
                        for i in 0..rows {
                            sum = sum + v[i] * m[i][j];
                        }
                        res[j] = sum;
                    }
                    Ok(Self::Vector(res))
                }
                (Self::Matrix(m), Self::Vector(v)) => {
                    let (rows, cols) = Self::shape(m)?;
                    if cols != v.len() {
                        return Err(format!(
                            "dimension mismatch for Matrix * Vector: matrix_cols={}, vector={}",
                            cols,
                            v.len()
                        ));
                    }
                    let mut res = vec![T::zero(); rows];
                    for i in 0..rows {
                        let mut sum = T::zero();
                        for j in 0..cols {
                            sum = sum + m[i][j] * v[j];
                        }
                        res[i] = sum;
                    }
                    Ok(Self::Vector(res))
                }
                (Self::Matrix(a), Self::Matrix(b)) => Ok(Self::Matrix(Self::mul_matrix(a, b)?)),
            }
        }

        pub fn pow(&self, mut exp: u64) -> Result<Self, String> {
            let base = match self {
                Self::Matrix(m) => m,
                _ => return Err("pow is supported only for Matrix".to_string()),
            };

            let (n, m) = Self::shape(base)?;
            if n != m {
                return Err(format!(
                    "pow requires a square matrix: got {}x{} matrix",
                    n, m
                ));
            }

            let mut result = Self::identity(n);
            let mut cur = Self::Matrix(base.clone());

            while exp > 0 {
                if exp & 1 == 1 {
                    result = result.mul(&cur)?;
                }
                cur = cur.mul(&cur)?;
                exp >>= 1;
            }

            Ok(result)
        }

        pub fn identity(n: usize) -> Self {
            let mut mat = vec![vec![T::zero(); n]; n];
            for (i, row) in mat.iter_mut().enumerate() {
                row[i] = T::one();
            }
            Self::Matrix(mat)
        }

        fn shape(m: &[Vec<T>]) -> Result<(usize, usize), String> {
            if m.is_empty() {
                return Ok((0, 0));
            }
            let cols = m[0].len();
            for (i, row) in m.iter().enumerate() {
                if row.len() != cols {
                    return Err(format!(
                        "matrix is not rectangular: row 0 has {}, row {} has {}",
                        cols,
                        i,
                        row.len()
                    ));
                }
            }
            Ok((m.len(), cols))
        }

        fn mul_matrix(a: &[Vec<T>], b: &[Vec<T>]) -> Result<Vec<Vec<T>>, String> {
            let (ra, ca) = Self::shape(a)?;
            let (rb, cb) = Self::shape(b)?;
            if ca != rb {
                return Err(format!(
                    "dimension mismatch for Matrix * Matrix: left={}x{}, right={}x{}",
                    ra, ca, rb, cb
                ));
            }

            let mut res = vec![vec![T::zero(); cb]; ra];
            for i in 0..ra {
                for k in 0..ca {
                    for j in 0..cb {
                        res[i][j] = res[i][j] + a[i][k] * b[k][j];
                    }
                }
            }
            Ok(res)
        }
    }
}
```
