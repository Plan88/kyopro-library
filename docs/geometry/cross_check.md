check if two line segment (ax, ay)-(bx, by) and (cx, cy)-(dx, dy) are cross
## C++
```c++
bool check(double ax, double ay, double bx, double by, double cx, double cy, double dx, double dy) {
    double ta = (cx - dx) * (ay - cy) + (cy - dy) * (cx - ax);
    double tb = (cx - dx) * (by - cy) + (cy - dy) * (cx - bx);
    double tc = (ax - bx) * (cy - ay) + (ay - by) * (ax - cx);
    double td = (ax - bx) * (dy - ay) + (ay - by) * (ax - dx);

    return tc * td < 0 && ta * tb < 0;
}
```
