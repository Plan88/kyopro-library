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
mod trie {
    use num_traits::PrimInt;

    pub struct Trie<T: PrimInt> {
        child: Vec<Vec<Option<usize>>>,
        num_data: Vec<u32>,
        num_eos: Vec<u32>,
        size: usize,
        base: T,
    }

    impl<T: PrimInt> Trie<T> {
        pub fn new(size: usize, base: T) -> Self {
            let child = vec![vec![None; size]];
            let num_data = vec![0];
            let num_eos = vec![0];

            Self {
                child,
                num_data,
                num_eos,
                size,
                base,
            }
        }

        pub fn insert(&mut self, x: &Vec<T>) {
            let mut pos = 0;

            for &xi in x.iter() {
                let index = (xi - self.base).to_usize().unwrap();

                if self.child[pos][index].is_none() {
                    self.insert_node(pos, index);
                }

                self.num_data[pos] += 1;
                pos = self.child[pos][index].unwrap();
            }
            self.num_data[pos] += 1;
            self.num_eos[pos] += 1;
        }

        pub fn erase(&mut self, x: &Vec<T>) -> bool {
            if let Some(path) = self.get_path(x) {
                let last_pos = *path.last().unwrap();
                if self.num_eos[last_pos] == 0 {
                    return false;
                }

                self.num_eos[last_pos] -= 1;

                for pos in path.into_iter() {
                    self.num_data[pos] -= 1;
                }
                return true;
            }
            false
        }

        /// xが存在するかどうか
        pub fn exists(&self, x: &Vec<T>) -> bool {
            if let Some(path) = self.get_path(x) {
                let last_pos = *path.last().unwrap();
                return self.num_eos[last_pos] > 0;
            }
            false
        }

        pub fn get_kth(&self, k: usize) -> Option<Vec<T>> {
            if self.num_data[0] < k as u32 {
                return None;
            }

            let mut cumsum = 0;
            let mut pos = 0;
            let mut val = vec![];

            loop {
                for i in 0..self.size {
                    if let Some(next) = self.child[pos][i] {
                        if cumsum + self.num_data[next] < k as u32 {
                            cumsum += self.num_data[next];
                        } else {
                            pos = next;
                            val.push(self.base + T::from(i).unwrap());
                            break;
                        }
                    }
                }

                if cumsum + self.num_eos[pos] >= k as u32 {
                    break;
                }
            }

            Some(val)
        }

        fn get_node(&self) -> Vec<Option<usize>> {
            let child = vec![None; self.size];
            child
        }

        fn get_next_index(&self) -> usize {
            self.child.len()
        }

        /// pos番のnodeのindex番目にnodeを追加する
        fn insert_node(&mut self, pos: usize, index: usize) {
            let new_index = self.get_next_index();
            self.child[pos][index] = Some(new_index);

            let child = self.get_node();
            self.child.push(child);
            self.num_data.push(0);
            self.num_eos.push(0);
        }

        /// xを辿るようなnodeの番号のリストを取得する
        /// 存在しない場合はNoneを返す
        fn get_path(&self, x: &Vec<T>) -> Option<Vec<usize>> {
            let mut path = vec![0];
            let mut pos = 0;

            for &xi in x.iter() {
                let index = (xi - self.base).to_usize().unwrap();
                if self.child[pos][index].is_none() {
                    return None;
                }
                pos = self.child[pos][index].unwrap();
                path.push(pos);
            }

            Some(path)
        }
    }
}
```

## Example

- [ARC033 C - データ構造 (Rust)](https://atcoder.jp/contests/arc033/submissions/51472880)
