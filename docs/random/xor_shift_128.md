## C++
```c++
struct FastRNG {
    unsigned long long x, y, z, w;

    FastRNG(int seed = -1) {
        std::mt19937_64 mt;
        if (seed == -1) {
            std::random_device rnd; // 非決定的な乱数生成器
            mt = std::mt19937_64(rnd());
        } else {
            mt = std::mt19937_64(seed);
        }
        set_parameter(mt);
    }

    unsigned long long gen() {
        return xor128();
    }

    template <class T>
    T gen(T l, T r) {
        // generate random number in [l, r]
        return l + gen() % (r - l + 1);
    }

  private:
    void set_parameter(std::mt19937_64 mt) {
        x = mt();
        y = mt();
        z = mt();
        w = mt();
    }

    unsigned long long xor128() {
        unsigned long long t = (x ^ (x << 11));
        x = y;
        y = z;
        z = w;
        return (w = (w ^ (w >> 19)) ^ (t ^ (t >> 8)));
    }
};
```
