## C++
```c++
template<int size, int base>
struct Trie {
    vector<vector<int>> dat;
    vector<int> init, depth;

    Trie(): Trie(vector<string>{}) {}
    Trie(const vector<string>& v){
        init.resize(size, -1);
        depth.push_back(0);
        dat.push_back(init);

        for(string s : v) insert(s);
    }

    void insert(const string& s){
        int pos = 0;
        for(char c : s){
            int x = c - base;
            if(dat[pos][x] == -1){
                dat[pos][x] = dat.size();
                dat.push_back(init);
                depth.push_back(depth[pos]+1);
            }
            pos = dat[pos][x];
        }
    }

    bool exist(const string& s){
        int pos = 0;
        for(char& c : s){
            int x = c - base;
            if(dat[pos][x] == -1) return false;
            pos = dat[pos][x];
        }
        return true;
    }
};
```

## Rust
```rust
#[allow(dead_code)]
mod data_structure {
    use std::marker::PhantomData;

    /// Trie 木に乗せる型
    pub trait TrieNode {
        fn to_sequence(&self) -> Vec<usize>;
        fn revert(v: &[usize]) -> Self;
    }

    /// 英小文字のみの文字列を仮定
    impl TrieNode for String {
        fn to_sequence(&self) -> Vec<usize> {
            self.chars().map(|c| c as usize - 'a' as usize).collect()
        }
        fn revert(v: &[usize]) -> Self {
            v.iter()
                .map(|&x| (x + 'a' as usize) as u8 as char)
                .collect()
        }
    }

    impl TrieNode for u32 {
        fn to_sequence(&self) -> Vec<usize> {
            (0..32).map(|b| (self >> (31 - b) & 1) as usize).collect()
        }
        fn revert(v: &[usize]) -> Self {
            v.iter()
                .enumerate()
                .map(|(i, &x)| (x as u32) << (31 - i))
                .sum()
        }
    }

    impl TrieNode for u64 {
        fn to_sequence(&self) -> Vec<usize> {
            (0..64).map(|b| (self >> (63 - b) & 1) as usize).collect()
        }
        fn revert(v: &[usize]) -> Self {
            v.iter()
                .enumerate()
                .map(|(i, &x)| (x as u64) << (63 - i))
                .sum()
        }
    }

    pub struct Trie<T> {
        /// child[i][j] := i 番目の node の j 番目に対応する node の index
        child: Vec<Vec<Option<usize>>>,
        /// num_data[i] := i 番目の node とその子のデータの数
        num_data: Vec<u32>,
        /// cnt[i] := i 番目の node を終端とするデータの数
        cnt: Vec<u32>,
        size: usize,
        _marker: PhantomData<fn() -> T>,
    }

    impl Trie<u32> {
        pub fn new_with_u32() -> Self {
            Trie::new(2)
        }
    }

    impl Trie<u64> {
        pub fn new_with_u64() -> Self {
            Trie::new(2)
        }
    }

    impl Trie<String> {
        pub fn new_with_string() -> Self {
            Trie::new(26)
        }
    }

    impl<T: TrieNode> Trie<T> {
        pub fn new(size: usize) -> Self {
            let child = vec![vec![None; size]];
            let num_data = vec![0];
            let cnt = vec![0];

            Self {
                child,
                num_data,
                cnt,
                size,
                _marker: PhantomData,
            }
        }

        pub fn insert(&mut self, x: &T) {
            let mut pos = 0;
            let x = x.to_sequence();

            for xi in x {
                if self.child[pos][xi].is_none() {
                    self.insert_node(pos, xi);
                }

                self.num_data[pos] += 1;
                pos = self.child[pos][xi].unwrap();
            }
            self.num_data[pos] += 1;
            self.cnt[pos] += 1;
        }

        pub fn erase(&mut self, x: &T) -> bool {
            let x = x.to_sequence();
            if let Some(path) = self.get_path(&x) {
                let last_pos = *path.last().unwrap();
                if self.cnt[last_pos] == 0 {
                    return false;
                }

                self.cnt[last_pos] -= 1;

                for pos in path {
                    self.num_data[pos] -= 1;
                }
                return true;
            }
            false
        }

        /// xが存在するかどうか
        pub fn exists(&self, x: &T) -> bool {
            let x = x.to_sequence();
            if x.is_empty() {
                return false;
            }
            if let Some(path) = self.get_path(&x) {
                let last_node = *path.last().unwrap();
                return self.cnt[last_node] > 0;
            }
            false
        }

        /// k (0-indexed) 番目のデータを取得する
        pub fn get_kth(&self, k: usize) -> Option<T> {
            if self.num_data[0] < k as u32 {
                return None;
            }

            let mut cumsum = 0;
            let mut pos = 0;
            let mut v = vec![];

            loop {
                for i in 0..self.size {
                    if let Some(next) = self.child[pos][i] {
                        if cumsum + self.num_data[next] <= k as u32 {
                            cumsum += self.num_data[next];
                        } else {
                            pos = next;
                            v.push(i);
                            break;
                        }
                    }
                }

                if cumsum + self.cnt[pos] > k as u32 {
                    break;
                }
            }

            Some(T::revert(&v))
        }

        /// pos 番の node の index 番目に node を追加する
        fn insert_node(&mut self, pos: usize, index: usize) {
            self.child[pos][index] = Some(self.child.len());
            self.child.push(vec![None; self.size]);
            self.num_data.push(0);
            self.cnt.push(0);
        }

        /// x を辿るようなnodeの番号のリストを取得する
        /// 存在しない場合は None を返す
        fn get_path(&self, x: &[usize]) -> Option<Vec<usize>> {
            let mut path = vec![0];
            let mut pos = 0;

            for &xi in x {
                pos = self.child[pos][xi]?;
                path.push(pos);
            }

            Some(path)
        }
    }

    pub trait Unsigned {}
    impl Unsigned for u32 {}
    impl Unsigned for u64 {}

    impl<T: Unsigned + TrieNode> Trie<T> {
        fn xor(&self, x: &T, b: usize) -> Option<T> {
            if self.num_data[0] == 0 {
                return None;
            }
            let x = x.to_sequence();
            let mut pos = 0;
            let mut v = vec![];
            for xi in x {
                if self.child[pos][xi ^ b].is_some_and(|pos| self.num_data[pos] > 0) {
                    v.push(b);
                    pos = self.child[pos][xi ^ b].unwrap();
                } else if self.child[pos][xi ^ (1 - b)].is_some_and(|pos| self.num_data[pos] > 0) {
                    v.push(1 - b);
                    pos = self.child[pos][xi ^ (1 - b)].unwrap();
                } else {
                    unreachable!();
                }
            }
            Some(T::revert(&v))
        }
        pub fn xor_min(&self, x: &T) -> Option<T> {
            self.xor(x, 0)
        }
        pub fn xor_max(&self, x: &T) -> Option<T> {
            self.xor(x, 1)
        }
    }
}
```

## Example

- [ARC033 C - データ構造 (Rust)](https://atcoder.jp/contests/arc033/submissions/67029103)
- [ARC122 D - XOR Game (Rust)](https://atcoder.jp/contests/arc122/submissions/67028989)
