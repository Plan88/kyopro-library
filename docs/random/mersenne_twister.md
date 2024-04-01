## C++
```c++
struct RNG {
    std::mt19937_64 mt;

    RNG(int seed = -1) {
        if (seed == -1) {
            std::random_device rnd; // 非決定的な乱数生成器
            mt = std::mt19937_64(rnd());
        } else {
            mt = std::mt19937_64(seed);
        }
    }

    unsigned long long gen() {
        return mt();
    }

    template <class T>
    T gen(T l, T r) {
        // generate random number in [l, r]
        return l + gen() % (r - l + 1);
    }
};
```
