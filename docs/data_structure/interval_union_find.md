## C++
```c++
struct IntervalUnionFind {
    IntervalUnionFind(int N): N(N), left(N, -1), length(N, 1) {
        for(int i = 0; i < N; ++i) {
            left[i] = i;
        }
    }

    // x のグループの index の最小値
    int root(int x) {
        if (left[x] == x) return x;
        return left[x] = root(left[x]);
    }

    // x のグループと y のグループを merge する
    // 2 つのグループは隣り合ってることを想定
    void unite(int x, int y) {
        if (x > y) std::swap(x, y);

        int rx = root(x), ry = root(y);
        if(ry > 0 and rx == root(left[ry - 1])) {
            left[ry] = rx;
            length[rx] += length[ry];
            length[ry] = 0;
        }
    }

    bool same(int x, int y) {
        return root(x) == root(y);
    }

    // x のグループの区間 [l, r)
    std::pair<int, int> interval(int x) {
        int rx = root(x);
        return std::pair<int, int>(rx, rx + length[rx]);
    }

    size_t size(int x) {
        return length[root(x)];
    }

  private:
    int N;
    // left[i] = i のグループの index の最小値
    // length[i] = i のグループの区間の長さ
    std::vector<int> left, length;
};
```
