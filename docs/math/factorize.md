## c++
```c++
vector<pair<long long, int>> factorize(long long n){
    vector<pair<long long, int>> res;
    
    if(n%2 == 0){
        res.emplace_back(2,0);
        while(n%2 == 0){
            n /= 2;
            res.back().second++;
        }
    }
    
    for(long long  i=3; i*i<=n; i+=2){
        if(n%i) continue;
        res.emplace_back(i,0);
        while(n%i == 0){
            n /= i;
            res.back().second++;
        }
    }
    if(n != 1) res.emplace_back(n,1);
    return res;
}
```

## Rust
```Rust
fn factorize(mut x: u64) -> Vec<(u64, u64)> {
    let mut fact = vec![];
    let sqrt = x.sqrt();

    let mut divide = |p| {
		let mut count = 0;
		while x % p == 0 {
			x /= p;
			count += 1;
		}
		if count > 0 {
			fact.push((p, count));
		}
    };

    divide(2);

    for p in (3..=sqrt).step_by(2) {
		divide(p);
    }

    if x > 1 {
		fact.push((x, 1));
    }

    fact
}
```
