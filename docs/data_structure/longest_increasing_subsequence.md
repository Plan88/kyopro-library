## C++

### Narrow ver
```c++
template<class T>
std::vector<int> longest_increasing_subsequence(std::vector<T> &v, T inf) {
    if(v.empty()) {
        return {};
    }

    int N = v.size();
    std::vector<T> dp(N, inf);
    std::vector<T> index(N, -1), prev(N, -1);

    auto lb = [&](T x) {
        return std::lower_bound(dp.begin(), dp.end(), x) - dp.begin();
    };

    for(int i = 0; i < N; i++) {
        int t = lb(v[i]);
        dp[t] = v[i];
        index[t] = i;
        if(t > 0) {
            prev[i] = index[t-1];
        }
    }

    int length = lb(inf);
    std::vector<int> subsequence(length, -1);
    int pos = index[length - 1];
    for(int i = length - 1; i >= 0; i--) {
        subsequence[i] = pos;
        pos = prev[pos];
    }
    return subsequence;
}
```

### Broad ver
```c++
template<class T>
std::vector<int> longest_increasing_subsequence(std::vector<T> &v, T inf) {
    if(v.empty()) {
        return {};
    }

    int N = v.size();
    std::vector<T> dp(N, inf);
    std::vector<T> index(N, -1), prev(N, -1);

    auto ub = [&](T x) {
        return std::upper_bound(dp.begin(), dp.end(), x) - dp.begin();
    };

    for(int i = 0; i < N; i++) {
        int t = ub(v[i]);
        dp[t] = v[i];
        index[t] = i;
        if(t > 0) {
            prev[i] = index[t-1];
        }
    }

    int length = std::lower_bound(dp.begin(), dp.end(), inf) - dp.begin();
    std::vector<int> subsequence(length, -1);
    int pos = index[length - 1];
    for(int i = length - 1; i >= 0; i--) {
        subsequence[i] = pos;
        pos = prev[pos];
    }
    return subsequence;
}
```

## Examples

- [ABC369 F - Gather Coins (C++)](https://atcoder.jp/contests/abc369/submissions/57342321)
