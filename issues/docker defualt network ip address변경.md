개요 : 
```
도커 기본 ip 대역이 vpn 대역과 겹쳐서 vm을 사용할 수 없게되었다.
이전에는 잘 돌아갔는데 갑자기 안된다.

그래서 도커 기본 브릿지 docker0의 ip를 변경해주려고한다.
- 일단 도커 네트워크 인터페이스 중 기본 생성되는 docker0가 vpn 대역과 겹쳤고 이를 변경
- 해결된줄알았는데 똑같이 동작 도중 튕기고 vm 접근 불가
```
내 생각 :
- <img width="960" alt="스크린샷 2022-06-27 오후 6 00 20" src="https://user-images.githubusercontent.com/62214428/175901641-5e789d22-8b3b-4e19-827d-a4649b422a1e.png">
```
도커 기본 네트워크가 vpn 대역과 겹친다면 별도의 네트워크 생성 후 컨테이너를 해당 네트워크에서 실행시키면 해결할 수 있지않을까???
애초에 처음 도커가 설치될 때 docker0이 vpn과 같은 대역으로 생성되는거 자체가 문제인가???

todo 해보기
```


- 참고 : https://thisstory.tistory.com/entry/Docker0-default-network-IP-address-변경-docker-브릿지-기본IP-주소-변경
```
그런데 생각해보니 지금 도커 설치할때도 문제가 생겨서 멈추고 vm 튕긴다.
```
