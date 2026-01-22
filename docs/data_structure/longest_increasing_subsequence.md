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
        if(v[i] < dp[t]) {
            dp[t] = v[i];
            index[t] = i;
        }
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
        if(v[i] < dp[t]) {
            dp[t] = v[i];
            index[t] = i;
        }
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

## Rust

### Narrow ver
```Rust
/// Returns the indices of one of the longest increasing subsequences in `a`.
/// i.e., for the returned vector `res`, `a[res[0]] < a[res[1]] < ... < a[res[res.len() - 1]]`.
fn longest_increasing_subsequence<T>(a: &[T]) -> Vec<usize>
where
    T: Ord,
{
    if a.is_empty() {
        return vec![];
    }

    let n = a.len();
    let mut dp: Vec<&T> = Vec::with_capacity(n);
    let mut index = vec![n; n];
    let mut prev_index = vec![None; n];

    for (i, ai) in a.iter().enumerate() {
        let pos = dp.partition_point(|x| *x < ai);
        if pos == dp.len() {
            dp.push(ai);
        } else {
            dp[pos] = ai;
        }
        index[pos] = i;
        if pos > 0 {
            prev_index[i] = Some(index[pos - 1]);
        }
    }

    let len = dp.len();
    let mut seq = Vec::with_capacity(len);
    let mut now = index[len - 1];
    while let Some(prev) = prev_index[now] {
        seq.push(now);
        now = prev;
    }
    seq.push(now);
    seq.reverse();
    seq
}
```


## Examples

- [ABC369 F - Gather Coins (C++)](https://atcoder.jp/contests/abc369/submissions/57342321)
