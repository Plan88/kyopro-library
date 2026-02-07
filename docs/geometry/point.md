## Rust
```rust
mod geometry {
    use num_traits::Zero;
    use std::ops::{Add, Mul, Sub};

    #[derive(Debug, Clone, Copy, PartialEq, Eq)]
    pub struct Point<T> {
        pub x: T,
        pub y: T,
    }
    impl<T> Point<T> {
        pub fn new(x: T, y: T) -> Self {
            Self { x, y }
        }
    }

    impl<T: Copy + Add<Output = T>> Add for Point<T> {
        type Output = Self;
        fn add(self, rhs: Self) -> Self {
            Self::new(self.x + rhs.x, self.y + rhs.y)
        }
    }
    impl<T: Copy + Sub<Output = T>> Sub for Point<T> {
        type Output = Self;
        fn sub(self, rhs: Self) -> Self {
            Self::new(self.x - rhs.x, self.y - rhs.y)
        }
    }
    impl<T: Copy + Mul<Output = T>> Mul<T> for Point<T> {
        type Output = Self;
        fn mul(self, k: T) -> Self {
            Self::new(self.x * k, self.y * k)
        }
    }

    impl<T: Copy + Add<Output = T> + Mul<Output = T>> Point<T> {
        pub fn dot(self, rhs: Self) -> T {
            self.x * rhs.x + self.y * rhs.y
        }
    }
    impl<T: Copy + Sub<Output = T> + Mul<Output = T>> Point<T> {
        pub fn cross(self, rhs: Self) -> T {
            self.x * rhs.y - self.y * rhs.x
        }
    }

    impl<T> Ord for Point<T>
    where
        T: Ord,
    {
        fn cmp(&self, other: &Self) -> std::cmp::Ordering {
            match self.x.cmp(&other.x) {
                std::cmp::Ordering::Equal => self.y.cmp(&other.y),
                ord => ord,
            }
        }
    }
    impl<T> PartialOrd for Point<T>
    where
        T: Ord,
    {
        fn partial_cmp(&self, other: &Self) -> Option<std::cmp::Ordering> {
            Some(self.cmp(other))
        }
    }

    /// 線分
    #[derive(Debug, Clone, Copy, PartialEq, Eq)]
    pub struct Segment<T> {
        pub a: Point<T>,
        pub b: Point<T>,
    }
    impl<T> Segment<T> {
        pub fn new(a: Point<T>, b: Point<T>) -> Self {
            Self { a, b }
        }
    }
    impl<T: Copy + Sub<Output = T>> Segment<T> {
        /// 始点 a から終点 b への有向ベクトル
        pub fn vector(self) -> Point<T> {
            self.b - self.a
        }
    }
    impl<T: Copy + Add<Output = T> + Sub<Output = T> + Mul<Output = T>> Segment<T> {
        /// 自分のベクトルと点ベクトルの内積
        pub fn dot_vec(self, v: Point<T>) -> T {
            self.vector().dot(v)
        }
        /// 自分のベクトルと他線分の内積
        pub fn dot(self, other: Segment<T>) -> T {
            let v = self.vector();
            v.dot(other.vector())
        }
        /// 自分のベクトルと点ベクトルの外積
        pub fn cross_vec(self, v: Point<T>) -> T {
            self.vector().cross(v)
        }
        /// 自分のベクトルと他線分の外積
        pub fn cross(self, other: Segment<T>) -> T {
            let v = self.vector();
            v.cross(other.vector())
        }
        /// ベクトルの長さの二乗
        pub fn len2(self) -> T {
            let v = self.vector();
            v.dot(v)
        }
    }

    /// 有向線分 a->b に対する点の位置
    #[derive(Debug, Clone, Copy, PartialEq, Eq)]
    pub enum RelativePosition {
        /// 反時計回り側
        Left,
        /// 時計回り側
        Right,
        /// 線分上（端点含む）
        OnSegment,
        /// 線分外で前側（b 側の延長）
        InFront,
        /// 線分外で後ろ側（a 側の延長）
        Behind,
    }

    /// 有向線分 a -> b に対して、点 p の位置関係を返す
    pub fn relate_point_to_directed_segment<T>(seg: Segment<T>, p: Point<T>) -> RelativePosition
    where
        T: Copy + Zero + PartialOrd + Sub<Output = T> + Add<Output = T> + Mul<Output = T>,
    {
        let ab = seg.vector();
        let ap = p - seg.a;
        let v = ab.cross(ap);
        if v > T::zero() {
            return RelativePosition::Left;
        }
        if v < T::zero() {
            return RelativePosition::Right;
        }

        // 共線時：前/後/線分上を投影で判定
        let t = ap.dot(ab); // |ab|^2 * 投影係数
        if t < T::zero() {
            RelativePosition::Behind
        } else {
            let ab2 = seg.len2();
            if t <= ab2 {
                RelativePosition::OnSegment
            } else {
                RelativePosition::InFront
            }
        }
    }

    /// 3点が作る三角形の符号付き面積の2倍
    /// 正: a->b->c が反時計回り, 負: 時計回り
    pub fn signed_triangle_area<T>(a: Point<T>, b: Point<T>, c: Point<T>) -> T
    where
        T: Copy + Sub<Output = T> + Add<Output = T> + Mul<Output = T>,
    {
        let ab = b - a;
        let ac = c - a;
        ab.cross(ac)
    }

    /// 三角形abcに点pが含まれるか（辺上を含む）
    pub fn contains_point_in_triangle<T>(a: Point<T>, b: Point<T>, c: Point<T>, p: Point<T>) -> bool
    where
        T: Copy + Zero + PartialOrd + Sub<Output = T> + Add<Output = T> + Mul<Output = T>,
    {
        let ab = (b - a).cross(p - a);
        let bc = (c - b).cross(p - b);
        let ca = (a - c).cross(p - c);

        (ab >= T::zero() && bc >= T::zero() && ca >= T::zero())
            || (ab <= T::zero() && bc <= T::zero() && ca <= T::zero())
    }
}
```
