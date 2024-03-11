## c++
```c++
template<long long mod=1000000007>
struct modint {
    long long x;
    modint(long long x=0):x((x%mod+mod)%mod){}
    long long val(){
        return x;
    }
    modint<mod> operator-() const { return modint(-x);}
    modint<mod>& operator+=(const modint a) {
        if ((x += a.x) >= mod) x -= mod;
        return *this;
    }
    modint<mod>& operator-=(const modint a) {
        if ((x += mod-a.x) >= mod) x -= mod;
    	return *this;
    }
    modint<mod>& operator*=(const modint a) {
    	(x *= a.x) %= mod;
        return *this;
    }
    modint<mod> operator+(const modint a) const {
        modint<mod> res(*this);
        return res+=a;
    }
    modint<mod> operator-(const modint a) const {
        modint<mod> res(*this);
        return res-=a;
    }
    modint<mod> operator*(const modint a) const {
        modint<mod> res(*this);
        return res*=a;
    }
    modint<mod> pow(long long t) const {
        if (!t) return 1;
        modint<mod> a = pow(t>>1);
        a *= a;
    	if (t&1) a *= *this;
        return a;
    }

	// must be gcd(x,mod)==1
    modint<mod> inv() const {
		// a^{-1} = 1/a mod p (拡張Euclidの互除法)
		long long b = mod, u = 1, v = 0, z = x;
		while(b){
			long long t = z / b;
			z -= t * b; swap(z, b);
			u -= t * v; swap(u, v);
		}
		u %= mod;
		if (u < 0) u += mod;
		return modint<mod>(u);
	}

    //modint inv() const {
    //    return pow(mod-2);
    //}

    modint<mod>& operator/=(const modint a) {
    	return (*this) *= a.inv();
    }
    modint<mod> operator/(const modint a) const {
        modint<mod> res(*this);
        return res/=a;
    }
};
using mint = modint<998244353>;
```
