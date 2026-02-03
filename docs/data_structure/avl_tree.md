## Rust
```rust
#[allow(dead_code)]
mod data_structure {
    use std::cmp::Ordering;

    pub struct AVLTree<K, V> {
        root: Option<Box<Node<K, V>>>,
    }

    impl<K, V> Default for AVLTree<K, V> {
        fn default() -> Self {
            Self::new()
        }
    }

    impl<K, V> AVLTree<K, V> {
        pub fn new() -> Self {
            Self { root: None }
        }
    }

    impl<K: Ord, V> AVLTree<K, V> {
        pub fn insert(&mut self, key: K, value: V) {
            self.root = insert(self.root.take(), key, value);
        }

        pub fn get_count(&self, key: &K) -> usize {
            get_count(&self.root, key)
        }

        pub fn contains(&self, key: &K) -> bool {
            self.get_count(key) > 0
        }

        pub fn get_at_k(&self, k: usize) -> Option<(&K, &V)> {
            let node = get_at_k(&self.root, k);
            node.as_ref().map(|n| (&n.key, &n.value))
        }

        pub fn get_at_k_unique(&self, k: usize) -> Option<(&K, &V)> {
            let node = get_at_k_unique(&self.root, k);
            node.as_ref().map(|n| (&n.key, &n.value))
        }

        pub fn erase_one(&mut self, key: &K) {
            self.root = erase_one(self.root.take(), key);
        }

        pub fn erase_all(&mut self, key: &K) {
            self.root = erase_all(self.root.take(), key);
        }

        pub fn lower_bound(&self, key: &K) -> Option<(&K, &V)> {
            let node = lower_bound(&self.root, key);
            node.as_ref().map(|n| (&n.key, &n.value))
        }

        pub fn upper_bound(&self, key: &K) -> Option<(&K, &V)> {
            let node = upper_bound(&self.root, key);
            node.as_ref().map(|n| (&n.key, &n.value))
        }

        pub fn len(&self) -> usize {
            self.root.as_ref().map_or(0, |n| n.agg.count_all)
        }

        pub fn len_unique(&self) -> usize {
            self.root.as_ref().map_or(0, |n| n.agg.count_unique)
        }

        pub fn is_empty(&self) -> bool {
            self.root.is_none()
        }

        /// 指定した key を持つノードへの可変参照を返す
        fn get_node_mut(&mut self, key: &K) -> &mut Option<Box<Node<K, V>>> {
            let mut cur = &mut self.root;
            while let Some(node) = cur {
                let next: *mut Option<Box<Node<K, V>>> = match node.key.cmp(key) {
                    Ordering::Less => &mut node.right,
                    Ordering::Equal => return cur,
                    Ordering::Greater => &mut node.left,
                };
                cur = unsafe { &mut *next };
            }
            cur
        }
    }

    #[derive(Debug)]
    struct Node<K, V> {
        key: K,
        value: V,
        left: Option<Box<Self>>,
        right: Option<Box<Self>>,
        agg: Agg,
    }
    type Nd<K, V> = Option<Box<Node<K, V>>>;

    #[derive(Debug)]
    struct Agg {
        /// number of occurrences of this node
        count: usize,
        /// height of the subtree rooted at this node
        /// root node has height 1
        height: usize,
        /// number of unique elements in the subtree rooted at this node
        count_unique: usize,
        /// number of all elements in the subtree rooted at this node
        count_all: usize,
    }

    impl Default for Agg {
        fn default() -> Self {
            Self {
                count: 1,
                height: 1,
                count_unique: 1,
                count_all: 1,
            }
        }
    }

    impl Agg {
        fn empty() -> Self {
            Self {
                count: 0,
                height: 0,
                count_unique: 0,
                count_all: 0,
            }
        }

        fn merge(&mut self, left: &Self, right: &Self) {
            self.height = 1 + left.height.max(right.height);
            self.count_unique = 1 + left.count_unique + right.count_unique;
            self.count_all = self.count + left.count_all + right.count_all;
        }
    }

    impl<K, V> Node<K, V> {
        fn new(key: K, value: V) -> Self {
            Self {
                key,
                value,
                left: None,
                right: None,
                agg: Agg::default(),
            }
        }

        fn update(&mut self) {
            let empty = Agg::empty();
            let left = self.left.as_ref().map(|n| &n.agg).unwrap_or(&empty);
            let right = self.right.as_ref().map(|n| &n.agg).unwrap_or(&empty);
            self.agg.merge(left, right);
        }
    }

    /// tree に key を挿入する
    fn insert<K: Ord, V>(tree: Nd<K, V>, key: K, value: V) -> Nd<K, V> {
        let mut node = match tree {
            None => Box::new(Node::new(key, value)),
            Some(mut node) => {
                match node.key.cmp(&key) {
                    Ordering::Less => node.right = insert(node.right, key, value),
                    Ordering::Equal => {
                        node.agg.count += 1;
                        node.value = value;
                    }
                    Ordering::Greater => node.left = insert(node.left, key, value),
                }
                node
            }
        };
        node.update();
        rebalance(Some(node))
    }

    /// tree から key を持つノードを1つ削除する
    /// 削除後の根ノードを返す
    fn erase_one<K: Ord, V>(tree: Nd<K, V>, key: &K) -> Nd<K, V> {
        if let Some(mut node) = tree {
            match node.key.cmp(&key) {
                Ordering::Less => {
                    node.right = erase_one(node.right, key);
                }
                Ordering::Equal => {
                    if node.agg.count > 1 {
                        node.agg.count -= 1;
                    } else {
                        // ノードを削除
                        return erase_root(Some(node));
                    }
                }
                Ordering::Greater => {
                    node.left = erase_one(node.left, key);
                }
            }
            node.update();
            rebalance(Some(node))
        } else {
            None
        }
    }

    /// tree から key を持つノードをすべて削除する
    fn erase_all<K: Ord, V>(tree: Nd<K, V>, key: &K) -> Nd<K, V> {
        if let Some(mut node) = tree {
            match node.key.cmp(&key) {
                Ordering::Less => {
                    node.right = erase_all(node.right, key);
                }
                Ordering::Equal => {
                    // ノードを削除
                    return erase_root(Some(node));
                }
                Ordering::Greater => {
                    node.left = erase_all(node.left, key);
                }
            }
            node.update();
            rebalance(Some(node))
        } else {
            None
        }
    }

    /// tree の根ノードを削除する
    /// 削除後の根ノードを返す
    fn erase_root<K: Ord, V>(tree: Nd<K, V>) -> Nd<K, V> {
        if let Some(mut node) = tree {
            if node.left.is_none() {
                return node.right;
            } else if node.right.is_none() {
                return node.left;
            } else if let (new_left, Some(mut max)) = take_max(node.left.take()) {
                max.left = new_left;
                max.right = node.right;
                max.update();
                return rebalance(Some(max));
            } else if let (new_right, Some(mut min)) = take_min(node.right.take()) {
                min.right = new_right;
                min.left = node.left;
                min.update();
                return rebalance(Some(min));
            } else {
                return None;
            }
        } else {
            None
        }
    }

    /// tree の key 以上の最小のノードを返す
    fn lower_bound<'a, 'b, K: Ord, V>(tree: &'a Nd<K, V>, value: &'b K) -> &'a Nd<K, V> {
        let mut cur = tree;
        let mut lower = None;
        while let Some(node) = cur {
            match node.key.cmp(value) {
                Ordering::Less => {
                    cur = &node.right;
                }
                Ordering::Equal => {
                    return cur;
                }
                Ordering::Greater => {
                    lower = Some(cur);
                    cur = &node.left;
                }
            }
        }
        lower.unwrap_or(&None)
    }

    /// tree の key より大きい最小の要素を返す
    fn upper_bound<'a, 'b, K: Ord, V>(tree: &'a Nd<K, V>, key: &'b K) -> &'a Nd<K, V> {
        let mut cur = tree;
        let mut upper = None;
        while let Some(node) = cur {
            match node.key.cmp(key) {
                Ordering::Less | Ordering::Equal => {
                    cur = &node.right;
                }
                Ordering::Greater => {
                    upper = Some(cur);
                    cur = &node.left;
                }
            }
        }
        upper.unwrap_or(&None)
    }

    /// tree に key がいくつ含まれているか返す
    fn get_count<K: Ord, V>(tree: &Nd<K, V>, key: &K) -> usize {
        let mut cur = tree;
        while let Some(node) = cur {
            match node.key.cmp(&key) {
                Ordering::Less => cur = &node.right,
                Ordering::Equal => return node.agg.count,
                Ordering::Greater => cur = &node.left,
            }
        }
        0
    }

    /// k (0-indexed) 番目の要素を返す
    fn get_at_k<K: Ord, V>(tree: &Nd<K, V>, mut k: usize) -> &Nd<K, V> {
        let mut cur = tree;
        while let Some(node) = cur {
            let left_count_all = node.left.as_ref().map_or(0, |n| n.agg.count_all);
            match k.cmp(&left_count_all) {
                Ordering::Less => cur = &node.left,
                Ordering::Equal | Ordering::Greater if k < left_count_all + node.agg.count => {
                    return cur;
                }
                _ => {
                    k -= left_count_all + node.agg.count;
                    cur = &node.right;
                }
            }
        }
        &None
    }

    /// k (0-indexed) 番目のユニークな要素を返す
    fn get_at_k_unique<K: Ord, V>(tree: &Nd<K, V>, mut k: usize) -> &Nd<K, V> {
        let mut cur = tree;
        while let Some(node) = cur {
            let left_count_unique = node.left.as_ref().map_or(0, |n| n.agg.count_unique);
            match k.cmp(&left_count_unique) {
                Ordering::Less => cur = &node.left,
                Ordering::Equal => {
                    return cur;
                }
                _ => {
                    k -= left_count_unique + 1;
                    cur = &node.right;
                }
            }
        }
        &None
    }

    /// tree から最大のノードを取り除き、残った木の根ノードと取り除かれたノードを返す
    fn take_max<K, V>(tree: Nd<K, V>) -> (Nd<K, V>, Nd<K, V>) {
        if let Some(mut node) = tree {
            if node.right.is_none() {
                let left = node.left.take();
                (left, Some(node))
            } else {
                let (new_right, max_node) = take_max(node.right.take());
                node.right = new_right;
                node.update();
                let balanced_node = rebalance(Some(node));
                (balanced_node, max_node)
            }
        } else {
            (None, None)
        }
    }

    /// tree から最小のノードを取り除き、残った木の根ノードと取り除かれたノードを返す
    fn take_min<K, V>(tree: Nd<K, V>) -> (Nd<K, V>, Nd<K, V>) {
        if let Some(mut node) = tree {
            if node.left.is_none() {
                let right = node.right.take();
                (right, Some(node))
            } else {
                let (new_left, min_node) = take_min(node.left.take());
                node.left = new_left;
                node.update();
                let balanced_node = rebalance(Some(node));
                (balanced_node, min_node)
            }
        } else {
            (None, None)
        }
    }

    /// tree をバランスさせる
    /// バランス後の根ノードを返す
    fn rebalance<K, V>(tree: Nd<K, V>) -> Nd<K, V> {
        match get_height_diff(&tree) {
            x if x > 1 => {
                // 左部分木が高い
                let left_height_diff = get_height_diff(&tree.as_ref().unwrap().left);
                if left_height_diff < 0 {
                    // 左右部分木の高さ差が負 -> 左右回転
                    let mut new_tree = tree;
                    new_tree.as_mut().unwrap().left =
                        rotate_left(new_tree.as_mut().unwrap().left.take());
                    rotate_right(new_tree)
                } else {
                    // 右回転
                    rotate_right(tree)
                }
            }
            x if x < -1 => {
                // 右部分木が高い
                let right_height_diff = get_height_diff(&tree.as_ref().unwrap().right);
                if right_height_diff > 0 {
                    // 右左部分木の高さ差が正 -> 右左回転
                    let mut new_tree = tree;
                    new_tree.as_mut().unwrap().right =
                        rotate_right(new_tree.as_mut().unwrap().right.take());
                    rotate_left(new_tree)
                } else {
                    // 左回転
                    rotate_left(tree)
                }
            }
            _ => tree,
        }
    }

    /// tree を右回転させる
    /// 回転後の根ノードを返す
    fn rotate_right<K, V>(tree: Nd<K, V>) -> Nd<K, V> {
        if let Some(mut node) = tree {
            if let Some(mut left) = node.left {
                node.left = left.right.take();
                left.right = Some(node);
                left.right.as_mut().unwrap().update();
                left.update();
                Some(left)
            } else {
                panic!("AVLTree is Broken (attempt to rotate right but new root is empty)");
            }
        } else {
            None
        }
    }

    /// tree を左回転させる
    /// 回転後の根ノードを返す
    fn rotate_left<K, V>(tree: Nd<K, V>) -> Nd<K, V> {
        if let Some(mut node) = tree {
            if let Some(mut right) = node.right {
                node.right = right.left.take();
                right.left = Some(node);
                right.left.as_mut().unwrap().update();
                right.update();
                Some(right)
            } else {
                panic!("AVLTree is Broken (attempt to rotate left but new root is empty)");
            }
        } else {
            None
        }
    }

    /// tree の左右の子の高さの差を返す
    fn get_height_diff<K, V>(tree: &Nd<K, V>) -> i32 {
        if let Some(node) = tree {
            let left_height = node.left.as_ref().map_or(0, |n| n.agg.height);
            let right_height = node.right.as_ref().map_or(0, |n| n.agg.height);
            left_height as i32 - right_height as i32
        } else {
            0
        }
    }
}
```

## Examples
- [ARC033 C - データ構造 (Rust)](https://atcoder.jp/contests/arc033/submissions/72977237)
