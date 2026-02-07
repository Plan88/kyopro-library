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

## Rust
```Rust
pub mod modint {
    use std::ops::{Add, AddAssign, Div, DivAssign, Mul, MulAssign, Sub, SubAssign};
    type Int = u128;

    #[derive(Clone, Copy)]
    pub struct ModInt(pub Int);

    impl ModInt {
        const MOD: Int = 998244353;

        pub fn new(x: Int) -> Self {
            Self(x % Self::MOD)
        }

        pub fn add(&self, other: Self) -> Self {
            let mut val = self.0 + other.0;
            if val >= Self::MOD {
                val -= Self::MOD;
            }
            Self(val)
        }

        pub fn sub(&self, other: Self) -> Self {
            let val = if self.0 >= other.0 {
                self.0 - other.0
            } else {
                Self::MOD - other.0 + self.0
            };
            Self(val)
        }

        pub fn mul(&self, other: Self) -> Self {
            Self((self.0 * other.0) % Self::MOD)
        }

        pub fn div(&self, other: Self) -> Self {
            self.mul(other.inv())
        }

        pub fn pow(&self, mut n: Int) -> Self {
            let mut val = 1;
            let mut pow = self.0;
            while n > 0 {
                if (n & 1) == 1 {
                    val = (val * pow) % Self::MOD;
                }
                pow = (pow * pow) % Self::MOD;
                n >>= 1;
            }
            Self(val)
        }

        pub fn inv(&self) -> Self {
            self.pow(Self::MOD - 2)
        }
    }

    impl Neg for ModInt {
        type Output = ModInt;
        fn neg(self) -> Self::Output {
            ModInt::new(Self::MOD - self.0)
        }
    }

    impl Add for ModInt {
        type Output = ModInt;
        fn add(self, rhs: Self) -> Self::Output {
            ModInt::add(&self, rhs)
        }
    }

    impl Sub for ModInt {
        type Output = ModInt;
        fn sub(self, rhs: Self) -> Self::Output {
            ModInt::sub(&self, rhs)
        }
    }

    impl Mul for ModInt {
        type Output = ModInt;
        fn mul(self, rhs: Self) -> Self::Output {
            ModInt::mul(&self, rhs)
        }
    }

    impl Div for ModInt {
        type Output = ModInt;
        fn div(self, rhs: Self) -> Self::Output {
            ModInt::div(&self, rhs)
        }
    }

    impl Add<Int> for ModInt {
        type Output = ModInt;
        fn add(self, rhs: Int) -> Self::Output {
            ModInt::add(&self, Self::new(rhs))
        }
    }

    impl Sub<Int> for ModInt {
        type Output = ModInt;
        fn sub(self, rhs: Int) -> Self::Output {
            ModInt::sub(&self, Self::new(rhs))
        }
    }

    impl Mul<Int> for ModInt {
        type Output = ModInt;
        fn mul(self, rhs: Int) -> Self::Output {
            ModInt::mul(&self, Self::new(rhs))
        }
    }

    impl Div<Int> for ModInt {
        type Output = ModInt;
        fn div(self, rhs: Int) -> Self::Output {
            ModInt::div(&self, Self::new(rhs))
        }
    }

    impl AddAssign<ModInt> for ModInt {
        fn add_assign(&mut self, rhs: ModInt) {
            *self = self.add(rhs);
        }
    }

    impl SubAssign<ModInt> for ModInt {
        fn sub_assign(&mut self, rhs: ModInt) {
            *self = self.sub(rhs);
        }
    }

    impl MulAssign<ModInt> for ModInt {
        fn mul_assign(&mut self, rhs: ModInt) {
            *self = self.mul(rhs);
        }
    }

    impl DivAssign<ModInt> for ModInt {
        fn div_assign(&mut self, rhs: ModInt) {
            *self = self.div(rhs);
        }
    }

    impl AddAssign<Int> for ModInt {
        fn add_assign(&mut self, rhs: Int) {
            *self = self.add(rhs);
        }
    }

    impl SubAssign<Int> for ModInt {
        fn sub_assign(&mut self, rhs: Int) {
            *self = self.sub(rhs);
        }
    }

    impl MulAssign<Int> for ModInt {
        fn mul_assign(&mut self, rhs: Int) {
            *self = self.mul(rhs);
        }
    }

    impl DivAssign<Int> for ModInt {
        fn div_assign(&mut self, rhs: Int) {
            *self = self.div(rhs);
        }
    }

    impl std::fmt::Debug for ModInt {
        fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
            write!(f, "{}", self.0)
        }
    }

    impl std::fmt::Display for ModInt {
        fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
            write!(f, "{}", self.0)
        }
    }
}
```
