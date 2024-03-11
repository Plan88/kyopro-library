## c++
```c++
struct SieveEratosthenes{
    vector<int> min_fact;

    SieveEratosthenes(int N){
        _build(N);
    }

    void _build(int N){
        min_fact.resize(N+1);
        for(int i=0; i<=N; i++) min_fact[i] = i;
        
        min_fact[0] = min_fact[1] = -1;

        for(int i=2; i*i<=N; i++){
            for(int j=i*i; j<=N; j+=i){
                if(i < min_fact[j])
                    min_fact[j] = i;
            }
        }
    }

    bool is_prime(int x){
        return min_fact[x] == x;
    }

    vector<pair<int,int>> factorize(int x){
        vector<pair<int,int>> ret;
        while(x>1){
            if(ret.empty() || ret.back().first != min_fact[x])
                ret.emplace_back(min_fact[x], 1);
            else
                ret.back().second++;

            x /= min_fact[x];
        }

        return ret;
    }
};
```
