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
