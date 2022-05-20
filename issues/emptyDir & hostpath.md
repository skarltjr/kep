개요 
```
로컬에서 쿠버네티스 클러스터 구축 후 storage class 붙이려다 팀원분이 emptydir나 hostpath 활용이 더 좋아보인다고 해주셨다
그래서 이들을 좀 더 알아보고자한다.
```

### emptyDir
```
쉽게말해 하나의 빈 volume을 위한 공용 파드를 생성한다.

아래 그림을 보면 
하나의 파드를 volume으로 다루고
여러 컨테이너가 같은 volume에 데이터를 공유한다.
이 경우 파드가 삭제되면 볼륨도 사라지니 일시적인 데이터만 허용한다
```
- ![스크린샷 2022-05-20 오후 1 47 44](https://user-images.githubusercontent.com/62214428/169452558-95b27f01-b7de-4121-a7e1-5a336d07e936.png)
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-1
spec:
  containers:
  - name: container1
    image: kubetm/init
    volumeMounts:
    - name: empty-dir
      mountPath: /mount1
  - name: container2
    image: kubetm/init
    volumeMounts:
    - name: empty-dir
      mountPath: /mount2
  volumes:
  - name : empty-dir
    emptyDir: {}
```

### hostPath
```
파드들이 올라간 노드의! path를 볼륨으로 사용. 
즉 노드의 자원을 활용한다.
다만 a노드의 볼륨은 b노드에 생성된 파드가 접근할 수 없다.
```
```
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-3
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: host-path
      mountPath: /mount1
  volumes:
  - name : host-path
    hostPath:
      path: /node-v
      type: DirectoryOrCreate
```

### 판단
```
현재 일시적인 실습 상황
node-1, node-2가 존재
emptyDir 파드를 하나 만들어서 volume잡고 pvc 활용하는게 더 좋다고 생각
```