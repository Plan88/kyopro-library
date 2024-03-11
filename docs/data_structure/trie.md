## c++
```c++
template<int size, int base>
struct Trie {
    vector<vector<int>> dat;
    vector<int> init, depth;

    Trie(): Trie(vector<string>{}) {}
    Trie(const vector<string>& v){
        init.resize(size, -1);
        depth.push_back(0);
        dat.push_back(init);

        for(string s : v) insert(s);
    }

    void insert(const string& s){
        int pos = 0;
        for(char c : s){
            int x = c - base;
            if(dat[pos][x] == -1){
                dat[pos][x] = dat.size();
                dat.push_back(init);
                depth.push_back(depth[pos]+1);
            }
            pos = dat[pos][x];
        }
    }

    bool exist(const string& s){
        int pos = 0;
        for(char& c : s){
            int x = c - base;
            if(dat[pos][x] == -1) return false;
            pos = dat[pos][x];
        }
        return true;
    }
};
```
