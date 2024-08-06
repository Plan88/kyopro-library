## C++
```c++
template<class T>
struct CartesianTree {
  public:
    CartesianTree(std::vector<T> A): val(A) {
        build();
    }

    int get_par_index(int i) {
        return par[i];
    }
    int get_left_index(int i) {
        return left[i];
    }
    int get_right_index(int i) {
        return right[i];
    }
    T get_par_value(int i) {
        return par[i] == -1 ? -1 : val[par[i]];
    }
    T get_left_value(int i) {
        return left[i] == -1 ? -1 : val[left[i]];
    }
    T get_right_value(int i) {
        return right[i] == -1 ? -1 : val[right[i]];
    }

  private:
    std::vector<T> val;
    std::vector<int> left, right, par;

    void build() {
        int N = val.size();
        left.resize(N, -1);
        right.resize(N, -1);
        par.resize(N, -1);

        std::stack<int> stk;
        for(int i = 0; i < N; i++) {
            int last = -1;
            while(!stk.empty() and val[stk.top()] >= val[i]) {
                last = stk.top();
                stk.pop();
            }
            if(last != -1) {
                left[i] = last;
                par[last] = i;
            }
            if(!stk.empty()) {
                int p = stk.top();
                par[i] = p;
                right[p] = i;
            }
            stk.push(i);
        }
    }
};
```

## Rust
```rust
mod data_structure {
    pub struct CartesianTree<T: Copy + Ord> {
        val: Vec<T>,
        par: Vec<Option<usize>>,
        left: Vec<Option<usize>>,
        right: Vec<Option<usize>>,
    }

    impl<T: Copy + Ord> CartesianTree<T> {
        pub fn new(val: Vec<T>) -> Self {
            Self::build(val)
        }

        fn build(val: Vec<T>) -> Self {
            let n = val.len();
            let mut par = vec![None; n];
            let mut left = vec![None; n];
            let mut right = vec![None; n];
            let mut stack = Vec::with_capacity(n);

            for i in 0..n {
                let mut last = None;
                while let Some(index) = stack.pop() {
                    if val[index] >= val[i] {
                        last = Some(index);
                    } else {
                        stack.push(index);
                        break;
                    }
                }

                if let Some(last) = last {
                    left[i] = Some(last);
                    par[last] = Some(i);
                }
                if !stack.is_empty() {
                    let p = *stack.last().unwrap();
                    par[i] = Some(p);
                    right[p] = Some(i);
                }
                stack.push(i);
            }

            Self {
                val,
                par,
                left,
                right,
            }
        }

        pub fn get_par_index(&self, i: usize) -> Option<usize> {
            self.par[i]
        }
        pub fn get_left_index(&self, i: usize) -> Option<usize> {
            self.left[i]
        }
        pub fn get_right_index(&self, i: usize) -> Option<usize> {
            self.right[i]
        }
        pub fn get_par_value(&self, i: usize) -> Option<T> {
            if let Some(par) = self.par[i] {
                Some(self.val[par])
            } else {
                None
            }
        }
        pub fn get_left_value(&self, i: usize) -> Option<T> {
            if let Some(left) = self.left[i] {
                Some(self.val[left])
            } else {
                None
            }
        }
        pub fn get_right_value(&self, i: usize) -> Option<T> {
            if let Some(right) = self.right[i] {
                Some(self.val[right])
            } else {
                None
            }
        }
    }
}
```

## Examples

- [Cartesian Tree (C++)](https://judge.yosupo.jp/submission/226423)
