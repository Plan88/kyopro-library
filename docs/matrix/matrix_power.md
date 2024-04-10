calculate

$$A^k\boldsymbol{x},\ A \in \mathbb{R}^{N\times N},\ \boldsymbol{x} \in \mathbb{R}^{N \times 1},\ k \in \mathbb{N}$$

**constraints**

- $0 \leq k \leq 10^{18}$

**complexity**

- $\mathrm{O}(N^3\log k)$

## C++
```c++
template<class T>
std::vector<std::vector<T>> matdot(std::vector<std::vector<T>>& A, std::vector<std::vector<T>>& B){
    int n = A.size(), m = B.size(), l = B[0].size();
    std::vector<std::vector<T>> ret(n, std::vector<T>(l,0));

    for(int i=0; i<n; i++)
        for(int j=0; j<l; j++)
            for(int k=0; k<m; k++)
                ret[i][j] += A[i][k]*B[k][j];

    return ret;
}

template<class T>
void matpow(std::vector<std::vector<T>>& dp, std::vector<std::vector<T>>& mat, long long k){
    while(k){
        if(k & 1) dp = matdot(mat, dp);
        mat = matdot(mat, mat);
        k >>= 1;
    }
}
```
