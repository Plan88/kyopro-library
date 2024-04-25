## C++
```c++
template <class T>
std::vector<int> z_algorithm(std::vector<T> &s) {
    int c = 0, n = s.size();
    std::vector<int> z(n, 0);
    for (int i = 1; i < n; i++) {
        int l = i - c;
        if (i + z[l] < c + z[c]) {
            z[i] = z[l];
        } else {
            int j = std::max(0, c + z[c] - i);
            while (i + j < n and s[j] == s[i + j])
                j++;
            z[i] = j;
            c = i;
        }
    }
    z[0] = n;
    return z;
}

std::vector<int> z_algorithm(std::string &s) {
    int n = s.size();
    std::vector<char> v(n);
    for (int i = 0; i < n; i++)
        v[i] = s[i];
    return z_algorithm(v);
}
```

## Example
- [ABC343 G - Compress Strings (C++)](https://atcoder.jp/contests/abc343/submissions/51251836)

## Reference

- [https://sen-comp.hatenablog.com/entry/2020/01/16/174230](https://sen-comp.hatenablog.com/entry/2020/01/16/174230)
