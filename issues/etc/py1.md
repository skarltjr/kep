```
class Unit:
    # 생성자
    def __init__(self,name,hp,speed): # 멤버 변수
        self.name = name
        self.hp=hp
        self.speed = speed

    def move(self):
        print(f"{self.speed}속도로 웁직입니다")
# 상속
class AttackUnit(Unit):
    def __init__(self, name, hp,damage,speed):
        Unit.__init__(self,name,hp,speed)
        self.demage = damage

    # 멤버 함수 / 항상 self를 파라미터로 갖는다
    def attack(self,location):
        if location != None:
            self.hp /= 2
            print(f"{location} 지상 공격 중 ")

class Flyable:
    def __init__(self,flying_speed):
        self.flying_speed = flying_speed

    def fly(self,location):
        print(f"{location}부터 {self.flying_speed}속도로 날아서 이동중입니다.")

# 다중 상속
class FlyableUnit(AttackUnit,Flyable):
    def __init__(self, name, hp, damage,speed):
        AttackUnit.__init__(self,name,hp,damage,speed)
        Flyable.__init__(self,speed)

    # 오버라이딩
    def move(self,location):
        self.fly(self,location)

    def attack(self,location):
        if location != None:
            self.hp /= 2
            print(f"{location} 공중 공격 중 ")


# --------

# 객체 생성
myUnit = Unit("kidol",100,100)
# 파이썬은 null대신 none이 있다.
myUnit1 = Unit("kiseok",None,100)
myUnit2 = AttackUnit("kidol",100,150,100)

# 멤버 변수 접근
print(myUnit1.hp)
myUnit2.name = " new Name"
print(myUnit2.name)

# 추가 변수를 외부에서 만들어서 사용할 수 있다.
myUnit2.age = 10
print(myUnit2.age)
# 다만 이 경우 myUnit2만 해당 변수를 갖는것이지 나머지 객체는 적용 x


# 멤버 함수 사용
print(f"origin hp = {myUnit2.hp}")
myUnit2.attack("home")

# 다중 상속 객체 활용
print("1")
valkiri = FlyableUnit("valk",100,200,180)
valkiri.fly("home")
valkiri.attack("home")
```
