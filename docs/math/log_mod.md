## c++
```c++
long long mul_modp(long long a, long long b){
    // a*b mod p
    return ((a % MOD) * (b % MOD)) % MOD;
}

long long pow_modp(long long a, long long n){
    // a^n mod p
    long long res = 1;
    while (n > 0) {
        if (n & 1) res = mul_modp(res, a);
        a = mul_modp(a, a);
        n >>= 1;
    }
    return res;
}

long long modinv(long long a){
    // a^{-1} = 1/a mod p (拡張Euclidの互除法)
    long long b = MOD, u = 1, v = 0;
    while (b) {
        long long t = a / b;
        a -= t * b; swap(a, b);
        u -= t * v; swap(u, v);
    }
    u %= MOD;
    if (u < 0) u += MOD;
    return u;
}

long long log_modp(long long a, long long b) {
    // a^x ≡ b (mod. m) となる最小の正の整数 x を求める
    a %= MOD, b %= MOD;

    // calc sqrt{M}
    long long lo = -1, hi = MOD;
    while (hi - lo > 1) {
        long long mid = (lo + hi) / 2;
        if (mid * mid >= MOD) hi = mid;
        else lo = mid;
    }
    long long sqrtM = hi;

    // {a^0, a^1, a^2, ..., a^sqrt(m)} 
    map<long long, long long> apow;
    long long amari = 1;
    for (long long r = 0; r < sqrtM; ++r) {
        if (!apow.count(amari)) apow[amari] = r;
        (amari *= a) %= MOD;
    }

    // check each A^p
    long long A = pow_modp(modinv(a), sqrtM);
    amari = b;
    for (long long q = 0; q < sqrtM; ++q) {
        if (apow.count(amari)) {
            long long res = q * sqrtM + apow[amari];
            if (res > 0) return res;
        }
        (amari *= A) %= MOD;
    }

    // no solutions
    return -1;
}
```
