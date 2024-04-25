## C++
```c++
template <class T = int>
struct SlopeTrick {

    SlopeTrick(T inf) : inf(inf) {
        ql.push(-inf);
        qr.push(inf);
    }

    void add_slope_l(T a) {
        // add f(x) = max(0, a-x)
        T r0 = qr.top();
        m += std::max(T(0), a - r0);

        a -= num_shift;
        qr.push(a);
        T b = qr.top();
        qr.pop();
        ql.push(b);
    }

    void add_slope_r(T a) {
        // add f(x) = max(0, x-a)
        T l0 = ql.top();
        m += std::max(T(0), l0 - a);

        a -= num_shift;
        ql.push(a);
        T b = ql.top();
        ql.pop();
        qr.push(b);
    }

    void add_const(T a) {
        // add f(x) = a
        m += a;
    }

    void shift(T a) {
        // f(x) <- f(x-a)
        num_shift += a;
    }

    void cummin_l() {
        // f(x) <- min_{y <= x} f(y)
        while (qr.size() > 1) {
            qr.pop();
        }
    }

    void cummin_r() {
        // f(x) <- min_{x <= y} f(y)
        while (ql.size() > 1) {
            ql.pop();
        }
    }

    T min() {
        // min f(x)
        return m;
    }

    T argmin() {
        // argmin f(x)
        T a = ql.top();
        if (a != -inf)
            a += num_shift;
        return a;
    }

  private:
    std::priority_queue<T, std::vector<T>, std::greater<T>> qr;
    std::priority_queue<T> ql;
    T m = 0, num_shift = 0, inf;
};
```

## Example

- [ABC127 F - Absolute Minima (C++)](https://atcoder.jp/contests/abc127/submissions/52627818)

## Reference

- [https://maspypy.com/slope-trick-1-%E8%A7%A3%E8%AA%AC%E7%B7%A8](https://maspypy.com/slope-trick-1-%E8%A7%A3%E8%AA%AC%E7%B7%A8)
