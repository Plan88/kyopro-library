整数 $S$ のビットが立つ位置を要素としたときの部分集合を列挙

## C++
```c++
void enumerate_subset(int S) {
    // S 自身も含める時は T = S で初期化する
    for(int T = S & (S - 1); T > 0; T = S & (T - 1)) {
    }
}
```
