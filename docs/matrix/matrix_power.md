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

## Example

- [ABC009 D - 漸化式 (C++)](https://atcoder.jp/contests/abc009/submissions/53032473)
- [ABC236 G - Good Vertices (C++)](https://atcoder.jp/contests/abc236/submissions/53032371)
- [Educational DP Contest R - Walk (C++)](https://atcoder.jp/contests/dp/submissions/53031726)
