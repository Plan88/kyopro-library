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

struct Circle {
    static constexpr double EPS = 1e-9;
    V o;
    double r;
    Circle(V o = V(), double r = 0) : o(o), r(r) {}

    std::vector<V> get_intersection(const Circle &c) {
        if (is_separate(c))
            return {};
        if (is_inside(c))
            return {};

        V v = c.o - o;
        double d = v.norm();

        // circumscribed
        if (std::abs(d - (r + c.r)) < EPS)
            return std::vector<V>{v * (r / d) + o};
        // inscribed
        if (std::abs(d - std::abs(r - c.r)) < EPS)
            return std::vector<V>{v * (std::max(r, c.r) / d) + o};

        // intersect
        double x = (r * r - c.r * c.r + d * d) / (2 * d), l = std::sqrt(r * r - x * x);
        V v1 = v * (x / d), v2 = v1.rotate90() * (l / x);
        return std::vector<V>{v1 + v2 + o, v1 - v2 + o};
    }

    bool is_intersect(const Circle &c) {
        return !(is_inside(c) or is_separate(c));
    }

    bool is_inside(const V &v) {
        return (v - o).norm() < r + EPS;
    }

    bool is_inside(const Circle &c) {
        V v = c.o - o;
        double d = v.norm();
        return d < std::abs(r - c.r) + EPS;
    }

    bool is_separate(const Circle &c) {
        V v = c.o - o;
        double d = v.norm();
        return r + c.r < d + EPS;
    }
};
```

## Example

- [ABC 151 F - Enclose All (C++)](https://atcoder.jp/contests/abc151/submissions/51780122)
- [ABC 157 F - Yakiniku Optimization Problem (C++)](https://atcoder.jp/contests/abc157/submissions/51780230)
- [ABC 181 F - Silver Woods (C++)](https://atcoder.jp/contests/abc181/submissions/51780322)
