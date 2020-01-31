---
layout: mypost
title: Django BBS 开发基础
categories: [Python, Django, Web, BBS, Tutorial]
---

# tianh bbs

作者：tianh，本文遵循 CC-BY-NC 协议。

联系我：thy@mail.ecust.edu.cn。

# 0 概述

这份文档以 MTV 体系结构创建一个 BBS 系统。

<img src="image-20200131032039867.png" alt="image-20200131032039867" style="zoom:50%;" />

在此过程中介绍了 Django 开发中的一些方面，包括

1. 配置文件

   一个 Django 项目的结构。

2. Django ORM

   ORM，在 Python 类与数据表之间建立映射，开发者不需要再写 SQL 代码，可以专注于业务逻辑。

3. Django 管理后台

   ORM 提供了丰富的 API，用于对 Model 进行增删改查，除此之外，Django 还提供了一个图形页面，几乎不用写代码就能完成这些操作。

4. Django 视图

   视图接受一个 HttpRequest，返回一个 HttpResponse。

5. Django 模板

   直接在 Python 中写 HTML 是很麻烦的，模板将两部分分离，用于动态地生成 HTML。

6. Django 表单

   Web 应用中与后端交互常采用表单提交，所有的表单处理流程都很相似，所以 Django 将其抽象为表单系统

7. Django 用户

   用户是 Web 应用中的关键概念，不同的用户有不同的权限。Django 用户系统提供了身份验证和权限管理的功能。

其中，2/4/5 部分是最重要的，任何一个 MVC 体系结构的 Web 应用都需要使用。另外，1 是 Django 项目的整体配置信息，3/6/7 是 Django 为 Web 应用提供的更方便的工具。

最近更新于：2020.1.31

# 1 项目准备

使用 PyCharm 创建项目 `tianh_bbs` ，运行正常。

运行 `pip install mysqlclient` 安装 mysqlclient，用于 Python 连接数据库。

安装 MySQL，执行 `create database django_bbs`，创建用于此项目的数据库。

## 1.1 项目骨架

### 1.1.1 manage.py

执行命令 `python manage.py runserver` 运行当前项目。

### 1.1.2 settings.py

配置语言和时区：

```python
LANGUAGE_CODE = 'zh-hans'

TIME_ZONE = 'Asia/Shanghai'

USE_I18N = True

USE_L10N = True

USE_TZ = False
```

配置数据库：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'django_bbs',
        'USER': 'root',
        'PASSWORD': 'admin',
        'HOST': '127.0.0.1',
        'PORT': '3306',
    }
}
```

运行 `python manage.py migrate` 进行数据库迁移，这之后在 MySQL 可以发现数据库不是空的了：

```shell script
mysql> use django_bbs;
Database changed
mysql> show tables;
+----------------------------+
| Tables_in_django_bbs       |
+----------------------------+
| auth_group                 |
| auth_group_permissions     |
| auth_permission            |
| auth_user                  |
| auth_user_groups           |
| auth_user_user_permissions |
| django_admin_log           |
| django_content_type        |
| django_migrations          |
| django_session             |
+----------------------------+
10 rows in set (0.01 sec)
```

#### 1.1.2.1 数据库迁移

Django 中进行数据库迁移有两个命令：

* `python manage.py makemigrations`
* `python manage.py migrate`

`makemigrations` 命令会检测应用目录下是否存在 `migrations` 目录，如果没有则创建。首先根据应用的表的结构生成 `0001_initial.py` 文件，里面定义了数据表的 `Schema`，再执行 `migrate` 就可以创建数据表了。

对于将来应用每一次表结构定义修改，都需要先执行 `makemigrations` ，再执行 `migrate`。

对于 Django 内置应用，数据库迁移文件已经生成好了，所以可以直接执行 `migrate` 命令。

为了防止重复迁移，Django 在 `django_migrations` 这个表中记录了每一次迁移，如：

```shell script
mysql> select * from django_migrations;
+----+--------------+------------------------------------------+----------------------------+
| id | app          | name                                     | applied                    |
+----+--------------+------------------------------------------+----------------------------+
|  1 | contenttypes | 0001_initial                             | 2020-01-30 23:19:18.588151 |
|  2 | auth         | 0001_initial                             | 2020-01-30 23:19:18.972120 |
|  3 | admin        | 0001_initial                             | 2020-01-30 23:19:19.927565 |
|  4 | admin        | 0002_logentry_remove_auto_add            | 2020-01-30 23:19:20.179920 |
|  5 | admin        | 0003_logentry_add_action_flag_choices    | 2020-01-30 23:19:20.190890 |
|  6 | contenttypes | 0002_remove_content_type_name            | 2020-01-30 23:19:20.404320 |
|  7 | auth         | 0002_alter_permission_name_max_length    | 2020-01-30 23:19:20.528029 |
|  8 | auth         | 0003_alter_user_email_max_length         | 2020-01-30 23:19:20.668612 |
|  9 | auth         | 0004_alter_user_username_opts            | 2020-01-30 23:19:20.679584 |
| 10 | auth         | 0005_alter_user_last_login_null          | 2020-01-30 23:19:20.782308 |
| 11 | auth         | 0006_require_contenttypes_0002           | 2020-01-30 23:19:20.788292 |
| 12 | auth         | 0007_alter_validators_add_error_messages | 2020-01-30 23:19:20.800261 |
| 13 | auth         | 0008_alter_user_username_max_length      | 2020-01-30 23:19:20.932905 |
| 14 | auth         | 0009_alter_user_last_name_max_length     | 2020-01-30 23:19:21.035678 |
| 15 | auth         | 0010_alter_group_name_max_length         | 2020-01-30 23:19:21.303914 |
| 16 | auth         | 0011_update_proxy_permissions            | 2020-01-30 23:19:21.319907 |
| 17 | sessions     | 0001_initial                             | 2020-01-30 23:19:21.374724 |
+----+--------------+------------------------------------------+----------------------------+
17 rows in set (0.00 sec)
```

## 1.2 创建用户和应用

### 1.2.1 创建超级用户

运行：

```shell script
(venv) ...\tianh_bbs>python manage.py createsuperuser
用户名 (leave blank to use 'tianh'): admin
电子邮件地址: admin@email.com
Password: admin
Password (again): admin
密码跟 用户名 太相似了。
密码长度太短。密码必须包含至少 8 个字符。
这个密码太常见了。
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.

```

然后运行项目，在 `http://127.0.0.1:8000/admin/` 就可以用超级用户登陆了。

### 1.2.2 创建应用

运行命令 `python manage.py startapp post`，创建应用。

- 外层的 `__init__.py` 文件标识 post 是一个 Python 包
- `admin.py` 用于将 Model 定义注册到管理后台，是 Django Admin 应用的配置文
  件。
- `apps.py` 用于应用程序本身的配置。
- `migrations` 目录用于存储 modesl.py 文件中 Model 的定义及修改。
- `migrations/__init__.py` 文件标识 migrations 是一个 Python 包。
-  `models.py` 用于定义应用中所需要的数据表。
-  `tests.py` 用于编写当前应用程序的单元测试。
-  `views.py` 用于编写应用程序的视图。

### 1.2.3 requirements.txt

运行命令 `pip freeze > requirements.txt`，生成 requirements.txt 文件。

重建环境时可以使用 `pip install -r requirements.txt`。

到这里我们就完成了项目的基本配置。

---

# 2 Django ORM

## 2.1 构建数据表

### 2.1.1 post 应用的 Models 定义

post 应用承载功能：

* 用户可以在 BBS 中发表 Topic
* 用户可以针对 Topic 发表 Comment
* 可以对每一个 Comment 支持或反对

`post/models.py`

```python
from django.db import models


class BaseModel(models.Model):
    """
    post 应用中 Model 的基类
    """

    class Meta:
        abstract = True
        ordering = ['-created_time']

    created_time = models.DateTimeField(auto_now_add=True, help_text=u'创建时间')
    last_modified = models.DateTimeField(auto_now=True, help_text=u'修改时间')

    def __str__(self):
        raise NotImplementedError
```

再把 Topic 和 Comment 实现，见源代码。

### 2.1.2 数据库迁移

`tianh_bbs/settings.py`

```python
INSTALLED_APPS = [
    'post.apps.PostConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

运行 `python manage.py makemigrations post` 生成迁移文件。

运行 `python manage.py sqlmigrate post 0001` 查看迁移文件。

运行 `python manage.py check` 进行自动检查。

运行 `python manage.py migrate` 完成迁移。

迁移后可以在 MySQL 中发现已经创建了新的表：

```shell script
mysql> desc post_topic;
+---------------+--------------+------+-----+---------+----------------+
| Field         | Type         | Null | Key | Default | Extra          |
+---------------+--------------+------+-----+---------+----------------+
| id            | int(11)      | NO   | PRI | NULL    | auto_increment |
| created_time  | datetime(6)  | NO   |     | NULL    |                |
| last_modified | datetime(6)  | NO   |     | NULL    |                |
| title         | varchar(255) | NO   | UNI | NULL    |                |
| content       | longtext     | NO   |     | NULL    |                |
| is_online     | tinyint(1)   | NO   |     | NULL    |                |
| user_id       | int(11)      | NO   | MUL | NULL    |                |
+---------------+--------------+------+-----+---------+----------------+
7 rows in set (0.02 sec)
```

## 2.2 Model 的相关概念

### 2.2.1 Model 的组成部分

每个 Model 都是 Python 类，由四部分组成：

* 继承自 `django.db.model.Model`
* `Model` 元数据声明
* 若干个 `Field` 类型的字段
* `__str__` 方法

更多内容参见 [Django Model]() 。

## 2.3 Model 查询 API

在应用中创建的每一个 Model 类，Django 都会自动添加一个名为 objects 的 django.db.models.Manager 对象。
通过这个对象我们可以实现增删改查。

### 2.3.1 创建实例

#### 2.3.1.1 通过实例化 Model 创建实例

```shell script
(venv) ...\bbs\tianh_bbs>python manage.py shell
Python 3.7.3 (v3.7.3:ef4ec6ed12, Mar 25 2019, 22:22:05) [MSC v.1916 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from post.models import Comment, Topic
>>> from django.contrib.auth.models import User
>>> user = User.objects.get(username='admin')
>>> topic = Topic(title='first topic',content='This is the first topic',user=user)
>>> topic.save()
>>> comment=Comment(
... content='Very good',topic=topic,up=88,down=32)
>>> comment.save()
```

#### 2.3.1.2 通过 objects 创建实例

```shell script
user = User.objects.get(username='admin')
topic_2 = Topic.objects.create(title='second topic', content='This is the second topic!', user=user)
Comment.objects.create(content='good!', topic=topic_2, up=30, down=17)
```

这两种方法都可以改变数据表

```shell script
mysql> select * from post_topic;
+----+----------------------------+----------------------------+--------------+---------------------------+-----------+---------+
| id | created_time               | last_modified              | title        | content                   | is_online | user_id |
+----+----------------------------+----------------------------+--------------+---------------------------+-----------+---------+
|  1 | 2020-01-31 02:51:58.063847 | 2020-01-31 02:51:58.063847 | first topic  | This is the first topic   |         1 |       1 |
|  2 | 2020-01-31 12:13:39.552741 | 2020-01-31 12:13:39.552741 | second topic | This is the second topic! |         1 |       1 |
+----+----------------------------+----------------------------+--------------+---------------------------+-----------+---------+
2 rows in set (0.00 sec)

mysql> select * from post_comment;
+----+----------------------------+----------------------------+-----------+----+------+----------+
| id | created_time               | last_modified              | content   | up | down | topic_id |
+----+----------------------------+----------------------------+-----------+----+------+----------+
|  1 | 2020-01-31 02:53:22.035212 | 2020-01-31 02:53:22.035212 | Very good | 88 |   32 |        1 |
|  2 | 2020-01-31 12:14:44.754924 | 2020-01-31 12:14:44.754924 | good!     | 30 |   17 |        2 |
+----+----------------------------+----------------------------+-----------+----+------+----------+
2 rows in set (0.00 sec)
```

### 2.3.2 查询实例

#### 2.3.2.1 单实例查询

##### 2.3.2.1.1 get() 方法

运行命令

```shell script
>>> from post.models import Topic,Comment
>>> from django.contrib.auth.models import User
>>> Topic.objects.get(id=1)
<Topic: 1: first topic>
>>> Topic.objects.get(id=3)
Traceback (most recent call last):
  ...
post.models.Topic.DoesNotExist: Topic matching query does not exist.
>>> Topic.objects.get()
Traceback (most recent call last):
  ...
post.models.Topic.MultipleObjectsReturned: get() returned more than one Topic -- it returned 2!
```

所以在使用 get() 方法时应该注意确保查询结果存在且唯一。

```python
from post.models import Topic
try:
    topic = Topic.objects.get(id=1, title='first topic')
except Topic.DoesNotExist:
    # do something
    print()
except Topic.MultipleObjectsReturned:
    # do something
    print()
```

如果使用自定义主键，可以用 `pk` 代替主键名称。

##### 2.3.2.1.2 get_or_create() 方法

这种方法返回元组 `(object, created)` 

```shell script
>>> Topic.objects.get_or_create(id=1, title='first topic')
(<Topic: 1: first topic>, False)
>>> Topic.objects.get_or_create(user=user, title='fourth topic', content='This is the fourth topic!')
(<Topic: 4: fourth topic>, True)
```

但是如果查询结果是多条数据，这个方法也会抛出 `MultipleObjectsReturned` 异常。

##### 2.3.2.1.3 first() last() 方法

运行命令

```shell script
>>> Topic.objects.first()
<Topic: 2: second topic>
>>> Topic.objects.last()
<Topic: 1: first topic>
```

#### 2.3.2.2 QuerySet 查询

这一部分首先对 QuerySet 查询进行介绍，
然后给出了执行 QuerySet 查询的几个方法，
最后介绍了 QuerySet 对象的切片操作。

除了这里介绍的方法，还有 distinct()，only()，defer() 等。

##### 2.3.2.2.1 惰性加载

QuerySet 只有在真正使用的时候才访问数据库检索记录，可以通过 `print(qs.query)` 查看 SQL 语句。

查看返回结果是否存在，使用 `qs.exists()` 。

查看返回结果的数量，使用 `qs.count()` 。

##### 2.3.2.2.2 all()

命令 `Topic.objects.all()` 会返回包含所有数据记录的 QuerySet。

##### 2.3.2.2.3 reverse()

命令 `Topic.objects.reverse()` 会返回逆序排列的所有数据 QuerySet。

##### 2.3.2.2.4 order_by()

命令 `Topic.objects.ordr_by('-title','created_time')` 会返回，先按标题逆序排列，再按创建时间顺序排列的所有数据 QuerySet。如果你不传递参数，不会按默认的排序规则返回，而是没有任何规则地返回。

##### 2.3.2.2.5 filter() 和 exclude()

filter() 函数将参数转化成 WHERE 子句。

| 条件查询关键字 | 含义 | WHERE 子句 |
| ------------- | ---- | ---------- |
| gt | 大于 | > |
| gte | 大于等于 | >= |
| lt | 小于 | < |
| lte | 小于等于 | <= |
| exact | 等于 | = |
| iexact | 大小写不敏感，等于 | LIKE xyz |
| in | 是否在集合中 | IN (abc,xyz) |
| contains | 是否包含 | LIKE BINARY %xyz% |
| startwith | 开头是 | LIKE BINARY xyz% |
| endwith | 结尾是 | LIKE BINARY %xyz |

例子：

```shell script
>>> from post.models import Topic,Comment
>>> from django.contrib.auth.models import User
>>> Topic.objects.filter()
<QuerySet [<Topic: 2: second topic>, <Topic: 1: first topic>]>
>>> Topic.objects.filter(id=1)
<QuerySet [<Topic: 1: first topic>]>
>>> Topic.objects.filter(id__lt=1)
<QuerySet []>
>>> Topic.objects.filter(id__gt=1)
<QuerySet [<Topic: 2: second topic>]>
```

exclude() 与 filter() 功能相反，返回不满去条件的结果。

##### 2.3.2.2.6 链式查询

```shell script
>>> Comment.objects.filter(
... content__contains='good'
... ).exclude(
... down__gt=20
... ).filter(
... up__gt=20
... )
<QuerySet [<Comment: 2: good!>]>
```

要记得，每次 order_by() 都会清除前面所有的排序方式。

##### 2.3.2.2.7 values() 和 values_list()

上面的方法返回的都是由对象组成的 QuerySet。

values() 可以返回由字典组成的 QuerySet，如

```shell script
>>> Comment.objects.values('id','up')
<QuerySet [{'id': 2, 'up': 30}, {'id': 1, 'up': 88}]>
```

values_list() 可以返回由元组组成的 QuerySet，如

```shell script
>>> Comment.objects.values_list('id','up')
<QuerySet [(2, 30), (1, 88)]>
```

使用 `flat` 参数还可以返回单个元素组成的 QuerySet，如

```shell script
>>> Comment.objects.values_list('id',flat=True)
<QuerySet [2, 1]>
```

##### 2.3.2.2.8 QuerySet 切片

QuerySet 对象支持切片操作，但是要注意

* 不支持从末尾切片
* 如果使用 step 切片，会执行数据库查询并返回实例列表，使用其他切片语法只会返回QuerySet
* 切片返回的 QuerySet 不能执行过滤或排序

#### 2.3.2.3 RawQuerySet 查询

对于复杂的查询，objects 提供了 raw() 方法，使用 SQL 语句执行查询，返回一个 RawQuerySet 对象。

这种方法还有一个特性，例如使用 `topic = Topic.objects.raw('SELECT id FROM post_topic` 之后，仍然可以获得 `topic.title`，Django会执行第二次查询。

RawQuerySet 支持索引和切片，但是不能在 RawQuerySet 上执行过滤等方法。

因为这种方法需要考虑不同数据库后端的特性，所以建议尽量使用常规 ORM 进行查询。

##### 2.3.2.3.1 参数化查询

可以在 SQL 语句中使用 `%s` 或者 `%(key)s` 占位，再传入一个列表或字典实现参数化查询。

#### 2.3.2.4 关联查询

##### 2.3.2.4.1 反向查询

因为 Comment 中定义了 ForeignKey 指向 Topic，所以每一个 Topic 对象都有一个管理器可以用来查询和它相关的 Comment 实例。
默认管理器名称为 `小写模型名_set`。

```shell script
>>> topic = Topic.objects.get(id=1)
>>> topic.comment_set.all()
<QuerySet [<Comment: 1: very good!>]>
```

需要注意对于 `OneToOne` 型关系，管理器代表的是一个单一的对象，且名称是小写的 Model 名。

##### 2.3.2.4.2 跨关系查询

```shell script
>>> Comment.objects.filter(topic__title__contains='first')
<QuerySet [<Comment: 1: Very good>]>
```

这种方式还可以更深，比如 `Comment.objects.filter(topic__user__username='admin')`，也支持反向查询，如 `Topic.objects.filter(comment__up__gte=30)`。

#### 2.3.2.5 F 对象

F 对象在没有获取数据值的情况下对某一字段进行引用。F 对象支持加减乘除、取模、幂等计算，运算符两边可以是具体的数值或 F 对象的引用。

F 对象支持跨关系查询

```shell script
>>> Comment.objects.filter(topic__content__contains=F('topic__t
itle'))
<QuerySet [<Comment: 2: good!>, <Comment: 1: Very good>]>
```

**例 1 赞小于等于反对**

```shell script
>>> from django.db.models import F
>>> Comment.objects.filter(up__lte=F('down'))
<QuerySet []>
```

**例 2 顶大于等于踩两倍**

```python
from post.models import Comment
from django.db.models import F
Comment.objects.filter(up__gt=F('down')*2)
```

**例 3 更新字段值**

```python
from django.db.models import F
from post.models import Comment
comment = Comment.objects.get(id=1)
comment = F('up') + 1
comment.save()
```

这种方式与 `comment.up += 1` 相比，将更新操作交给数据库完成，保证线程安全。

#### 2.3.2.6 Q 对象

Q 对象用于复杂查询，可以用 `&`，`|` 组合，可以用 `~` 取反。

```shell script
>>> from django.db.models import Q
>>> Comment.objects.filter(Q(up__gt=10)|Q(down__gt=10))
<QuerySet [<Comment: 2: good!>, <Comment: 1: Very good>]>
```

#### 2.3.2.7 聚合查询和分组查询

聚合和分组都是生成统计值的过程，聚合指对 QuerySet 整体生成一个统计值，分组指为 QuerySet 中的每一个对象生成一个统计值。

##### 2.3.2.7.1 聚合查询

```shell script
>>> from django.db.models import Avg, Count, Min, Max, Sum
>>> Comment.objects.filter(topic=1).aggregate(up_s=Sum('up'))
{'up_s': 88}
```

这里的 `topic=1` 等价于 `topic__pk=1` 。

##### 2.3.2.7.2 分组查询

默认情况下，`annotate` 会对每一个 Model 对象计算统计值。

```shell script
>>> for topic in Topic.objects.annotate(Count('comment')):
...     print('%d: %d' % (topic.id, topic.comment__count))
...
2: 1
1: 1
```

但是，如果使用了 values() 方法，就会按照 values() 中指定的字段对 Model 进行分组，再按每个组计算统计值。

比如，要获得每个 Topic 下所有 up 的和，

```shell script
>>> Comment.objects.values('topic_id').annotate(Sum('up')).order_by()
<QuerySet [{'topic_id': 2, 'up__sum': 30}, {'topic_id': 1, 'up_
_sum': 88}]>
```

`order_by()` 是为了清楚原本的排序方式，不然会生成错误的 SQL 语句，这是很重要的。

### 2.3.3 更新实例

#### 2.3.3.1 改变 Model 实例

```shell script
>>> comment = Comment.objects.get(id=1)
>>> comment.up = 90
>>> comment.save()
>>> # save 会更新 Comment 表的所有列
```

#### 2.3.3.2 Update() 方法

Update() 是 QuerySet 的方法，如

```shell script
>>> Comment.objects.filter(id=1).update(up=90, down=30)
1
```

因为 Update() 可以一次更新多个对象，所以返回值是受影响的数据记录的条数。

### 2.3.4 删除实例

不管是 Model 实例还是 QuerySet，都可以调用 delete() 方法删除实例。

这个方法返回一个二元组：第一个元素是删除的实例个数，第二个元素是一个字典，记录每一个 Model 类型删除的实例个数。


