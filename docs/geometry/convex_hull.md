## C++
```c++
struct V {
    using Type = double;
    Type x, y;
    int id;
    V(Type x = 0, Type y = 0) : x(x), y(y) {}

    V &operator+=(const V &v) {
        x += v.x;
        y += v.y;
        return *this;
    }
    V operator+(const V &v) const { return V(*this) += v; }
    V &operator-=(const V &v) {
        x -= v.x;
        y -= v.y;
        return *this;
    }
    V operator-(const V &v) const { return V(*this) -= v; }
    V &operator*=(Type s) {
        x *= s;
        y *= s;
        return *this;
    }
    V operator*(Type s) const { return V(*this) *= s; }
    V &operator/=(Type s) {
        x /= s;
        y /= s;
        return *this;
    }
    V operator/(Type s) const { return V(*this) /= s; }

    Type dot(const V &v) const { return x * v.x + y * v.y; }
    Type norm2() const { return x * x + y * y; }
    Type norm() const { return std::sqrt(norm2()); }
    V &rotate(Type theta) {
        x = x * std::cos(theta) - y * std::sin(theta);
        y = x * std::sin(theta) + y * std::cos(theta);
        return *this;
    }
    V rotate90() { return V(-y, x); }
};

// 3D cross product of OA and OB vectors, (i.e z-component of their "2D" cross product, but remember that it is not defined in "2D").
// Returns a positive value, if OAB makes a counter-clockwise turn,
// negative for clockwise turn, and zero if the points are collinear.
double cross(const V &O, const V &A, const V &B) {
    return (A.x - O.x) * (B.y - O.y) - (A.y - O.y) * (B.x - O.x);
}

// Returns a list of points on the convex hull in counter-clockwise order.
// Note: the last point in the returned list is the same as the first one.
std::vector<V> convex_hull(std::vector<V> pt) {
    size_t n = pt.size(), k = 0;
    if (n <= 3)
        return pt;
    std::vector<V> H(2 * n);

    // Sort points lexicographically
    std::sort(pt.begin(), pt.end(), [](V a, V b) {
        return a.x < b.x or (a.x == b.x and a.y < b.y);
    });

    // Build lower hull
    for (size_t i = 0; i < n; ++i) {
        while (k >= 2 && cross(H[k - 2], H[k - 1], pt[i]) <= 0)
            k--;
        H[k++] = pt[i];
    }

    // Build upper hull
    for (size_t i = n - 1, t = k + 1; i > 0; --i) {
        while (k >= t && cross(H[k - 2], H[k - 1], pt[i - 1]) <= 0)
            k--;
        H[k++] = pt[i - 1];
    }

    H.resize(k - 1);
    return H;
}
```

## Rust
```rust
#[allow(dead_code)]
mod geometry {
    use std::ops::{Add, Div, Mul, Sub};

    use num::{Float, Num};

    /// Returns a list of points on the convex hull in counter-clockwise order.
    /// Note: the last point in the returned list is the same as the first one.
    pub fn construct_convex_hull<T>(mut points: Vec<Vector2D<T>>) -> Vec<Vector2D<T>>
    where
        T: Num + Copy + PartialOrd + Eq,
    {
        let n = points.len();
        if n <= 3 {
            points.push(points[0]);
            return points;
        }
        let mut k = 0;
        let mut convex_hull = vec![];

        // Sort points lexicographically
        points.sort();

        // Build lower hull
        for i in 0..n {
            while k > 1
                && cross_product(convex_hull[k - 2], convex_hull[k - 1], points[i]) <= T::zero()
            {
                k -= 1;
            }
            while convex_hull.len() <= k {
                convex_hull.push(Vector2D::zero());
            }
            convex_hull[k] = points[i];
            k += 1;
        }

        // Build upper hull
        let t = k + 1;
        for i in (0..n - 1).rev() {
            while k >= t
                && cross_product(convex_hull[k - 2], convex_hull[k - 1], points[i]) <= T::zero()
            {
                k -= 1;
            }
            while convex_hull.len() <= k {
                convex_hull.push(Vector2D::zero());
            }
            convex_hull[k] = points[i];
            k += 1;
        }
        convex_hull.truncate(k);

        convex_hull
    }

    /// 3D cross product of OA and OB vectors, (i.e z-component of their "2D" cross product, but remember that it is not defined in "2D").
    /// Returns a positive value, if OAB makes a counter-clockwise turn,
    /// negative for clockwise turn, and zero if the points are collinear.
    fn cross_product<T: Num + Copy>(o: Vector2D<T>, a: Vector2D<T>, b: Vector2D<T>) -> T {
        let ao = a - o;
        let bo = b - o;
        ao.x * bo.y - ao.y * bo.x
    }

    #[derive(Clone, Copy, PartialEq, PartialOrd, Eq)]
    pub struct Vector2D<T: Num + Copy> {
        pub x: T,
        pub y: T,
    }

    impl<T: Num + Copy> Vector2D<T> {
        pub fn new(x: T, y: T) -> Self {
            Self { x, y }
        }
        pub fn zero() -> Self {
            Self::new(T::zero(), T::zero())
        }
        pub fn rotate90(&self) -> Self {
            Self {
                x: self.y,
                y: self.x,
            }
        }
        pub fn dot(&self, other: Self) -> T {
            self.x * other.x + self.y + other.y
        }
        pub fn norm2(&self) -> T {
            self.x * self.x + self.y * self.y
        }
    }

    impl<T: Float> Vector2D<T> {
        pub fn norm(&self) -> T {
            Float::sqrt(self.norm2())
        }
    }

    impl<T: Num + Copy> Add for Vector2D<T> {
        type Output = Self;
        fn add(self, rhs: Self) -> Self::Output {
            Self {
                x: self.x + rhs.x,
                y: self.y + rhs.y,
            }
        }
    }
    impl<T: Num + Copy> Sub for Vector2D<T> {
        type Output = Self;
        fn sub(self, rhs: Self) -> Self::Output {
            Self {
                x: self.x - rhs.x,
                y: self.y - rhs.y,
            }
        }
    }
    impl<T: Num + Copy> Mul<T> for Vector2D<T> {
        type Output = Self;
        fn mul(self, rhs: T) -> Self::Output {
            Self {
                x: self.x * rhs,
                y: self.y * rhs,
            }
        }
    }
    impl<T: Float> Div<T> for Vector2D<T> {
        type Output = Self;
        fn div(self, rhs: T) -> Self::Output {
            Self {
                x: self.x / rhs,
                y: self.y / rhs,
            }
        }
    }

    impl<T: Num + Copy + PartialOrd + Eq> Ord for Vector2D<T> {
        fn cmp(&self, other: &Self) -> std::cmp::Ordering {
            self.partial_cmp(other).unwrap()
        }
    }
}
```

## Example

- [AGC021 B - Holes (C++)](https://atcoder.jp/contests/agc021/submissions/52228189)
