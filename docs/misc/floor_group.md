## C++
$\mathcal{O}(\log N)$
```c++
int l = 1;
while(l<=N){
    int r = N/(N/l)+1; // [l,r)は商が同じ
    cout << "[" << l << "," << r << ")" << endl;
    l = r;
}
```
