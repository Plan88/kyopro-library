## C++
```c++
struct Line {
    // a*x + b*y + c = 0
    long long a;
    long long b;
    long long c;

    Line(long long a, long long b, long long c) {
        long long g = std::gcd(a, std::gcd(b, c));
        a /= g;
        b /= g;
        c /= g;

        if (a < 0) {
            a *= -1;
            b *= -1;
            c *= -1;
        } else if (a == 0 and b < 0) {
            b *= -1;
            c *= -1;
        }
        this->a = a;
        this->b = b;
        this->c = c;
    }

    static Line from_point(long long x1, long long y1, long long x2, long long y2) {
        // assume (x1, y1) != (x2, y2)
        long long dx = x1 - x2, dy = y1 - y2;
        return Line(dy, -dx, dx * y1 - dy * x1);
    }

    bool operator==(const Line &other) {
        return a == other.a and b == other.b and c == other.c;
    }
};

inline bool operator<(const Line &lhs, const Line &rhs) {
    if (lhs.a != rhs.a)
        return lhs.a < rhs.a;
    if (lhs.b != rhs.b)
        return lhs.b < rhs.b;
    if (lhs.c != rhs.c)
        return lhs.c < rhs.c;
    return false;
}
```

## Example
[ARC 173 B - Make Many Triangles (C++)](https://atcoder.jp/contests/arc173/submissions/51779544)
