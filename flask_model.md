## Flask中模型的操作

### 基本的数据操作
- 正如我上篇文章中创建的stu表,其中有s_id、s_name、s_age等字段,我需要插入一些数据进去  
插入图片basefield  
```python
stu.route('/addstu/')
def add_stu():
	stu = Student()
	stu.s_name = '于禁'
	db.session.add(stu)
	db.session.commit()
	return '创建成功'
```
- 很简单的一行数据就这样添加成功了,`db.session.commit()`是必须的,不然是不会在mysql中进行创建的,只会有个缓存数据。但是这样明显创建效率不高,当然flask是支持原生sql语句的
```python
sql = 'insert into Student (s_name) values ('于禁')'
db.session.execute(sql)
db.session.commit()
```
这样也可以创建一条数据,而且可以通过sql的多条插入来实现。那么怎么通过原始方法来写呢?我认为将要写的值放入列表,遍历进行添加也是可以的,但是毫无疑问效率是个问题,这就需要在model中加入一个魔法方法,还有`db.session.add_all()`来实现了。首先我来改一下模型吧  
插入图片newmodel  
通过建立一个基础类定义一个共有字段,其他建立的类继承这个类的字段和方法,就跟python的父类、子类之间的继承差不多。通过这个方式创建,我们就可以通过如下方法进行字段数据的修改、插入了  
```python
stu_dict = {
    '张飞': 20,
    '关羽': 22,
    '刘备': 25,
    '貂蝉': 18,
    '吕布': 23,
    '黄月英': 16,
    '小乔': 16
}
for key in stu_dict.keys():
    stu = Student(key, stu_dict[key])
    stu.add_update()
```
这样就可以一个个的添加这一组数据,当然mysql是支持一次性添加一组数据的  
```python
stu_list = []
for key in stu_dict.keys():
    stu = Student(key,stu_dict[key])
    stu_list.append(stu)
db.session.add_all(stu_list)
db.session.commit()
```  
`add_all()`支持添加一组数据  
- 查询数据的方式和django中的查询语句出入不大
	- filter查询  
	`stus = Student.query.filter(Student.s_sex == 1)`
	- filter_by查询
	`stus = Student.query.filter_by(s_sex=1)`
	- get查询,只能索引主键
	`stu = Student.query.get(1)`
	- 在查询时,也可以通过and_或者or_来进行条件的筛选
	`stus = Student.query.filter(and_(Student.s_sex == 1, Student.s_name == "诸葛亮"))`这样筛选出来的是同时满足这两个条件的数据  
	`stus = Student.query.filter(or_(Student.s_age == 18, Student.s_name == "黄月英"))`这样只要是满足其中一个条件的数据就会被筛选出来  
	但是or_和and_只支持filter不支持filter_by
- 修改数据
	- 通过.属性的方式进行修改,只能一条条的修改  
	```python
	stu = Student.query.get(1)
	stu.s_name = '无常'
	db.session.commit()
	```
	- 通过update进行修改,可以同时修改多条数据
	```python
	Student.query.filter(Student.s_sex == 1).update({'s_sex': 1})  
	db.session.commit()
	```
- 删除数据 
```python
stu = Student.query.get(1)
db.session.delete(stu)
db.session.commit()
```
### 关联数据库
- 一对一、一对多关联
	- 一般是在一表中加入`info = db.relationship('StuInfo', backref='student')`  
	多表中加入字段`s_id = db.Column(db.Integer, db.ForeignKey('stu.id'))`,这样就可以建立两表之间的联系  
	学生查学生信息  
	```python
	stu = Student.query.get(1)
	stuinfo = stu.info
	```  
	学生信息查学生  
	```python
	stuinfo = StuInfo.query.get(1)
	stu = stuinfo.student
	```
	需要注意的是,如果在relationship中没有加入backref字段的话,学生依旧可以查询到学生信息,但是通过学生信息就不能找到学生,但是可以通过自己的属性s_id获得学生的id。
	- 一对一只需在relationship中加入`uselist=False`就可以了
- 多对多关联
	- 多对多表的建立需要第三个表来存放这两个表的关系,django中是自动创建的,但是flask中需要我们手动创建  
	```python
	sc = db.Table('sc', db.Column('s_id', db.Integer, db.ForeignKey('stu.id')),
				db.Column('c_id',db.Integer, db.ForeignKey('cou.id'))
	```	
	此外,我们还需要在这两个表中的任意一个中插入一条关系语句  
	`stu = db.relationship('Student', secondary=sc, backref='course')`
	这样我们就成功地建立了多对多关联,这里的secondary就是指定了关联关系表,而关联关系就是在sc表中建立的,如果不写secondary是不能建立多对多关联的,插入数据时会报错。
- 关联数据库的数据插入
	- 一对一、一对多
	数据库在建立的时候,定义了ForeignKey的表会有这个key字段,只需要在插入数据时,给这个key字段一个foreignkey的值就可以成功的建立关联了。
	```python
	info = StuInfo.query.get(1)
	info.s_id = 1
	```  
	这样StuInfo的id为一的数据就和Stu的id为1的数据建立了关联
	- 多对多
	多对多关系建立的时候不会在表中创建关联字段
	```python
	stu = Student.query.get(4)
    cou = Course.query.get(1)
    cou.stu.append(stu)
    db.session.add(cou)
    db.session.commit()
	```
	这样就会在建立的第三个库中建立数据,而且stu和cou的关联也建立好了