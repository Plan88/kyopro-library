## C++
```c++
struct V {
    using Type = double;
    Type x, y;
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
```

## Rust
```rust
#[allow(dead_code)]
mod geometry {
    use std::ops::{Add, Div, Mul, Sub};

    use num::{Float, Num};

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
