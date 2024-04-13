長さ $N$ の文字列 $S$ に対して、$S_i$ を中心とした回文の長さの半径を求める。
長さが偶数の回文は、各文字の間に特別な文字を挿入することで求められる

**complexity**

- $O(\log N)$

## C++
```c++
template <class T>
std::vector<int> manachar(std::vector<T> &v) {
    // c := 現時点で右端が最大になるような回文の中心
    int c = 0, n = int(v.size());
    std::vector<int> r(n, 1);

    for (int i = 0; i < n; i++) {
        int j = c - (i - c); // c を中心として, i と対称となる位置
        if (i + r[j] < c + r[c]) {
            r[i] = r[j];
        } else {
            int len = c + r[c] - i; // c の右端と i の距離 = 現時点で確定してる回文の長さ
            while (i - len >= 0 and i + len < n and v[i - len] == v[i + len])
                len++;
            r[i] = len;
            c = i;
        }
    }

    return r;
}

std::vector<int> manachar(std::string &s) {
    int n = s.size();
    std::vector<char> v(n);
    for (int i = 0; i < n; i++)
        v[i] = s[i];
    return manachar(v);
}
```

## Reference

- [https://www.slideshare.net/hcpc_hokudai/ss-121539419](https://www.slideshare.net/hcpc_hokudai/ss-121539419)
