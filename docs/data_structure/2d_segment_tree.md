## c++
```c++
template<class S, S (*op)(S, S), S (*e)()>
struct SegmentTree2D{
    public:
        int H, W, szH, szW;
        vector<vector<S>> val;

        SegmentTree2D(): SegmentTree2D(0, 0) {}
        SegmentTree2D(int H, int W): SegmentTree2D(vector<vector<S>>(H, vector(W, e()))) {}
        SegmentTree2D(const vector<vector<S>>& v): H(v.size()), W(v.empty() ? 0 : v[0].size()) {
            szH = 1, szW = 1;
            while(szH < H) szH <<= 1;
            while(szW < W) szW <<= 1;

            val.resize(szH<<1, vector<S>(szW<<1, e()));
            for(int i=0; i<H; i++)for(int j=0; j<W; j++) val[szH + i][szW + j] = v[i][j];
            for(int i=(szH<<1)-1; i>0; i--)for(int j=(szW<<1)-1; j>0; j--) update(i, j);
        }

        void set(int x, int y, S v){
            assert(0 <= x and x<H and 0 <= y and y<W);

            x += szH, y += szW;
            val[x][y] = v;

            for(int i=x; i>1; i>>=1)for(int j=y; j>1; j>>=1){
                if(i==x and j==y) continue;
                update(i, j);
            }
        }

        S get(int i, int j){
            assert(0 <= i and i<H and 0 <= j and j<W);

            i += szH, j += szW;
            return val[i][j];
        }

        S prod(int lh, int rh, int lw, int rw){
            lh += szH, rh += szH, lw += szW, rw += szW;
            S vlh = e(), vrh = e();

            while(lh < rh){
                if(lh & 1){
                    S vlw = e(), vrw = e();
                    int tmp_lw = lw, tmp_rw = rw;
                    while(tmp_lw < tmp_rw){
                        if(tmp_lw & 1) vlw = op(vlw, val[lh][tmp_lw++]);
                        if(tmp_rw & 1) vrw = op(val[lh][--tmp_rw], vrw);
                        tmp_lw >>= 1;
                        tmp_rw >>= 1;
                    }
                    vlh = op(vlh, op(vlw, vrw));
                    lh++;
                }
                if(rh & 1){
                    --rh;
                    S vlw = e(), vrw = e();
                    int tmp_lw = lw, tmp_rw = rw;
                    while(tmp_lw < tmp_rw){
                        if(tmp_lw & 1) vlw = op(vlw, val[rh][tmp_lw++]);
                        if(tmp_rw & 1) vrw = op(val[rh][--tmp_rw], vrw);
                        tmp_lw >>= 1;
                        tmp_rw >>= 1;
                    }
                    vrh = op(op(vlw, vrw), vrh);
                }
                lh >>= 1;
                rh >>= 1;
            }

            return op(vlh, vrh);
        }

    private:
        void update(int i, int j){
            if(szH <= i and szW <= j) return;
            S v1 = op(_get(i, 2*j), _get(i, 2*j+1));
            S v2 = op(_get(2*i, j), _get(2*i+1, j));
            val[i][j] = op(v1, v2);
        }

        S _get(int i, int j){
            if(i >= 2*szH or j >= 2*szW) return e();
            return val[i][j];
        }
};
```
