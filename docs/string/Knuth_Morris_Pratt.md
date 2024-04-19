## C++
```c++
template <class T>
std::vector<int> KMP(std::vector<T> &v) {
    int N = v.size(), j = -1;
    std::vector<int> A(N + 1);
    A[0] = -1;
    for (int i = 0; i < N; i++) {
        while (j >= 0 && v[i] != v[j])
            j = A[j];
        if (i < N - 1 and v[i + 1] == v[j])
            A[i + 1] = A[++j];
        else
            A[i + 1] = ++j;
    }
    return A;
}

std::vector<int> KMP(std::string &s) {
    int N = s.size();
    std::vector<char> v(N);
    for (int i = 0; i < N; i++)
        v[i] = s[i];
    return KMP(v);
}

```

## Reference

- [https://www.slideshare.net/hcpc_hokudai/ss-121539419](https://www.slideshare.net/hcpc_hokudai/ss-121539419)
