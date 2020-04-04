## 章节1

### 创建虚拟环境

virtualenv  名字

cd  名字

cd Scripts

进入 activate

退出 deactive

### 虚拟环境virtualenvwrapper

安装： pip install virtualenvwrapper-win

创建虚拟环境：mkvirtualenv  xxx

切换到某个虚拟环境：workon xxx

退出当前虚拟环境：deactivate

删除某个虚拟环境：rmvirtualenv xxx

列出所有虚拟环境：lsvirtualenv

### 配置文件

Flask  项目的配置，都是通过 app.config  对象来进行配置的。比如要配置一个项目处
于 DEBUG  模式下，那么可以使用 app.config['DEBUG] = True  来进行设置，那么 Flask  项目将
以 DEBUG  模式运行。在 Flask  项目中，有四种方式进行项目的配置

- ​    app = Flask(__name__)
  ​	app.config['DEBUG'] = True

- 因为 app.config  是 flask.config.Config  的实例，而 Config  类是继承自 dict  ，因此可以
  通过 update  方法：

  app.config.update(
  DEBUG=True,
  SECRET_KEY='...'）

- 如果你的配置项特别多，你可以把所有的配置项都放在一个模块中，然后通过加载模块的方式
  进行配置，假设有一个 settings.py  模块，专门用来存储配置项的，此时你可以通
  过 app.config.from_object()  方法进行加载，并且该方法既可以接收模块的的字符串名称，
  也可以模块对象：

   import settings
   app.config.from_object(settings)

- 也可以通过另外一个方法加载，该方法就是 app.config.from_pyfile()  ，该方法传入一个文
  件名，通常是以 .py  结尾的文件，但也不限于只使用 .py  后缀的文件

  app.config.from_pyfile('settings.py',silent=True)

### url与试图

#### url与函数的映射

- string: 默认的数据类型，接受没有任何斜杠 /  的字符串。
- int: 整形
- float: 浮点型。
- path： 和 string  类似，但是可以传递斜杠 /  。  xxx：8000/a/b/c/

​		当有多个子域名的时候  也可以一起接收

- uuid：  uuid  类型的字符串。

- any：可以指定多种路径，这个通过一个例子来进行说明:

  @app.route('/<any(article,blog):url_path>'>)

  ​	article/... 与blog/...  可以指向一个试图

 如果不想定制子路径来传递参数，也可以通过传统的 ?=  的形式来传递参数，例如： /article?
id=xxx  ，这种情况下，可以通过 request.args.get('id')  来获取 id  的值。如果是 post  方法，
则可以通过 request.form.get('id')  来进行获取。

## 章节2 jinja

### 模板传参

可以将多个参数放到一个字典，传参数字典的时候，使用两个星号，将字典打散成关键字参数

### 过滤器

使用方式`{{ value|default('默认值') }}`。如果value这个`key`不存在，那么就会使用`default`过滤器提供的默认值。如果你想使用类似于`python`中判断一个值是否为False（例如：None、空字符串、空列表、空字典等），那么就必须要传递另外一个参数`{{ value|default('默认值',boolean=True) }}`。
可以使用`or`来替代`default('默认值',boolean=True)`。例如：`{{ signature or '此人很懒，没有留下任何说明' }}`。

11. truncate(value,length=255,killwords=False)：截取length长度的字符串。
12. striptags(value)：删除字符串中所有的HTML标签，如果出现多个空格，将替换成一个空格。
13. trim：截取字符串前面和后面的空白字符。
14. string(value)：将变量转换成字符串。
15. wordcount(s)：计算一个长字符串中单词的个数。

### 自定义过滤器

@app.template_filter('cut')
def cut(value):
    value = value.replace("hello",'')
    return value

```
​```html
<p>{{ article|cut }}</p>
```

### 宏

- 定义宏

```html
{%macros input(name,value='',type='text')%}
<input type="{{type}}",name="{{name}}",value="{{value}}">
{%endmacros%}
```

- 使用宏

```html
<p>{{input('username')}}</p>
<p>{{input('password',type='password')}}</p>
```

- 导入宏

1. `import "宏文件的路径" as xxx`。
2. `from '宏文件的路径' import 宏的名字 [as xxx]`。
3. 宏文件路径，不要以相对路径去寻找，都要以`templates`作为绝对路径去找。
4. 如果想要在导入宏的时候，就把当前模版的一些参数传给宏所在的模版，那么就应该在导入的时候使用`with context`。示例：`from 'xxx.html' import input with context`。

### include

1. 这个标签相当于是直接将指定的模版中的代码复制粘贴到当前位置。
2. `include`标签，如果想要使用父模版中的变量，直接用就可以了，不需要使用`with context`。
3. `include`的路径，也是跟`import`一样，直接从`templates`根目录下去找，不要以相对路径去找。

### set.with语句

在模板中，可以使用set来定义变量

```html
{% set username='知了课堂' %}
<p>用户名：{{ username }}</p>
```

一旦定义了这个变量，那么在后面的代码中，都可以使用这个变量

`with`语句定义的变量，只能在`with`语句块中使用，超过了这个代码块，就不能再使用了

```html
{% with classroom = 'zhiliao1班' %}
<p>班级：{{ classroom }}</p>
{% endwith %}
```

`with`语句也不一定要跟一个变量，可以定义一个空的`with`语句，以后在`with`块中通过`set`定义的变量，就只能在这个`with`块中使用了：

```html
{% with %}
    {% set classroom = 'zhiliao1班' %}
    <p>班级：{{ classroom }}</p>
{% endwith %}
```

### 加载静态文件

加载静态文件使用的是`url_for`函数。然后第一个参数需要为`static`，第二个参数需要为一个关键字参数`filename='路径'`。示例：

```html
{{ url_for("static",filename='xxx') }}
```

### 继承

继承语法：{% extends "base.html" %}

调用父模板代码中的代码

父：

```html
{% block body_block %}
        <p style="background: red;">这是父模板中的代码</p>
{% endblock %}
```

子：

```html
{% block body_block %}
    {{ super() }}
    <p style="background: green;">我是子模板中的代码</p>
{% endblock %}
```

如果想要在另外一个模版中使用其他模版中的代码。那么可以通过`{{ self.其他block名字() }}`就可以了。示例代码如下：

```html
{% block title %}
    知了课堂首页
{% endblock %}

{% block body_block %}
    {{ self.title() }}
    <p style="background: green;">我是子模板中的代码</p>
{% endblock %}
```

## 章节3 试图高级

### add_url_rule

add_url_rule(rule,endpoint=None,view_func=None)

这个方法用来添加url与视图函数的映射。如果没有填写`endpoint`，那么默认会使用`view_func`的名字作为`endpoint`。以后在使用`url_for`的时候，就要看在映射的时候有没有传递`endpoint`参数，如果传递了，那么就应该使用`endpoint`指定的字符串，如果没有传递，那么就应该使用`view_func`的名字。

## 章节4 数据库

### 连接数据库

使用SQLALchemy去连接数据库，需要使用一些配置信息，然后将他们组合成满足条件的字符串：
```python
HOSTNAME = '127.0.0.1'
PORT = '3306'
DATABASE = 'first_sqlalchemy'
USERNAME = 'root'
PASSWORD = '123456'
```

dialect+driver://username:password@host:port/database

```
DB_URI = 'mysql+pymysql://{0}:{1}@{2}:{3}/{4}?charset=utf8'.format(USERNAME,PASSWORD,HOSTNAME,PORT,DATABASE)
```

然后使用`create_engine`创建一个引擎`engine`，然后再调用这个引擎的`connect`方法，就可以得到这个对象，然后就可以通过这个对象对数据库进行操作了：
```python
engine = create_engine(DB_URI)

# 判断是否连接成功
conn = engine.connect()
result = conn.execute('select 1')
print(result.fetchone())
```

###　ＯＲＭ

将ｏｒｍ映射到数据库

```python
    from sqlalchemy.ext.declarative import declarative_base
    engine = create_engine(DB_URI)
    Base = declarative_base(engine)
```

```python
    class Person(Base):
        __tablename__ = 'person'
```

使用`Base.metadata.create_all()`来将模型映射到数据库中。

### 用session进行增删改查

orm 的操作需要session的会话对象进行操作

 ```python
    from sqlalchemy.orm import sessionmaker

    engine = create_engine(DB_URI)
    session = sessionmaker(engine)()
 ```

#### 添加对象

```python
p = Person(name='zhiliao',age=18,country='china')
session.add(p)
session.commit()
```

一次性添加多条数据

```python
p1 = Person(name='zhiliao1',age=19,country='china')
p2 = Person(name='zhiliao2',age=20,country='china')
session.add_all([p1,p2])
session.commit()
```

#### 查找对象

```python
# 查找某个模型对应的那个表中所有的数据：
all_person = session.query(Person).all()
# 使用filter_by进行条件查询
    all_person = session.query(Person).filter_by(name='zhiliao').all()
# 使用filter来做条件查询
    all_person = session.query(Person).filter(Person.name=='zhiliao').all()
# 使用get方法查找数据，get方法是根据id来查找的，只会返回一条数据或者None
person = session.query(Person).get(primary_key)
# 使用first方法获取结果集中的第一条数据
person = session.query(Person).first()
```

#### 修改对象

```python
person = session.query(Person).first()
person.name = 'ketang'
session.commit()
```

#### 删除对象

```python
person = session.query(Person).first()
session.delete(person)
session.commit()
```

### 常用数据类型

DECIMAL：定点类型。是专门为了解决浮点类型精度丢失的问题的。在存储钱相关的字段的时候建议大家都使用这个数据类型。并且这个类型使用的时候需要传递两个参数，第一个参数是用来标记这个字段总能能存储多少个数字，第二个参数表示小数点后有多少位。

Enum：枚举类型。指定某个字段只能是枚举中指定的几个值，不能为其他值。在ORM模型中，使用Enum来作为枚举，示例代码如下：

```python
    class Article(Base):
        __tablename__ = 'article'
        id = Column(Integer,primary_key=True,autoincrement=True)
        tag = Column(Enum("python",'flask','django'))
```

  在Python3中，已经内置了enum这个枚举的模块，我们也可以使用这个模块去定义相关的字段。示例代码如下：
    ```python
    class TagEnum(enum.Enum):
        python = "python"
        flask = "flask"
        django = "django"
    ```

    class Article(Base):
        __tablename__ = 'article'
        id = Column(Integer,primary_key=True,autoincrement=True)
        tag = Column(Enum(TagEnum))
    
    article = Article(tag=TagEnum.flask)
### Column常用参数

1. primary_key：设置某个字段为主键。
2. autoincrement：设置这个字段为自动增长的。
3. default：设置某个字段的默认值。在发表时间这些字段上面经常用。
4. nullable：指定某个字段是否为空。默认值是True，就是可以为空。
5. unique：指定某个字段的值是否唯一。默认是False。
6. onupdate：在数据更新的时候会调用这个参数指定的值或者函数。在第一次插入这条数据的时候，不会用onupdate的值，只会使用default的值。常用的就是`update_time`（每次更新数据的时候都要更新的值）。
7. name：指定ORM模型中某个属性映射到表中的字段名。如果不指定，那么会使用这个属性的名字来作为字段名。如果指定了，就会使用指定的这个值作为参数。这个参数也可以当作位置参数，在第1个参数来指定。
    ```python
    title = Column(String(50),name='title',nullable=False)
    title = Column('my_title',String(50),nullable=False)
    ```

### fliter过滤条件

- equals

  ```python
  article = session.query(Article).filter(Article.title == "title0").first()
  print(article)
  ```

- like

   ```python
  session.query(Article).filter(User.name.like('%ed%'))
   ```

- in

  ```python
  query.filter(User.name.in_(['ed','wendy','jack']))
      # 同时，in也可以作用于一个Query
  query.filter(User.name.in_(session.query(User.name).filter(User.name.like('%ed%'))))
  ```

- not in

  ```python
  query.filter(~User.name.in_(['ed','wendy','jack']))
  query.filter(User.name.notin_(['ed','wendy','jack']))
  ```

- is null

   ```python
  query.filter(User.name==None)
      # 或者是
  query.filter(User.name.is_(None))
   ```

- and

   ```python
      from sqlalchemy import and_
      query.filter(and_(User.name=='ed',User.fullname=='Ed Jones'))
      # 或者是传递多个参数
      query.filter(User.name=='ed',User.fullname=='Ed Jones')
      # 或者是通过多次filter操作
    query.filter(User.name=='ed').filter(User.fullname=='Ed Jones')
   ```

- or

  ```python
      from sqlalchemy import or_  query.filter(or_(User.name=='ed',User.name=='wendy'))
  ```

如果想要查看orm底层转换的sql语句，可以在filter方法后面不要再执行任何方法直接打印就可以看到了。比如：

```python
articles = session.query(Article).filter(or_(Article.title=='abc',Article.content=='abc'))
        print(articles)
```

### 外键约束

1. RESTRICT：父表数据被删除，会阻止删除。默认就是这一项。 
2. NO ACTION：在MySQL中，同RESTRICT。 
3. CASCADE：级联删除。 
4. SET NULL：父表数据被删除，子表数据会设置为NULL。

外键关系通过 ondelete='' ''来导入

### 一对多

```python
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True,autoincrement=True)
    username = Column(String(50),nullable=False)

    # articles = relationship("Article")

    def __repr__(self):
        return "<User(username:%s)>" % self.username

class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer,primary_key=True,autoincrement=True)
    title = Column(String(50),nullable=False)
    content = Column(Text,nullable=False)
    uid = Column(Integer,ForeignKey("user.id"))

    author = relationship("User",backref="articles")
```

### 一对一

```python
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True,autoincrement=True)
    username = Column(String(50),nullable=False)

    extend = relationship("UserExtend",uselist=False)

    def __repr__(self):
        return "<User(username:%s)>" % self.username

class UserExtend(Base):
    __tablename__ = 'user_extend'
    id = Column(Integer, primary_key=True, autoincrement=True)
    school = Column(String(50))
    uid = Column(Integer,ForeignKey("user.id"))

    user = relationship("User",backref="extend")
```

当然，也可以借助`sqlalchemy.orm.backref`来简化代码：
```python
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer,primary_key=True,autoincrement=True)
    username = Column(String(50),nullable=False)

    # extend = relationship("UserExtend",uselist=False)

    def __repr__(self):
        return "<User(username:%s)>" % self.username

class UserExtend(Base):
    __tablename__ = 'user_extend'
    id = Column(Integer, primary_key=True, autoincrement=True)
    school = Column(String(50))
    uid = Column(Integer,ForeignKey("user.id"))

    user = relationship("User",backref=backref("extend",uselist=False))
```

### orm层面的删除数据

1. save-update：默认选项。在添加一条数据的时候，会把其他和他相关联的数据都添加到数据库中。这种行为就是save-update属性影响的。 
2. delete：表示当删除某一个模型中的数据的时候，是否也删掉使用relationship和他关联的数据。
3. delete-orphan：表示当对一个ORM对象解除了父表中的关联对象的时候，自己便会被删除掉。当然如果父表中的数据被删除，自己也会被删除。这个选项只能用在一对多上，不能用在多对多以及多对一上。并且还需要在子模型中的relationship中，增加一个single_parent=True的参数。 
4. merge：默认选项。当在使用session.merge，合并一个对象的时候，会将使用了relationship相关联的对象也进行merge操作。 
5. expunge：移除操作的时候，会将相关联的对象也进行移除。这个操作只是从session中移除，并不会真正的从数据库中删除。 
6. all：是对save-update, merge, refresh-expire, expunge, delete几种的缩写。

### 三种排序方式

1. order_by：可以指定根据这个表中的某个字段进行排序，如果在前面加了一个-，代表的是降序排序。
2. 在模型定义的时候指定默认排序：有些时候，不想每次在查询的时候都指定排序的方式，可以在定义模型的时候就指定排序的方式。有以下两种方式：
    * relationship的order_by参数：在指定relationship的时候，传递order_by参数来指定排序的字段。
    * 在模型定义中，添加以下代码：

     __mapper_args__ = {
         "order_by": title
       }
    即可让文章使用标题来进行排序。
3. 正序排序与倒序排序：默认是使用正序排序。如果需要使用倒序排序，那么可以使用这个字段的`desc()`方法，或者是在排序的时候使用这个字段的字符串名字，然后在前面加一个负号。

### limit,offset以及切片操作

1. limit：可以限制每次查询的时候只查询几条数据。
2. offset：可以限制查找数据的时候过滤掉前面多少条。
3. 切片：可以对Query对象使用切片操作，来获取想要的数据。可以使用`slice(start,stop)`方法来做切片操作。也可以使用`[start:stop]`的方式来进行切片操作。一般在实际开发中，中括号的形式是用得比较多的。希望大家一定要掌握。示例代码如下：
```python
articles = session.query(Article).order_by(Article.id.desc())[0:10]
```

###　懒加载

在一对多，或者多对多的时候，如果想要获取多的这一部分的数据的时候，往往能通过一个属性就可以全部获取了。比如有一个作者，想要或者这个作者的所有文章，那么可以通过user.articles就可以获取所有的。但有时候我们不想获取所有的数据，比如只想获取这个作者今天发表的文章，那么这时候我们可以给relationship传递一个lazy='dynamic'，（定义在一的那一方）以后通过user.articles获取到的就不是一个列表，而是一个AppenderQuery对象了。这样就可以对这个对象再进行一层过滤和排序等操作。
通过`lazy='dynamic'`，获取出来的多的那一部分的数据，就是一个`AppenderQuery`对象了。这种对象既可以添加新数据，也可以跟`Query`一样，可以再进行一层过滤。
总而言之一句话：如果你在获取数据的时候，想要对多的那一边的数据再进行一层过滤，那么这时候就可以考虑使用`lazy='dynamic'`。
lazy可用的选项：
1. `select`：这个是默认选项。还是拿`user.articles`的例子来讲。如果你没有访问`user.articles`这个属性，那么sqlalchemy就不会从数据库中查找文章。一旦你访问了这个属性，那么sqlalchemy就会立马从数据库中查找所有的文章，并把查找出来的数据组装成一个列表返回。这也是懒加载。
2. `dynamic`：这个就是我们刚刚讲的。就是在访问`user.articles`的时候返回回来的不是一个列表，而是`AppenderQuery`对象。