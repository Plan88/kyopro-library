## c++
```c++
long long mod_inv(long long a, long long mod){
    // a^{-1} = 1/a mod p (拡張Euclidの互除法)
    long long b = mod, u = 1, v = 0;
    while (b) {
        long long t = a / b;
        a -= t * b; swap(a, b);
        u -= t * v; swap(u, v);
    }
    u %= mod;
    if (u < 0) u += mod;
    return u;
}
long long mod_pow(long long a, long long n, long long mod){
    long long ret = 1;
    long long p = a % mod;
    while(n){
        if(n&1) ret = ret * p % mod;
        p = p * p % mod;
        n >>= 1;
    }
    return ret;
}


template<int mod, int primitive_root>
class NTT {
public:
	int get_mod() const { return mod; }
	void _ntt(vector<long long>& a, int sign) {
		const int n = a.size();
		assert((n ^ (n&-n)) == 0); //n = 2^k

		const int g = 3; //g is primitive root of mod
		int h = (int)mod_pow(g, (mod - 1) / n, mod); // h^n = 1
		if (sign == -1) h = (int)mod_inv(h, mod); //h = h^-1 % mod

		//bit reverse
		int i = 0;
		for (int j=1; j<n-1; ++j) {
			for (int k = n >> 1; k >(i ^= k); k >>= 1);
			if (j < i) swap(a[i], a[j]);
		}

		for (int m=1; m<n; m*=2) {
			const int m2 = 2 * m;
			const long long base = mod_pow(h, n / m2, mod);
			long long w = 1;
			for(int x=0; x<m; x++) {
				for (int s = x; s < n; s += m2) {
					long long u = a[s];
					long long d = a[s + m] * w % mod;
					a[s] = u + d;
					if (a[s] >= mod) a[s] -= mod;
					a[s + m] = u - d;
					if (a[s + m] < 0) a[s + m] += mod;
				}
				w = w * base % mod;
			}
		}

		for (auto& x : a) if (x < 0) x += mod;
	}
	void ntt(vector<long long>& input) {
		_ntt(input, 1);
	}
	void intt(vector<long long>& input) {
		_ntt(input, -1);
		const int n_inv = mod_inv(input.size(), mod);
		for (auto& x : input) x = x * n_inv % mod;
	}

	// 畳み込み演算を行う
	vector<long long> convolution(const vector<long long>& a, const vector<long long>& b){
		int ntt_size = 1;
		while (ntt_size < (int)(a.size()+b.size())) ntt_size *= 2;

		vector<long long> _a = a, _b = b;
		_a.resize(ntt_size); _b.resize(ntt_size);

		ntt(_a);
		ntt(_b);

		for(int i=0; i<ntt_size; i++){
			(_a[i] *= _b[i]) %= mod;
		}

		intt(_a);
		return _a;
	}
};

typedef NTT<167772161,3> NTT_1;
typedef NTT<469762049,3> NTT_2;
typedef NTT<1224736769,3> NTT_3;

// garnerのアルゴリズムを直書きしたversion，速い
vector<long long> int32mod_convolution(vector<long long> a, vector<long long> b,int mod){
	for (auto& x : a) x %= mod;
	for (auto& x : b) x %= mod;
	
	NTT_1 ntt1; NTT_2 ntt2; NTT_3 ntt3;
	assert(ntt1.get_mod() < ntt2.get_mod() && ntt2.get_mod() < ntt3.get_mod());
	auto x = ntt1.convolution(a, b);
	auto y = ntt2.convolution(a, b);
	auto z = ntt3.convolution(a, b);

	// garnerのアルゴリズムを極力高速化した
	const long long m1 = ntt1.get_mod(), m2 = ntt2.get_mod(), m3 = ntt3.get_mod();
	const long long m1_inv_m2 = mod_inv(m1, m2);
	const long long m12_inv_m3 = mod_inv(m1 * m2, m3);
	const long long m12_mod = m1 * m2 % mod;
	vector<long long> ret(x.size());
	for(int i=0; i<(int)x.size(); i++){
		long long v1 = (y[i] - x[i]) *  m1_inv_m2 % m2;
		if (v1 < 0) v1 += m2;
		long long v2 = (z[i] - (x[i] + m1 * v1) % m3) * m12_inv_m3 % m3;
		if (v2 < 0) v2 += m3;
		long long constants3 = (x[i] + m1 * v1 + m12_mod * v2) % mod;
		if (constants3 < 0) constants3 += mod;
		ret[i] = constants3;
	}

	return ret;
}

//2^23より大きく，primitive rootに3を持つもの
// const int mods[] = { 1224736769, 469762049, 167772161, 595591169, 645922817, 897581057, 998244353 };
```
