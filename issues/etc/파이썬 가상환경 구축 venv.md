참고 : https://docs.python.org/ko/3.7/tutorial/venv.html

1. 가상환경 구축
```
python3 -m venv 가상환경이름

서로 다른 환경에서 개발해도 문제없이 진행해보자
```

2. 가상환경 실행
```
source tutorial-env/bin/activate

이제 하나의 가상환경이 실행된 것
```

3. a 가상환경에서 설치한 패키지는 b 가상환경엔 없으니 b에선 실행이 안될 것
```
a 가상환경에서 pip3 install flask

b에서 import해보면 x!!
```
