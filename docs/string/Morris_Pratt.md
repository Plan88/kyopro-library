長さ $N$ の文字列 $S$ に対して、数列 $A_i (i=1, \ldots, N+1)$ を求める

$$A_i = S_{1, \ldots, i-1}\text{の接頭辞と接尾辞が最大何文字一致しているか}$$

**complaxity**

- $O(N)$

## C++
```c++
template <class T>
std::vector<int> MP(std::vector<T> &v) {
    int N = v.size(), j = -1;
    std::vector<int> A(N + 1);
    A[0] = -1;
    for (int i = 0; i < N; i++) {
        while (j >= 0 && v[i] != v[j])
            j = A[j];
        A[i + 1] = ++j;
    }
    return A;
}

std::vector<int> MP(std::string &s) {
    int N = s.size();
    std::vector<char> v(N);
    for (int i = 0; i < N; i++)
        v[i] = s[i];
    return MP(v);
}
```

## Reference

- [https://www.slideshare.net/hcpc_hokudai/ss-121539419](https://www.slideshare.net/hcpc_hokudai/ss-121539419)
