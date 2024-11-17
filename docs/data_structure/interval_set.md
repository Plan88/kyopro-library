## C++
```c++
template<class T = int>
struct IntervalSet {
    using I = std::pair<T, T>;
    IntervalSet(T neg_inf): neg_inf(neg_inf) {}

    void insert(T l, T r) {
        T new_l = l, new_r = r;
        auto itr = st.lower_bound(I(l, neg_inf));
        while(itr != st.end() and (*itr).second <= new_r) {
            new_l = std::min(new_l, (*itr).second);
            new_r = std::max(new_r, (*itr).first);
            st.erase(itr++);
        }
        st.insert(I(new_r, new_l));
    }

    void erase(T l, T r) {
        auto itr = st.lower_bound(I(l, l));
        std::vector<I> v;
        while(itr != st.end() and (*itr).second < r) {
            v.push_back(*itr);
            st.erase(itr++);
        }
        for(auto [s, e] : v) {
            if(l <= s and e <= r) continue;
            if(s < l) st.insert(I(l, s));
            if(r < e) st.insert(I(e, r));
        }
    }

    size_t size() {
        return st.size();
    }

    bool is_covered(T x) {
        // x がある区間で覆われているか
        auto itr = st.lower_bound(I(x, x));
        if(itr == st.end()) return false;
        T l = (*itr).second, r = (*itr).first;
        return l <= x and x < r;
    }

    I is_covered_by(T x) {
        // x を覆っている区間, 存在しなければ [neg_inf, neg_inf) を返す
        auto itr = st.lower_bound(I(x, x));
        if(itr == st.end()) return I(neg_inf, neg_inf);
        T l = (*itr).second, r = (*itr).first;
        if(l <= x and x < r) return I(l, r);
        return I(neg_inf, neg_inf);
    }
  private:
    std::set<I> st;
    T neg_inf; // -inf
};
```
