## Rust
```Rust
mod data_structure {
    /// 長方形 (xl, xr, yl, yr) の和集合の面積を求める。
    /// 各長方形は xl <= x < xr, yl <= y < yr の領域を表す。
    pub fn union_area(rects: &[(i64, i64, i64, i64)]) -> i64 {
        if rects.is_empty() {
            return 0;
        }

        // y座標を収集・座標圧縮
        let mut ys: Vec<i64> = Vec::with_capacity(rects.len() * 2);
        for &(_, _, yl, yr) in rects {
            ys.push(yl);
            ys.push(yr);
        }
        ys.sort();
        ys.dedup();

        let compress_y = |y: i64| -> usize { ys.binary_search(&y).unwrap() };

        // x方向のイベントを生成: (x座標, +1/-1, y区間)
        let mut events: Vec<(i64, i32, usize, usize)> = Vec::with_capacity(rects.len() * 2);
        for &(xl, xr, yl, yr) in rects {
            let cyl = compress_y(yl);
            let cyr = compress_y(yr);
            events.push((xl, 1, cyl, cyr));
            events.push((xr, -1, cyl, cyr));
        }
        events.sort();

        let mut seg = SegTree::new(&ys);

        let mut area: i64 = 0;
        let mut prev_x = events[0].0;

        for &(x, delta, yl, yr) in &events {
            // 前のx位置からここまでの面積を加算
            area += (x - prev_x) * seg.covered();
            prev_x = x;

            // y区間のcntを更新
            seg.add(yl, yr, delta);
        }

        area
    }

    /// スイープライン専用セグメントツリー
    /// 各リーフは圧縮後のy区間 [ys[i], ys[i+1]) に対応する。
    struct SegTree {
        n: usize,
        cnt: Vec<i32>,
        covered: Vec<i64>,
        /// 各リーフ区間の実際の長さ
        len: Vec<i64>,
    }

    impl SegTree {
        fn new(ys: &[i64]) -> Self {
            let m = ys.len() - 1; // 区間の数
            let n = m.next_power_of_two();

            let mut len = vec![0i64; 2 * n];
            for i in 0..m {
                len[n + i] = ys[i + 1] - ys[i];
            }
            for i in (1..n).rev() {
                len[i] = len[2 * i] + len[2 * i + 1];
            }

            Self {
                n,
                cnt: vec![0; 2 * n],
                covered: vec![0; 2 * n],
                len,
            }
        }

        fn covered(&self) -> i64 {
            self.covered[1]
        }

        /// 圧縮後の区間 [l, r) に delta を加算
        fn add(&mut self, l: usize, r: usize, delta: i32) {
            self.add_inner(1, 0, self.n, l, r, delta);
        }

        fn add_inner(
            &mut self,
            k: usize,
            seg_l: usize,
            seg_r: usize,
            l: usize,
            r: usize,
            delta: i32,
        ) {
            if seg_r <= l || r <= seg_l {
                return;
            }
            if l <= seg_l && seg_r <= r {
                self.cnt[k] += delta;
                self.pull(k);
                return;
            }
            let mid = (seg_l + seg_r) / 2;
            self.add_inner(2 * k, seg_l, mid, l, r, delta);
            self.add_inner(2 * k + 1, mid, seg_r, l, r, delta);
            self.pull(k);
        }

        fn pull(&mut self, k: usize) {
            if self.cnt[k] > 0 {
                self.covered[k] = self.len[k];
            } else if k >= self.n {
                self.covered[k] = 0;
            } else {
                self.covered[k] = self.covered[2 * k] + self.covered[2 * k + 1];
            }
        }
    }
}
```
