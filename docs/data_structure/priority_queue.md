## Rust

構造体の独自の比較関数の使い方を定期的に忘れるためメモ

```rust
#[derive(PartialEq, Eq)]
struct MyStruct {
    key1: usize,
    key2: i32,
}

impl Ord for MyStruct {
    fn cmp(&self, other: &Self) -> std::cmp::Ordering {
        // ここに大小比較を書く
        // 大きい方が top にくるようになる
        self.key1
            .cmp(&other.key1)
            .then_with(|| self.key2.cmp(&other.key2))
            .reverse()
    }
}

impl PartialOrd for MyStruct {
    fn partial_cmp(&self, other: &Self) -> Option<std::cmp::Ordering> {
        Some(self.cmp(other))
    }
}
```
