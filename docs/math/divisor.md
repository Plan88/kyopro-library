## C++
```c++
std::vector<long long> make_divisors(long long n){
    std::vector<long long> divisors;
    for(long long i=1; i*i<=n; i++){
        if(n%i == 0){
            divisors.push_back(i);
            if(i*i != n) divisors.push_back(n/i);
        }
    }
    // std::sort(divisors.begin(), divisors.end());
    return divisors;
}
```
