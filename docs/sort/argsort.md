## c++
```c++
auto argsort = [&](int i, int j){
    // [-PI, PI)でソート
    long double thetai = atan2l(y[i], x[i]);
    long double thetaj = atan2l(y[j], x[j]);
    return thetai < thetaj;
};
vector<int> p(N); for(int i=0; i<N; i++) p[i] = i;
sort(p.begin(), p.end(), argsort);
```
