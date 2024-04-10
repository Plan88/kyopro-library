## C++
```c++
std::vector<std::string> split(const std::string &str, char sep) {
    std::vector<string> v;
    std::stringstream ss(str);
    std::string buffer;
    while (std::getline(ss, buffer, sep)) {
        v.push_back(buffer);
    }
    return v;
}
```
