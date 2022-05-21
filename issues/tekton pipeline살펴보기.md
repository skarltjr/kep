환경 : 
```
로컬에서 vagrant - virtual box를 활용한 쿠버네티스 클러스터 구축
따라서 클라우드 서비스를 활용한 storage class & ingress 설정등이 어려웠고 
storage class -> vm을 활용한 hostPath사용
ingress -> nodePort로 tekton dashboard접근
```

### tekton이란
```
tekton은 ci/cd 파이프라인을 빠르게 구축하기 위한 프레임워크를 제공하는 cloud-native 오픈소스 프로젝트
tekton의 구성 요소들은 k8s custom resource definitions로 정의되어 다른 pod나 resource와 같이
k8s cli api call로 사용할 수 있다.
```

### tekton의 특징
1. 재사용 : tekton의 모든 task들은 다른 pipeline과 완전히 독립적으로 사용할 수 있다. 
즉 모듈호하가 잘 되어있어 여러 pipeline에서 필요한 task들을 갖다 쓸 수 있다.
2. 표준화 : tekton은 k8s의 custom resource를 사용해 정의
3. 확장성 : tekton hub를 통해 이미 제작된 여러 task를 가져다 사용할 수 있다. github actions의 market place처럼


### tekton 구조
- task
- taskRun
- pipeline
- pipelineRun


### tekton 구조 more
- step : 하나의 작업. ex) git clone, compile 등..
- task : step의 모음. task하나 당 하나의 pod으로 동작
- taskRun : task를 실행시키는 역할로 serviceAccount, resource, 특정 pod를 설정할 수 있다.
  - ex) : docker-registry push할 때 필요한 계정 secret - serviceAccount로 사용
- pipeline : task의 모음 / runAfter 구문으로 이전 task를 설정하여 그 다음 동작하도록 가능
- pipelineRun : 
  - pipeline을 실행. 
  - workspace라는 이름으로 task간 볼륨도 공유 가능
  - teskRun과 마찬가지로 serviceAccount, params, pod설정 등 가능
  - 참고로! pipelineRun을 실행시키면 각 task에 해당하는 taskRun을 자동으로 생성시켜 실행해서 별도 taskRun필요없음
- workspace : task들의 볼륨
  - 1 = git clone
  - 2 = build
  - 라는 task가 있을 때 workspace를 통해 볼륨을 공유하여 1에서 clone한 repo를 바탕으로 2에서 build하는 등 수행
  - workspace는 pvc / emptyDir 등을 붙여줘서 볼륨을 잡는다

### 예제
- 목표 : git repo clone -> build -> image 생성 및 registry push
```
# Tekton pipeline v0.29.0 설치
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.29.0/release.yaml

# Tekton dashboard v0.14.0 설치
curl -O https://storage.googleapis.com/tekton-releases/dashboard/previous/v0.14.0/tekton-dashboard-release.yaml
- 여기서 issue : https://github.com/skarltjr/kep/blob/main/issues/로컬%20클러스터%20tekton%20dashboard%20접근.md
- 다운 후 apply

# Tekton triggers v0.11.2 설치 
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/previous/v0.11.2/release.yaml
```
- 과정 및 구조 파악 : 
```
1. 추후 github에있는 manifest파일 수정을 위한 git secret
apiVersion: v1
kind: Secret
metadata:
  name: git-secret
  annotations:
    tekton.dev/git-0: https://github.com
type: kubernetes.io/basic-auth
stringData:
  username: *** 본인거
  password: ***
```
```
2. 해당 secret을 활용하여 pipelineRun에서 활용하기위한 git serviceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: git-sa
# 위에서 생성한 secret의 name을 등록
secrets:
  - name: git-secret
```
```
3. docker registry push를 위한 secret & serviceAccount
- issue : https://github.com/skarltjr/kep/blob/main/issues/docker-secret.md

> secret 생성
kubectl create secret docker-registry docker-secret2\ 
  --docker-username=<username>\
  --docker-password=<password>\
  --docker-email=<email>
  
> sa 생성
- issue : https://github.com/skarltjr/kep/blob/main/issues/docker-secret.md

apiVersion: v1
kind: ServiceAccount
metadata:
  name: docker-sa

# 위에서 생성한 secret의 name을 등록
secrets:
  - name: docker-secret2
```
```
4. 위에것들 apply
```
```
5. pv & pvc생성
- pipelineRun에서 pipeline의 task수행 및 task들간 공유를 위한 볼륨인 workspace를
```
