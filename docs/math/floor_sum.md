calculate

$$\sum_{i = 0}^{n - 1} \left\lfloor \frac{a \times i + b}{m} \right\rfloor.$$

return $\bmod 2^{\mathrm{64}}$ if overflow

**constraints**

- $0 \leq n \lt 2^{32}$
- $1 \leq m \lt 2^{32}$

**complexity**

- $O(\log m)$

## C++
```c++
long long floor_sum(long long n, long long m, long long a, long long b) {
    long long ans = 0;
    if (a >= m) {
        ans += (n - 1) * n * (a / m) / 2;
        a %= m;
    }
    if (b >= m) {
        ans += n * (b / m);
        b %= m;
    }

    long long y_max = (a * n + b) / m, x_max = (y_max * m - b);
    if (y_max == 0) return ans;
    ans += (n - (x_max + a - 1) / a) * y_max;
    ans += floor_sum(y_max, a, m, (a - x_max % a) % a);
    return ans;
}
```

## Example

- [ARC111 E - Simple Math 3 (C++)](https://atcoder.jp/contests/arc111/submissions/28830654)
