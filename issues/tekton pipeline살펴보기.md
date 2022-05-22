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
 - https://frozenpond.tistory.com/111?category=1209055
 - crd란 custom api라고 볼 수 있을 거 같다.
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
- pipelineRun에서 pipeline의 task수행 및 task들간 공유를 위한 볼륨인 workspace를 위해 볼륨을 잡아줘야한다
- 그래서 storageclass 혹은 pv - hostPath등의 방법으로 공간을 마련
- pipelineRun은 pvc로 공간 요청

> pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: standard
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  capacity:
    storage: 5Gi
  hostPath:
    path: /data
    
> pvc
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 5G
storageClassName을 맞춰서 pvc가 pv를 할당받을 수 있도록한다.
```

```
pipeline구성 전!!
tekton catalog를 활용한 task

tekton catalog란?
미리 정의된 다양한 ttask들을 다운받을 수 있는 공간 / marketplace
ex) git-clone, docker-build 등


> git clone을 위한 task 다운
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.5/git-clone.yaml

> buildah를 통한 docker image build & push를 위한 task다운
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/buildah/0.3/buildah.yaml 

```
```
5. pipeline

apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: cicd-pipeline
spec:
  workspaces:
    - name: pipeline-shared-data
  tasks:

    # git clone
    - name: clone-repository
      # Tekton catalog를 통해 다운받은 task의 파라미터 적용
      params:
        - name: url
          value: https://github.com/skarltjr/ci_cd_test.git
        - name: revision
          value: "main"
        - name: deleteExisting
          value: "true"
      # 참조하는 task 입력
      taskRef:
        kind: Task
        name: git-clone
      # 공간 설정(다양한 설정이 있으니 Tekton Hub 참조)
      workspaces:
        - name: output
          workspace: pipeline-shared-data

    # buildha
    - name: build-image
      # task 순서를 정하기 위한 설정
      runAfter:
        - clone-repository
      params:
        - name: IMAGE
          value: "skarltjr/tekton:$(tasks.clone-repository.results.commit)"
      taskRef:
        kind: Task
        name: buildah
      workspaces:
        - name: source
          workspace: pipeline-shared-data
```
```
6. pipelineRun
  
  
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: pipeline-run
spec:
  # pipeline을 수행하면서 필요한 Service Account 계정을 설정
  # git과 docker 접근하기 위한 설정
  serviceAccountName: docker-sa
  pipelineRef:
    name: cicd-pipeline
  workspaces:
    - name: pipeline-shared-data
      # 위에서 생성한 pvc의 name 등록
      persistentvolumeclaim:
       claimName: pvc
```
