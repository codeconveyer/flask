# flask项目

## 虚拟环境的创建
- 创建虚拟环境
我们在使用pycharm启动一个新项目时,如果我们没有指定一个虚拟环境,那么pycharm会自动在这个项目下创建一个虚拟环境,这样的效果往往是不如人意的,而且有些项目是可以公用一个环境的,所以我们可以自行创建一个虚拟环境。
- [创建虚拟环境的三种方法](https://segmentfault.com/a/1190000005828284)
	- conda 创建虚拟环境
	- virtualenv 创建虚拟环境
	- python -m 创建虚拟环境

## 建立项目
打开pycharm在里面建立一个flask项目,解释器要选择自己创建的虚拟环境中的python  
显示图片first  
这样我们的一个最基本的flask项目就建立好了,默认端口是5000。可以通过在`app.run(debug,port)`进行配置修改端口和开启调试模式。  
可以通过修改configurations或者在控制台中输入`python xx.py`启动项目。  
这样看我们的flask项目建立非常简单,但是将方法、配置与启动放在一起总会显得臃肿。项目中最好是将不同功能放入不同包中,所以我们需要对项目进行整理。

## 结构调整  
既然要进行结构调整,我们就需要一些第三方库的来帮助我们,需要`pip install flask_script`,之后我们使用这个模块中的Manager对我们的app进行管理  
```
manager = Manager(app=app)
manager.run()
```  
进行如上操作之后我们就可以通过`python xx.py runserver -p 8080 -d`运行项目,当然最好是将这个配置放入configurations中。  
我在上面也说了,最好是不同的功能放入不同的文件夹或者文件中,而我们的flask项目需要使用页面,js、css,接口说明,所以我们可以建立这几个文件夹

- templates 页面管理 默认创建
- static 静态方法css、js 默认创建
- docs 接口说明
- utils 工具  

而我们也需要将方法,设置等放入不同的py文件中,manage.py放置在最外面  
显示图片easyframe  
当然这只是一个简单的架构,各位可以自行斟酌自己的架构,思想是自由的。

## 数据库的使用
- 这里我使用mysql作为我的数据库。flask使用数据库的内容是需要配置的  
	- `app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://user：password@host:port/name?charset=utf8`设置数据库链接信息,`?charset`不写的话默认'utf8'
	- `app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN'] = False`必须设置;设置为True的话,将会追踪对象的修改并且发送信号。这需要额外的内存
- 这里的数据库配置过于冗长,我们可以通过定义一个字典来保存这些数值,再定义一个字符串链接的函数也是可以的  
```
DATABASE={'DB':'','DRIVER':'','USER':'','PASSWORD':'','HOST':'','PORT':'','NAME':'',
}
```
```
def get_db_uri(DATABASE):  
	user = DATABASE.get('user)  
	password = BATABASE.get('password')  
	...  
	return "{}+{}://{}:{}@{}:{}/{}.format(db,driver...)"
```
- 然后需要`pip install flask-sqlalchemy`用这第三方库的内容来管理和使用我们的本地数据库。这个第三方方法是要进行初始化的,而且如果还有其他第三方库——`DebugToolbarExtension`、`Api`、`Marshmallow`等的内容的话,依旧是需要初始化,而且他们初始化的方式是一样的,所以我们可以定义一个方法  

```
db = SQLAlchemy()
api = Api()
def init_ext(app):
	db.init__app(app=app)
	api.init__app(app=app)
```

- 通过上述的步骤,我们就可以在自己的flask项目中使用自己的数据库了。但是这样的配置也是不够好的,因为我们有时会需要用redis来作为我们的数据存储,这样我们又会多几行`app.config`字段,所以我们可以将这些config也放入一个包中,在这个包中建一个类  

```
class Config
	SQLALCHEMY_DATABASE_URI = SQLALCHEMY_DATABASE_URI
    SQLALCHEMY_TRACK_MODIFICATIONS = False
	密钥
	SECRET_KEY
	使用redis,默认127.0.0.1：6379
	SESSION_TYPE = "redis"
	修改访问地址
	SESSION_REDIS = redis.Redis(host=,port=)
	定义前缀
	SESSION_KEY_PREFIX = "flask"
```  

之后只需要在create_app(config)函数中通过`app.config.from_object(config)`就可以正常加载设置了。

## views的管理
- 一个项目中往往不只有一个app,那么要对着不同app下的views进行管理的话,就需要用到这里的`Blueprint`了,可以直接重flask重导入

```
from flask import Blueprint
stu = Blueprint('stu', __name__)
@stu.route('/')
def index()
return 'hello world'
```

- 注册蓝图`app.register_blueprint(blueprint=stu, url_prefix='/stu')`这样就可以使用方法了。

## 一个简单的创建
- 在models中
```
from Base.function import db


class Student(db.Model):
    s_id = db.Column(db.Integer, primary_key=True)
    s_name = db.Column(db.String(20))
    s_age = db.Column(db.Integer)

    __tablename__ = 'stu'

```

- 在views中
```
from flask import Blueprint, render_template
from Stu.models import db

stu = Blueprint('stu', __name__)


@stu.route('/')
def index():
    return render_template('hello.html')


@stu.route('/createdb/')
def create_db():
    db.create_all()
    return '创建成功'


@stu.route('/dropdb/')
def drop_db():
    db.drop_all()
    return '删除成功'

```
- 之后运行项目,进入createdb方法,会显示出#创建成功#的页面,这是再访问自己的数据库,会发现表被创建了  
显示图model