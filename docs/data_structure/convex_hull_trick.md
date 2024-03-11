## c++
```c++
template<bool maximize=false>
struct ConvexHullTrick {
    using ll = long long;
    map<ll,ll> mp;

    void insert(ll a, ll b){
        if(maximize) a = -a, b = -b;

        if(mp.empty()){
            mp[a] = b;
            return;
        }
        else if(mp.size() == 1){
            if(mp.count(a)) chmin(mp[a], b);
            else mp[a] = b;
            return;
        }

        bool add = false;

        // 端
        if(a < (*mp.begin()).first or (*mp.rbegin()).first < a){
            mp[a] = b;
            add = true;
        }
        // 同じ傾きが存在
        else if(mp.count(a)){
            if(chmin(mp[a], b)) add = true;
        }
        else{
            pair<ll,ll> l = *mp.upper_bound(a), r = *mp.lower_bound(a);

            if(_judge(l, {a,b}, r)){
                mp[a] = b;
                add = true;
            }
        }

        if(add) remove(a);
        return;
    }

    void remove(ll a){
        // +
        {
            auto it = mp.lower_bound(a); ++it;

            while((*it).first != (*mp.rbegin()).first){
                --it;
                pair<ll,ll> l = *it;
                ++it;
                pair<ll,ll> m = *it;
                ++it;
                pair<ll,ll> r = *it;

                if(_judge(l,m,r)) break;

                mp.erase(m.first);
            }
        }
        // -
        {
            auto it = mp.upper_bound(a);
            while((*it).first != (*mp.begin()).first){
                ++it;
                pair<ll,ll> r = *it;
                --it;
                pair<ll,ll> m = *it;
                --it;
                pair<ll,ll> l = *it;

                if(_judge(l,m,r)) break;

                mp.erase(m.first);
            }
        }
        return;
    }

    bool _judge(pair<ll,ll> l, pair<ll,ll> m, pair<ll,ll> r){
        // mが必要 -> true
        ll a,b,al,bl,ar,br;
        tie(al,bl) = l; tie(a,b) = m; tie(ar,br) = r;

        return (a-al)*(br-bl) > (ar-al)*(b-bl);
    }

    ll query(ll x){
        if(mp.size() == 1){
            auto it = *mp.begin();
            ll a,b; tie(a,b) = it;
            return a*x + b;
        }

        auto itl = mp.begin(), itr = prev(mp.end());

        auto check = [&](){
            if(itl == itr) return false;
            itl++;
            ll p = (*itl).first, q = (*itr).first;
            itl--;
            return p != q;
        };

        while(check()){
            ll al,bl,ar,br; tie(al,bl) = *itl; tie(ar,br) = *itr;
            ll m = (al+ar)/2;
            auto it1 = mp.upper_bound(m);
            auto it0 = prev(it1);

            ll a0,b0,a1,b1; tie(a0,b0) = *it0; tie(a1,b1) = *it1;
            if(a0*x+b0 < a1*x+b1) itr = it0;
            else itl = it1;
        }

        ll al,bl,ar,br; tie(al,bl) = *itl; tie(ar,br) = *itr;
        ll ret = min(al*x+bl, ar*x+br);
        return ret;
    }
};
```
