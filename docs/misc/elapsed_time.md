## C++
```c++
inline double get_time_ms(void){
    return static_cast<double>(
        std::chrono::duration_cast<std::chrono::nanoseconds>(std::chrono::steady_clock::now().time_since_epoch()).count()
        )/1000000;
}
```
