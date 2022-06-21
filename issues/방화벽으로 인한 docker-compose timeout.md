1. 문제 :  
```
방화벽으로인해 pip install이 timeout발생
```
2. 해결 :
```
프록시 정보를 추가해준다
pip install --proxy=http://proxy.example.com -r requirements.txt
```

참고 : https://stackoverflow.com/questions/30992717/proxy-awareness-with-pip
