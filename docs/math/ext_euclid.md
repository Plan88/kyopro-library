## C++
```c++
// ax + by = gcd(a, b) を満たす (x, y)
std::pair<long long, long long> ext_euclid(long long a, long long b) {
    if (b == 0)
        return make_pair(1, 0);
    long long x, y;
    tie(y, x) = ext_euclid(b, a % b);
    y -= a / b * x;
    return make_pair(x, y);
}}
```
