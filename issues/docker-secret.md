개요 : 
```
tekton ci에서 docker-registry에 이미지 푸쉬를 위해 secret & service account 생성 후 pipelineRun에 serviceaccount적용했을 때 자꾸 
auth fail발생



apiVersion: v1
kind: Secret
metadata:
  name: docker-secret
  annotations:
    tekton.dev/docker-0: https://gcr.io # Described below
type: kubernetes.io/basic-auth
stringData:
  username: ***
  password: ***
  
  
apiVersion: v1
kind: ServiceAccount
metadata:
  name: docker-sa

# 위에서 생성한 secret의 name을 등록
secrets:
  - name: docker-secret
  
  
따라서 
kubectl create secret docker-registry docker-secret\ 
  --docker-username=<username>\
  --docker-password=<password>\
  --docker-email=<email>
 
방식으로 secret2를 만들어 적용해보니 성공!
```
