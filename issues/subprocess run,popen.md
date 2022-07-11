## subprocess run,popen
```
run은 blocking방식
popen은 non-blocking 방식

popen의 경우 제어권이 os에게 존재하기에 popen을 수행하면서 다른 동작이 가능하다
나의 경우 argocd에서 proj가 존재하는지 확인이후 동작을 수행하기에 blocking방식이 적절하다고 생각
```
