```
기존 service yaml 수정하는 방법

apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: MyApp // Selector를 통해 어떤 pod와 서비스 연결할것인지
  ports:
      # 기본적으로 그리고 편의상 `targetPort` 는 `port` 필드와 동일한 값으로 설정된다.
    - port: 80
      targetPort: 80
      # 선택적 필드
      # 기본적으로 그리고 편의상 쿠버네티스 컨트롤 플레인은 포트 범위에서 할당한다(기본값: 30000-32767)
      nodePort: 30007
```

```
apply된 service를 nodeport로 변경하는 방법

`kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```

```
아직 service가 없을 때 yaml말고 간단하게 명령어로 nodeport expose하는 방법

`kubectl expose deployment hello-deployment --type=NodePort --port=80 --target-port=80`
```
