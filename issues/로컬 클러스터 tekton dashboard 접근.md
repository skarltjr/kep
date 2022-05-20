개요:
```
로컬 k8s 클러스터에서 tekton dashboard를 설치했다.
로컬 클러스터는 vagrant with virtual box가 사용되었는데
tekton pipelines 예제에선 aws + ingress를 사용중이었다.

크게 두 문제가 있었는데
1. 클라우드 서비스 storage를 이용한 storageclass -> https://github.com/skarltjr/kep/blob/main/issues/emptyDir%20%26%20hostpath.md
2. ingress를 활용한 dashboard접근 -> nodePort로 수행
```

```
여기서 nodeport를 활용했는데 localhost:port로 접근하니 안됐었다.
당연하지만.. 그래서 pod describe로 pod가 동작중인 현재 노드와 노드의 ip를 확인후 ip:port로 접근
```
