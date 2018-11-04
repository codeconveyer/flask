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
![](https://github.com/codeconveyer/flask/raw/main/picture/first.png)  
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
![](https://github.com/codeconveyer/flask/raw/main/picture/easyframe.png)  
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
class Config:
    SQLALCHEMY_DATABASE_URI = SQLALCHEMY_DATABASE_URI
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    # 密钥
    SECRET_KEY = ""
    # 使用redis,默认127.0.0.1：6379
    SESSION_TYPE = "redis"
    # 修改访问地址
    SESSION_REDIS = redis.Redis(host="",port="")
    # 定义前缀
    SESSION_KEY_PREFIX = 'flask'
```  

之后只需要在create_app(config)函数中通过`app.config.from_object(config)`就可以正常加载设置了。

## views的管理
- 一个项目中往往不只有一个app,那么要对着不同app下的views进行管理的话,就需要用到这里的`Blueprint`了,可以直接从flask重导入

```
from flask import Blueprint
stu = Blueprint('stu', __name__)
@stu.route('/')
def index()
return 'hello world'
```

- 注册蓝图`app.register_blueprint(blueprint=stu, url_prefix='/stu')`这样就可以使用方法了。
- 依次进行创建,create_app函数最终如下
![](https://github.com/codeconveyer/flask/raw/main/picture/main.png)
- 最后在`manage.py`中使用Manager进行管理即可  
```python
from flask_script import Manager
from Base.main import create_app
from Base.config import Config


app = create_app(app)
manager = Manager(app=app)


if __name__ == '__main__':
    manager.run()
```

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
- 之后运行项目,进入createdb方法,会显示出*创建成功*的页面,这是再访问自己的数据库,会发现表被创建了  
![](https://github.com/codeconveyer/flask/raw/main/picture/model.png)

### flask默认session的安全问题
使用过flask进行项目开发的人应该知道,它自带的session是存放在客户端,用户浏览器中的cookie里面的。我们还是先看看falsk是如何对session进行处理的吧
```python
class SecureCookieSessionInterface(SessionInterface):
    """The default session interface that stores sessions in signed cookies
    through the :mod:`itsdangerous` module.
    """
    #: the salt that should be applied on top of the secret key for the
    #: signing of cookie based sessions.
    salt = 'cookie-session'
    #: the hash function to use for the signature.  The default is sha1
    digest_method = staticmethod(hashlib.sha1)
    #: the name of the itsdangerous supported key derivation.  The default
    #: is hmac.
    key_derivation = 'hmac'
    #: A python serializer for the payload.  The default is a compact
    #: JSON derived serializer with support for some extra Python types
    #: such as datetime objects or tuples.
    serializer = session_json_serializer
    session_class = SecureCookieSession

    def get_signing_serializer(self, app):
        if not app.secret_key:
            return None
        signer_kwargs = dict(
            key_derivation=self.key_derivation,
            digest_method=self.digest_method
        )
        return URLSafeTimedSerializer(app.secret_key,
				                              salt=self.salt,
                                      serializer=self.serializer,
                                      signer_kwargs=signer_kwargs)
```
从这段中可以看出,flask指定算法以及secret_key调用了`URLSafeTimedSerializer`类,那么这个类的作用是什么呢?我们需要再看一下
```python
class Signer(object):
    # ...
    def sign(self, value):
        """Signs the given string."""
        return value + want_bytes(self.sep) + self.get_signature(value)

    def get_signature(self, value):
        """Returns the signature for the given value"""
        value = want_bytes(value)
        key = self.derive_key()
        sig = self.algorithm.get_signature(key, value)
        return base64_encode(sig)


class Serializer(object):
    default_serializer = json
    default_signer = Signer
    # ....
    def dumps(self, obj, salt=None):
        """Returns a signed string serialized with the internal serializer.
        The return value can be either a byte or unicode string depending
        on the format of the internal serializer.
        """
        payload = want_bytes(self.dump_payload(obj))
        rv = self.make_signer(salt).sign(payload)
        if self.is_text_serializer:
            rv = rv.decode('utf-8')
        return rv

    def dump_payload(self, obj):
        """Dumps the encoded object. The return value is always a
        bytestring. If the internal serializer is text based the value
        will automatically be encoded to utf-8.
        """
        return want_bytes(self.serializer.dumps(obj))


class URLSafeSerializerMixin(object):
    """Mixed in with a regular serializer it will attempt to zlib compress
    the string to make it shorter if necessary. It will also base64 encode
    the string so that it can safely be placed in a URL.
    """
    def load_payload(self, payload):
        decompress = False
        if payload.startswith(b'.'):
            payload = payload[1:]
            decompress = True
        try:
            json = base64_decode(payload)
        except Exception as e:
            raise BadPayload('Could not base64 decode the payload because of '
                'an exception', original_error=e)
        if decompress:
            try:
                json = zlib.decompress(json)
            except Exception as e:
                raise BadPayload('Could not zlib decompress the payload before '
                    'decoding the payload', original_error=e)
        return super(URLSafeSerializerMixin, self).load_payload(json)

    def dump_payload(self, obj):
        json = super(URLSafeSerializerMixin, self).dump_payload(obj)
        is_compressed = False
        compressed = zlib.compress(json)
        if len(compressed) < (len(json) - 1):
            json = compressed
            is_compressed = True
        base64d = base64_encode(json)
        if is_compressed:
            base64d = b'.' + base64d
        return base64d


class URLSafeTimedSerializer(URLSafeSerializerMixin, TimedSerializer):
    """Works like :class:`TimedSerializer` but dumps and loads into a URL
    safe string consisting of the upper and lowercase character of the
    alphabet as well as ``'_'``, ``'-'`` and ``'.'``.
    """
    default_serializer = compact_json
```
emmm~~说实话我没看懂,在网上查了许多资料,说是对session进行序列化的方法,主要分为了四步,如果有懂的请务必留言指教:
1. json.dumps将对象转换成json字符串
2. 如果数据压缩过后长度更短,则用zlib进行压缩
3. 将数据进行base64编码
4. 通过hmac算法计算数据的签名,将签名附在数据后,用'.'分割  

其实通过这几步我们就知道flask只是对session进行了防篡改的操作,我们可以从客户端的cookie里面获取所有的session信息,而且如果使用的是session的默认处理,懂得源码的人完全是有可能破解session信息的。就例如我在网山寻找到的一段代代码(本人还是个菜鸟=-=)
```python
#!/usr/bin/env python3
import sys
import zlib
from base64 import b64decode
from flask.sessions import session_json_serializer
from itsdangerous import base64_decode

def decryption(payload):
    payload, sig = payload.rsplit(b'.', 1)
    payload, timestamp = payload.rsplit(b'.', 1)

    decompress = False
    if payload.startswith(b'.'):
        payload = payload[1:]
        decompress = True

    try:
        payload = base64_decode(payload)
    except Exception as e:
        raise Exception('Could not base64 decode the payload because of '
                         'an exception')

    if decompress:
        try:
            payload = zlib.decompress(payload)
        except Exception as e:
            raise Exception('Could not zlib decompress the payload before '
                             'decoding the payload')

    return session_json_serializer.loads(payload)
```
额,上述代码完全可以读取出session信息,所以这样是不安全的,但我们可以通过重写这里的方法,使我们的session不那么容易被别人破解,比如自己指定一个序列化方法。
