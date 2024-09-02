## C++
```c++
template<class T>
std::vector<std::pair<T, int>> run_length_encode(std::vector<T> v) {
    std::vector<std::pair<T, int>> encoded;
    int n = v.size(), left = 0;

    while (left < n) {
        int right = left + 1;
        while (right < n and v[left] == v[right]) right++;
        encoded.emplace_back(v[left], right - left);
        left = right;
    }

    return encoded;
}
```

## Rust
```rust
mod compression {
    pub fn run_length_encode<T: Eq + Clone>(v: &Vec<T>) -> Vec<(T, usize)> {
        let mut encoded = vec![];
        let n = v.len();
        let mut left = 0;

        while left < n {
            let mut right = left + 1;
            while right < n && v[left] == v[right] {
                right += 1;
            }
            encoded.push((v[left].clone(), right - left));
            left = right;
        }

        encoded
    }
}
```
