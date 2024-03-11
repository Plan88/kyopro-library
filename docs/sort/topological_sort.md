## c++
```c++
stack<int> stk;
for(int i=0; i<N; i++) if(deg[i] == 0) stk.push(i);
    
vector<int> sorted;
while(stk.size()){
    int i = stk.top(); stk.pop();
    sorted.push_back(i);
    for(int j : e[i]){
        deg[j]--;
        if(deg[j] == 0) stk.push(j);
    }
}
```
