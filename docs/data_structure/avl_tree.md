## Rust
```rust
#[allow(dead_code)]
mod data_structure {
    use std::cmp::Ordering;

    pub struct AVLTree<T: Ord> {
        root: AVLTreeInner<T>,
    }

    impl<T: Ord> AVLTree<T> {
        pub fn new() -> Self {
            Self {
                root: AVLTreeInner::Empty,
            }
        }

        pub fn insert(&mut self, value: T) {
            self.root.insert(value);
        }

        pub fn contains(&self, value: T) -> bool {
            self.root.contains(value)
        }

        pub fn get_at_k(&self, k: usize) -> &T {
            self.root.get_at_k(k)
        }

        pub fn erase_one(&mut self, value: T) -> bool {
            self.root.erase(value, 1)
        }

        pub fn erase_all(&mut self, value: T) -> bool {
            self.root.erase(value, usize::MAX)
        }

        pub fn lower_bond(&mut self, value: T) -> Option<&T> {
            self.root.lower_bound(value)
        }
    }

    #[derive(Debug, Clone)]
    struct Node<T: Ord> {
        value: T,
        left: Box<AVLTreeInner<T>>,
        right: Box<AVLTreeInner<T>>,
        count: usize,
        height: usize,
        count_all: usize,
    }

    impl<T: Ord> Node<T> {
        fn new(value: T) -> Self {
            Self {
                value,
                left: Box::new(AVLTreeInner::Empty),
                right: Box::new(AVLTreeInner::Empty),
                count: 1,
                height: 1,
                count_all: 1,
            }
        }
    }

    #[derive(Debug, Clone)]
    enum AVLTreeInner<T: Ord> {
        Empty,
        Node(Node<T>),
    }

    impl<T: Ord> AVLTreeInner<T> {
        fn insert(&mut self, value: T) {
            match *self {
                Self::Empty => *self = Self::Node(Node::new(value)),
                Self::Node(ref mut node) => match node.value.cmp(&value) {
                    Ordering::Less => node.right.insert(value),
                    Ordering::Equal => node.count += 1,
                    Ordering::Greater => node.left.insert(value),
                },
            }
            self.update_node_attributes();
            self.rebalance();
        }

        fn contains(&self, value: T) -> bool {
            match *self {
                Self::Empty => false,
                Self::Node(ref node) => match node.value.cmp(&value) {
                    Ordering::Less => node.right.contains(value),
                    Ordering::Equal => true,
                    Ordering::Greater => node.left.contains(value),
                },
            }
        }

        fn get_at_k(&self, k: usize) -> &T {
            match *self {
                Self::Empty => panic!("maybe k is larger than tree size"),
                Self::Node(ref node) => {
                    let count = node.count;
                    let left_count_all = self.get_left_count_all();
                    match k {
                        x if x < left_count_all => node.left.get_at_k(k),
                        x if x < left_count_all + count => &node.value,
                        _ => node.right.get_at_k(k - count - left_count_all),
                    }
                }
            }
        }

        fn erase(&mut self, value: T, count: usize) -> bool {
            let flag = match *self {
                Self::Empty => false,
                Self::Node(ref mut node) => match node.value.cmp(&value) {
                    Ordering::Less => node.right.erase(value, count),
                    Ordering::Equal => {
                        node.count = node.count.saturating_sub(count);
                        if node.count == 0 {
                            if let Self::Node(max) = node.left.take_max() {
                                node.value = max.value;
                                node.count = max.count;
                            } else if let Self::Node(min) = node.right.take_min() {
                                node.value = min.value;
                                node.count = min.count;
                            } else {
                                self.take();
                            }
                        }
                        true
                    }
                    Ordering::Greater => node.left.erase(value, count),
                },
            };
            self.update_node_attributes();
            self.rebalance();
            flag
        }

        fn lower_bound(&self, value: T) -> Option<&T> {
            match *self {
                Self::Empty => None,
                Self::Node(ref node) => {
                    let lower = match node.value.cmp(&value) {
                        Ordering::Less => node.right.lower_bound(value),
                        Ordering::Equal => Some(&node.value),
                        Ordering::Greater => {
                            let mut lower = Some(&node.value);
                            if let Some(value) = node.left.lower_bound(value) {
                                lower = Some(value);
                            }
                            lower
                        }
                    };
                    lower
                }
            }
        }

        fn take_max(&mut self) -> Self {
            let max = match *self {
                Self::Empty => Self::Empty,
                Self::Node(ref mut node) => {
                    let max = if node.right.is_empty() {
                        let new_root = node.left.take();
                        let max = self.take();
                        *self = new_root;
                        max
                    } else {
                        node.right.take_max()
                    };
                    max
                }
            };
            self.update_node_attributes();
            self.rebalance();
            max
        }

        fn take_min(&mut self) -> Self {
            let min = match *self {
                Self::Empty => Self::Empty,
                Self::Node(ref mut node) => {
                    let min = if node.left.is_empty() {
                        let new_root = node.right.take();
                        let max = self.take();
                        *self = new_root;
                        max
                    } else {
                        node.left.take_min()
                    };
                    self.rebalance();
                    min
                }
            };
            self.update_node_attributes();
            self.rebalance();
            min
        }

        fn take(&mut self) -> Self {
            std::mem::take(self)
        }

        fn rebalance(&mut self) {
            match self.get_height_diff() {
                x if x < -1 => self.rotate_left(),
                x if x > 1 => self.rotate_right(),
                _ => (),
            }
        }

        fn rotate_right(&mut self) {
            match *self {
                Self::Empty => (),
                Self::Node(ref mut node) => {
                    let mut new_root = node.left.take();
                    if let Self::Node(ref mut root_node) = new_root {
                        node.left = Box::new(root_node.right.take());
                        root_node.right = Box::new(self.take());
                        root_node.right.update_node_attributes();
                    } else {
                        panic!("AVLTree is Broken (attempt to rotate right but new root is empty)");
                    }
                    *self = new_root;
                    self.update_node_attributes();
                }
            }
        }

        fn rotate_left(&mut self) {
            match *self {
                Self::Empty => (),
                Self::Node(ref mut node) => {
                    let mut new_root = node.right.take();
                    if let Self::Node(ref mut root_node) = new_root {
                        node.right = Box::new(root_node.left.take());
                        root_node.left = Box::new(self.take());
                        root_node.left.update_node_attributes();
                    } else {
                        panic!("AVLTree is Broken (attempt to rotate left but new root is empty)");
                    }
                    *self = new_root;
                    self.update_node_attributes();
                }
            }
        }

        fn update_node_attributes(&mut self) {
            match *self {
                Self::Empty => (),
                Self::Node(ref mut node) => {
                    let left_tree_info = node.left.get_tree_info();
                    let right_tree_info = node.right.get_tree_info();
                    node.height = 1 + left_tree_info.height.max(right_tree_info.height);
                    node.count_all =
                        node.count + left_tree_info.count_all + right_tree_info.count_all;
                }
            }
        }

        fn get_tree_info(&self) -> TreeInfo {
            match *self {
                Self::Empty => TreeInfo {
                    count: 0,
                    height: 0,
                    count_all: 0,
                },
                Self::Node(ref node) => TreeInfo {
                    count: node.count,
                    height: node.height,
                    count_all: node.count_all,
                },
            }
        }

        fn get_height_diff(&self) -> isize {
            match *self {
                Self::Empty => 0,
                Self::Node(ref node) => {
                    let left = node.left.get_tree_info().height as isize;
                    let right = node.right.get_tree_info().height as isize;
                    left - right
                }
            }
        }

        fn get_left_count_all(&self) -> usize {
            match *self {
                Self::Empty => 0,
                Self::Node(ref node) => node.left.get_tree_info().count_all,
            }
        }

        fn is_empty(&self) -> bool {
            match *self {
                Self::Empty => true,
                _ => false,
            }
        }
    }

    impl<T: Ord> Default for AVLTreeInner<T> {
        fn default() -> Self {
            Self::Empty
        }
    }

    struct TreeInfo {
        count: usize,
        height: usize,
        count_all: usize,
    }
}
```

## Examples
- [ARC033 C - データ構造 (Rust)](https://atcoder.jp/contests/arc033/submissions/56654802)
