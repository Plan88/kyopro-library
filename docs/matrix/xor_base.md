## C++
```c++
std::vector<long long> calc_base(std::vector<long long> &v) {
    std::vector<long long> base;

    for (long long x : v) {
        for (long long b : base)
            x = std::min(x, b ^ x);
        if (x > 0)
            base.push_back(x);
    }
	
	return base;
}
```
