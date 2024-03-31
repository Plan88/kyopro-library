## C++
```c++
template<class T>
vector<int> z_algorithm(vector<T> s) {
    int c = 0, n = s.size();
    vector<int> z(n, 0);
    for(int i=1; i<n; i++) {
        int l = i - c;
        if(i + z[l] < c + z[c]) {
            z[i] = z[l];
        } else {
            int j = max(0, c + z[c] - i);
            while(i + j < n and s[j] == s[i + j]) j++;
            z[i] = j;
            c = i;
        }
    }
    z[0] = n;
    return z;
}

vector<int> z_algorithm(string s) {
    int n = s.size();
    vector<char> v(n);
    for(int i=0; i<n; i++) v[i] = s[i];
    return z_algorithm(v);
}
```