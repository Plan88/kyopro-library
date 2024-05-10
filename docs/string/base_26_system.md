## C++
```c++
struct Base26System {
    /* 英小文字と26進数の相互変換
     * "" = 0, a = 1, b = 2, ..., z = 26, aa = 27, ab = 28
     */
    Base26System() {
        num.resize(max_length + 1, 1);
        sum.resize(max_length + 1, 1);

        for(int i = 0; i < max_length; i++) {
            num[i + 1] = num[i] * BASE;
            sum[i + 1] = sum[i] + num[i + 1];
        }
    }

    long long to_integer(std::string &s) {
        int len = s.size();
        if (len > max_length)
            return -1;

        long long base = sum[len], val = 0;
        for(int i = len - 1; i >= 0; i--) {
            val *= BASE;
            val += s[i] - 'a';
        }

        return base + val;
    }

    std::string to_string(long long x) {
        int len = 0;
        while (x >= sum[len])
            len++;

        if (len > 0)
            x -= sum[len - 1];

        std::string s = "";
        for(int i = 0; i < len; i++) {
            s += 'a' + x % BASE;
            x /= 26;
        }

        std::reverse(s.begin(), s.end());

        return s;
    }

  private:
    const int BASE = 26;
    // long long で表記できる最大の長さ
    const int max_length = 13;

    // num[i] = i 文字の英小文字からなる文字列の種類数
    // sum[i] = i 文字以下の英小文字からなる文字列の種類数
    std::vector<long long> num, sum;
};
```

## Example

- [ABC 171 C - One Quadrillion and One Dalmatians (C++)](https://atcoder.jp/contests/abc171/submissions/53300108)
