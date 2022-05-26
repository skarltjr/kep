```
pip3 install flask
pip3 install flask-sqlalchemy
pip3 install mysql-connector-python
pip3 install requests







from unicodedata import name
from flask import Flask,request,jsonify,Blueprint
from flask_sqlalchemy import SQLAlchemy



# 
# settings 
#
mydb = {
    'user':'kidol',
    'password':'kidol',
    'database':'flask',
    'host':'localhost',
    'port':3306
}

DB_URL = f"mysql+mysqlconnector://{mydb['user']}:{mydb['password']}@{mydb['host']}:{mydb['port']}/{mydb['database']}?charset=utf8"
user_bp = Blueprint('user', __name__, url_prefix='/user')
app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = DB_URL
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db = SQLAlchemy(app)


#
# model
#
class user(db.Model):
    __tablename__ = "user"
    userId = db.Column(db.Integer,primary_key = True,autoincrement=True)
    username = db.Column(db.String(30))
    password = db.Column(db.String(30))

    def __init__(self,username,password):
        self.username = username
        self.password = password


db.create_all()

#
# api
#
@user_bp.route('/',methods=['POST'])
def createUser():
    req = request.get_json()
    newUser = user(
        username = req['username'],
        password = req['password']
    )
    
    insertUser(newUser)

    return jsonify({
        "message": "유저가 생성되었씁니다."
    })
def insertUser(data):
    db.session.add(data)
    db.session.commit()
    db.session.remove()


@user_bp.route('/<int:userId>',methods=['GET'])
def getUser(userId):
    target = user.query.filter_by(userId=userId).first()
    

    return jsonify({
        "username":target.username,
        "message":"유저 get 성공!"
    })

@user_bp.route('/<int:userId>',methods=['DELETE'])
def deleteUser(userId):
    target = user.query.filter_by(userId=userId).first()

    db.session.delete(target)
    db.session.commit()

    return "success to delete"


@user_bp.route('/<int:userId>',methods=['PUT'])
def updateUSer(userId):
    target = user.query.filter_by(userId=userId).first()

    req = request.get_json()
    target.username = req['username']
    target.password = req['password']
    
    db.session.commit()

    return jsonify({
        "message": "유저가 수정되었씁니다."
    })
 


#
# main
#
if __name__ == '__main__':
    app.register_blueprint(user_bp,name = 'user')
    app.run(port = 5001)
```
