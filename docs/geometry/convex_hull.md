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

## Example

- [AGC021 B - Holes (C++)](https://atcoder.jp/contests/agc021/submissions/52228189)
