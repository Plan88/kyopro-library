## Rust
```rust
pub mod data_structure {
    pub trait Monoid {
        fn e() -> Self;
        fn op(&self, rhs: &Self) -> Self;
    }

    struct Node<T> {
        /// 区間の左端の添字
        left: usize,
        /// 区間の右端の添字 + 1
        right: usize,
        /// [left, right) の値を集約したもの
        val: T,
        /// 親ノードの添字
        parent: Option<usize>,
        /// [left, mid) のノードの添字
        left_child: Option<usize>,
        /// [mid, right) のノードの添字
        right_child: Option<usize>,
    }

    impl<T: Monoid> Node<T> {
        fn new(left: usize, right: usize, parent: Option<usize>) -> Self {
            Self {
                left,
                right,
                val: T::e(),
                parent,
                left_child: None,
                right_child: None,
            }
        }
    }

    pub struct DynamicSegmentTree<T> {
        nodes: Vec<Node<T>>,
    }

    impl<T: Monoid> DynamicSegmentTree<T> {
        pub fn new(size: usize) -> Self {
            Self {
                nodes: vec![Node::new(0, size, None)],
            }
        }

        /// 添字 pos に値 x をセットする
        pub fn set(&mut self, pos: usize, x: T) {
            let mut cur = 0;
            let (mut l, mut r) = (self.nodes[0].left, self.nodes[0].right);
            while r - l > 1 {
                let m = (l + r) / 2;
                if (l..m).contains(&pos) {
                    if self.nodes[cur].left_child.is_none() {
                        self.nodes[cur].left_child = Some(self.nodes.len());
                        self.nodes.push(Node::new(l, m, Some(cur)));
                    }
                    cur = self.nodes[cur].left_child.unwrap();
                    r = m;
                } else {
                    if self.nodes[cur].right_child.is_none() {
                        self.nodes[cur].right_child = Some(self.nodes.len());
                        self.nodes.push(Node::new(m, r, Some(cur)));
                    }
                    cur = self.nodes[cur].right_child.unwrap();
                    l = m;
                }
            }

            self.nodes[cur].val = x;

            while let Some(pos) = self.nodes[cur].parent {
                cur = pos;
                self.update(cur);
            }
        }

        /// 添字 pos の値の参照を取得する
        /// pos に値がセットしてあれば Some(&val) を返す
        /// pos に値がセットしてなければ None を返す
        pub fn get(&self, pos: usize) -> Option<&T> {
            let mut cur = 0;
            let (mut l, mut r) = (self.nodes[0].left, self.nodes[0].right);
            while r - l > 1 {
                let m = (l + r) / 2;
                if (l..m).contains(&pos) {
                    cur = self.nodes[cur].left_child?;
                    r = m;
                } else {
                    cur = self.nodes[cur].right_child?;
                    l = m;
                }
            }
            Some(&self.nodes[cur].val)
        }

        pub fn all_prod(&self) -> T {
            self.prod(..)
        }

        /// 区間 range の集約を求める
        pub fn prod<R>(&self, range: R) -> T
        where
            R: std::ops::RangeBounds<usize>,
        {
            let (l, r) = self.to_half_open_pair(range);
            self.get_agg(0, l, r)
        }

        /// nodes[index] における、区間 range の集約を求める
        fn get_agg(&self, index: usize, left: usize, right: usize) -> T {
            let (l, r) = (self.nodes[index].left, self.nodes[index].right);
            if r <= left || right <= l {
                return T::e();
            }
            if left <= l && r <= right {
                return self.nodes[index].val.op(&T::e());
            }
            let lval = self.nodes[index]
                .left_child
                .map(|i| self.get_agg(i, left, right))
                .unwrap_or(T::e());
            let rval = self.nodes[index]
                .right_child
                .map(|i| self.get_agg(i, left, right))
                .unwrap_or(T::e());
            lval.op(&rval)
        }

        /// nodes[index] を更新する
        fn update(&mut self, index: usize) {
            let default = T::e();
            let left = self.nodes[index]
                .left_child
                .map(|i| &self.nodes[i].val)
                .unwrap_or(&default);
            let right = self.nodes[index]
                .right_child
                .map(|i| &self.nodes[i].val)
                .unwrap_or(&default);
            self.nodes[index].val = left.op(right);
        }

        /// range = [l, r) と表せる (l, r) を返す
        fn to_half_open_pair<R>(&self, range: R) -> (usize, usize)
        where
            R: std::ops::RangeBounds<usize>,
        {
            let l = match range.start_bound() {
                std::ops::Bound::Included(&l) => l,
                std::ops::Bound::Excluded(&l) => l + 1,
                std::ops::Bound::Unbounded => 0,
            };
            let r = match range.end_bound() {
                std::ops::Bound::Included(&r) => r + 1,
                std::ops::Bound::Excluded(&r) => r,
                std::ops::Bound::Unbounded => self.nodes[0].right,
            };
            (l, r)
        }
    }
}
```

## Examples
* [ABC403 G - Odd Position Sum Query (Rust)](https://atcoder.jp/contests/abc403/submissions/67206374)
