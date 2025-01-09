長さ $N$ の文字列 $S$ について、各 $i (i=1, ..., N)$ に対して、$S$ と $S_i$ 以降の共通接頭辞の長さを求める。

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

## Rust
```rust
pub mod string {
    pub fn z_algorithm<T: PartialEq>(v: &Vec<T>) -> Vec<usize> {
        let mut c = 0;
        let n = v.len();
        let mut z = vec![0; n];

        for i in 1..n {
            let l = i - c;
            if i + z[l] < c + z[c] {
                z[i] = z[l];
            } else {
                let mut j = (c + z[c]).saturating_sub(i);
                while i + j < n && v[j] == v[i + j] {
                    j += 1;
                }
                z[i] = j;
                c = i;
            }
        }
        z[0] = n;
        z
    }
}
```

## Example
- [ABC343 G - Compress Strings (C++)](https://atcoder.jp/contests/abc343/submissions/51251836)


## Reference

- [https://sen-comp.hatenablog.com/entry/2020/01/16/174230](https://sen-comp.hatenablog.com/entry/2020/01/16/174230)
