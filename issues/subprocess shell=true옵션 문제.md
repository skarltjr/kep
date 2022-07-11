### subprocess shell=true옵션 문제
```
shell=true를 적용하는 경우
"argocd login argocd-dev.kakaoenterprise.in --username xxx --password xxx; argocd proj get xxx"
명령어를 그대로 실행할 수 있다. 

그러나 
1. 명령어 str에서 공백 등 문제등이 모두 개발자의 책임 / 하드코딩의 문제

2. shell=True 를 사용할 때, 중간에 subprocess가 /bin/sh 를 통해서 실행된다.
ex) 기존 pid = 3303으로 python이 실행되었을 때 shell옵션으로 생성된 /bin/sh가 3303의 pid를 차지하게되고 기존 3303 pid를 가진 python server는 다른 pid를 갖게된다
예상으로는 그래서 python server에게 명령어 수행결과가 올바르게 전달되지 않았고 argocd proj가 존재함에도 없다는 결과를 반환한것같다.
```
