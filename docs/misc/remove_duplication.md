## C++
```c++
template<class T>
std::vector<T> remove_duplication(std::vector<T> v){
    std::sort(v.begin(), v.end());
	v.erase(std::unique(v.begin(), v.end()), v.end());
	return v;
}
```
