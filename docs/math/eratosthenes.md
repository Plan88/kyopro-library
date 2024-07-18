## C++
```c++
struct SieveEratosthenes{
    std::vector<int> min_fact;

    SieveEratosthenes(int N){
        _build(N);
    }

    void _build(int N){
        min_fact.resize(N+1);
        for(int i=0; i<=N; i++) min_fact[i] = i;
        
        min_fact[0] = min_fact[1] = -1;

        for(int i=2; i*i<=N; i++){
            for(int j=i*i; j<=N; j+=i){
                if(i < min_fact[j])
                    min_fact[j] = i;
            }
        }
    }

    bool is_prime(int x){
        return min_fact[x] == x;
    }

    std::vector<std::pair<int,int>> factorize(int x){
        std::vector<std::pair<int,int>> ret;
        while(x>1){
            if(ret.empty() || ret.back().first != min_fact[x])
                ret.emplace_back(min_fact[x], 1);
            else
                ret.back().second++;

            x /= min_fact[x];
        }

        return ret;
    }
};
```

## Rust
```rust
mod math {
    use itertools::Itertools;
    use num_integer::Roots;
    pub struct SieveEratosthenes {
        min_fact: Vec<u32>,
    }

    impl SieveEratosthenes {
        pub fn new(n: usize) -> Self {
            Self::build(n)
        }

        fn build(n: usize) -> Self {
            let mut min_fact = (0..=n as u32).collect_vec();
            min_fact[1] = 0;

            for i in 2..=n.sqrt() {
                for j in (i * i..=n).step_by(i) {
                    min_fact[j] = min_fact[j].min(i as u32);
                }
            }
            Self { min_fact }
        }

        pub fn is_prime(&self, x: u32) -> bool {
            if x < 2 {
                false
            } else {
                self.min_fact[x as usize] == x
            }
        }

        pub fn factorize(&self, mut x: u32) -> Vec<(u32, usize)> {
            let mut ret = vec![];

            while x > 1 {
                if ret.last().unwrap_or(&(0, 0)).0 == self.min_fact[x as usize] {
                    ret.last_mut().unwrap().1 += 1;
                } else {
                    ret.push((self.min_fact[x as usize], 1));
                }
                x /= self.min_fact[x as usize];
            }

            ret
        }
    }
}
```
