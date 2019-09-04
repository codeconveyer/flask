## flask_sqlachemy使用
我在[flask_model](https://github.com/codeconveyer/flask/blob/main/flask_model.md)中介绍了flask_sqlalchemy中的基本操作,这里在介绍一些更细节一些的操作.
1. 从数据库对象中只取出需要的字段  
    &emsp;&emsp;`Stu.query.filter`实际上是使用的sql`select * from stu where`,这样将所有字段取出,如果你在某个接口中只需要name的话就会导致多余字段占用额外内存从而一定程度影响效率(当然是多余字段很多的情况下)。有三个方法可以实现获取相应字段:  
    &emsp;&emsp;第一种是: `db.session.query(Stu.id, Stu.name)`  
    &emsp;&emsp;第二种是: `Stu.query.with_entities(Stu.id, Stu.name)`  
    &emsp;&emsp;第三种是: `Stu.query.values(Stu.id, Stu.name)`  
    &emsp;&emsp;前两者在不输入`.first`或者`.all`时候不会执行数据库操作,执行之后的相应输出格式为`(id, name)`和`[(id, name), (id, name)]`;而第三者就直接进行了数据库操作,他会将所有的结果放入一个迭代器中,表现形式为一个元祖。  
    &emsp;&emsp;这两种的输出都是一个类似元组的result,这样就会有一个问题,假如你是想要一个字段的值的话,他依旧会给你一个元祖类似`(1,)`,这样的话就需要再做一步处理,例如`[i[0] for i in result]`或者`[i.id for i in result]`(因为他其实是一个result对象,所以支持获取属性的操作)。  
2. filter和filter_by的一些操作  
    - `filter`和`filter_by`的区别  
    在于`filter_by`会默认选择你的语句中最后一个数据库对象作为他的`from table`来使用,例如`session.query(Stu1, Stu2).filter_by(id=1)`翻译为sql则是`select stu1.*, stu2.* from stu1, stu2 where stu2.id=1`,
    还有`join`操作会影响`filter_by`,比如`Stu1.query.join(Stu2).filter_by`中就是`where stu2`来进行搜索。  
    - 默认的链接符  
    `object.query.filter()`中默认使用的是`and`操作,如果你想使用`or`进行搜索的话,需要`from sqlalchemy import or_`然后将你需要的条件`or_(obj.id==1)`,其实一条语句中你可以链接多条`filter().filter()`,而且最终他们都是用`and`进行连接的  
    - 在`filter`中使用原始语句  
    有时候`sqlalchemy`并不能满足我们的搜索条件,例如数据库ARRAY类型的`@>`操作等,虽然有方法但总是不能好好运行,这时候我们就需要用到`text`来帮助我们输入原始的搜索条件,例如`text("stu.menbers@>'{1}'")`  
3. 使用数据库的函数  
&emsp;&emsp;`sqlalchemy`将所有的数据库函数都放在了`func`中,例如我们想要实现`select max(id) from stu`就可以用`session.query(func.max(obj.id))`来实现,当然它的使用是基于数据库的,所以如果数据库并没有你给出的函数,那么他会给你个错误
