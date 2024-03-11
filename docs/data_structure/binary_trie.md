## c++
```c++
template<class T, int bit=30>
struct BinaryTrie {
    vector<pair<int,int>> dat;

    BinaryTrie(): BinaryTrie(vector<T>{}) {}
    BinaryTrie(const vector<T> v){
        dat.emplace_back(-1, -1);

        for(T x : v) insert(x);
    }

    void insert(T x){
        int pos = 0;
        for(int i=bit-1; i>=0; i--){
            int b = 1 & (x>>i);
            if(b){
                if(dat[pos].second == -1){
                    dat[pos].second = dat.size();
                    dat.emplace_back(-1,-1);
                }
                pos = dat[pos].second;
            }
            else{
                if(dat[pos].first == -1){
                    dat[pos].first = dat.size();
                    dat.emplace_back(-1,-1);
                }
                pos = dat[pos].first;
            }
        }
    }

    bool exist(T x){
        int pos = 0;
        for(int i=bit-1; i>=0; i--){
            int b = 1 & (x>>i);
            if(b){
                if(dat[pos].second == -1) return false;
                pos = dat[pos].second;
            }
            else{
                if(dat[pos].first == -1) return false;
                pos = dat[pos].first;
            }
        }
        return true;
    }

    T min_xor(T x){
        if(dat.size() == 1) return -1;
        int pos = 0, ret = 0;
        for(int i=bit-1; i>=0; i--){
            int b = 1 & (x>>i);
            if(b){
                if(dat[pos].second == -1){
                    ret |= 1LL<<i;
                    pos = dat[pos].first;
                }
                else{
                    pos = dat[pos].second;
                }
            }
            else{
                if(dat[pos].first == -1){
                    ret |= 1LL<<i;
                    pos = dat[pos].second;
                }
                else{
                    pos = dat[pos].first;
                }
            }
        }
        return ret;
    }
};
```
