## Rust

```rust
/// 偏角ソート用の比較関数
fn argcmp((x1, y1): (i64, i64), (x2, y2): (i64, i64)) -> std::cmp::Ordering {
    ((y1, x1) < (0, 0))
        .cmp(&((y2, x2) < (0, 0)))
        .then_with(|| (x2 * y1).cmp(&(x1 * y2)))
}

/// (x0, y0) を中心とした角 (x0, y0), (x1, y1), (x2, y2) を求める
fn arg((x0, y0): (i64, i64), (x1, y1): (i64, i64), (x2, y2): (i64, i64)) -> f64 {
    let (a, b) = (x1 - x0, y1 - y0);
    let (c, d) = (x2 - x0, y2 - y0);

    let t1 = atan2(b as f64, a as f64);
    let t2 = atan2(d as f64, c as f64);

    let t = (t2 - t1).abs();
    t.min((std::f64::consts::PI * 2.0 - t).abs())
}
```
