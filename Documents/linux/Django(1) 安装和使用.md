# Django(1) 安装和使用

## Django 安装
`easy_install django`即可

## 使用

### 打开web服务器

1.在cmd中输入`c:\Python34\Scripts\django-admin.exe startproject mysite`
>project mysite会包含以下文件
```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
```
  

- manage.py:一个实用的命令行工具，可让你以各种方式与该 Django 项目进行交互。 你可以在 django-admin.py and manage.py 中查看关于 manage.py 所有的细节。
- mysite/\_\_init\_\_.py: 一个空文件，告诉 Python 该目录是一个 Python 包。
- mysite/settings.py: 该 Django 项目的设置/配置。
- mysite/urls.py: 该 Django 项目的 URL 声明; 一份由 Django 驱动的网站“目录”。
- mysite/wsgi.py: 一个 WSGI 兼容的 Web 服务器的入口，以便运行你的项目。

2.开启服务器
`python manage.py runserver` 可以在浏览器`http:\\127.0.0.1:8000`看到服务器已经打开
也可以通过`python manage.py runserver 8080`来指定端口

### 有意义的事情（投票系统)

1.数据库安装和设置([manual](1))
下载[sqlite下载页面](2)中`sqlite-shell-win32-XXX`和`sqlite-dll-win32-XXX`，并解压得到三个文件。利用命令`sqlite3 testDB.db`创建数据库。
在Django中配置sqlite3.
编辑mysite\settings.py中DATABASES中'default'以下值：

- ENGINE：从 'django.db.backends.postgresql_psycopg2', 'django.db.backends.mysql', 'django.db.backends.sqlite3', 'django.db.backends.oracle' 中选一个
- NAME：你的数据库名。如果你使用SQLite，该数据库将是你计算机上的一个文件；在这种情况下，:setting:NAME 将是一个完整的绝对路径，而且还包含该文件的名称。如果该文件不存在，它会在第一次同步数据库时自动创建（见下文）。
- USER：你的数据库用户名（sqlite不需要）
- PASSWORD: 你的数据库密码（sqlite不需要）
- HOST: 你的数据库主机地址（sqlite不需要）

默认情况下，:setting:INSTALLED_APPS 包含以下应用，这些都是由 Django 提供的：

- django.contrib.auth – 身份验证系统。
- django.contrib.contenttypes – 内容类型框架。
- django.contrib.sessions – session 框架。
- django.contrib.sites – 网站管理框架。
- django.contrib.messages – 消息框架。
- django.contrib.staticfiles – 静态文件管理框架。
所有这些应用中每个应用至少使用一个数据库表，所以在使用它们之前我们需要创建数据库中的表。要做到这一点，请运行以下命令：
`python manage.py migrate`

3.创建APP

在manage.py的同一个文件夹创建APP polls`python manage.py startapp polls`.polls 文件夹变成这样

```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```

使得app能够被访问
在polls/urls.py中添加

```python
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^$', views.index, name='index'),
]
```

在mysite/urls.py中添加

```python
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^polls/', include('polls.urls')),
    url(r'^admin/', admin.site.urls),
]
```

mysite中urls.py按照顺序进行正则匹配。匹配不包含域名和POST、GET等参数。具体查看re
模块。
一旦匹配成功，Django调用具体的view函数，并给予一个HttpRequest参数。
name值全局唯一，方便用来更改匹配方式。

4.建立模型

在投票系统中创建2个模型：Question和Choice。Question包括问题和发布日期。Choice包括选择和投票计数器。每个Choice都关联到一个Question.

实现这些功能需要在polls/models.py中添加下述代码

```python
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

其中每一个模型都是django.db.models.Model的子类。每个模型都有一些类变量。每个类变量是Field的子类，代表数据库中一个字段。CharField代码字符变量，DateTimeField代表日期。一些类变量需要参数，这些参数不是在数据库中用到，而是在验证中使用。最后用ForeignKey来告诉Django Choice关联到一个Question. Django支持多对多，多对1,1对1等多种关联关系。

5.激活模型

添加APP到安装列表(直接在列表最后加入polls好像也没有问题)
```python
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

然后运行`python manage.py makemigrations polls`告诉Django你对模型做了一些更改（这时是创建模型），改变被存储到migrate中。更改只是以文件的方式存储。根据更改需要对数据库进行相应的操作比如创建表等。就需要用到`python manage.py sqlmigrate polls 0001`.你会看到一些SQL语句。（这只是查看对应的SQL语句）。更改需要用到`python manage.py migrate`.(另外可以通过python manage.py check查看数据库是否有错。)。

migrations提供强大的功能：实时更改模型而不用担心数据库的数据不一致，也不需要重新删除在添加数据库。
修改模型的3步：
+   在models.py中修改模型
+   `python manage.py makemigrations` 根据修改建立migrations
+   `python manage.py migrate` 应用修改到数据库

#### 修改模块：添加显示功能

```python
from django.db import models
from django.utils.encoding import python_2_unicode_compatible

@python_2_unicode_compatible  # only if you need to support Python 2
class Question(models.Model):
    # ...
    def __str__(self):
        return self.question_text

@python_2_unicode_compatible  # only if you need to support Python 2
class Choice(models.Model):
    # ...
    def __str__(self):
        return self.choice_text
```

#### 添加自定义方法

```python
import datetime

from django.db import models
from django.utils import timezone


class Question(models.Model):
    # ...
    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```

6.创建admin

利用命令`python manage.py createsuperuser`创建admin用户。`python manage.py runserver`后，打开http://127.0.0.1:8000/admin/.进去后你可以看到可以修改的Groups和Users.这部分由django.contrib.auth认证框架提供。

7.注册polls(让polls app可以在网页中修改。)

在polls/admin.py中添加
```python
from django.contrib import admin

from .models import Question

admin.site.register(Question)
```

[1]: http://www.runoob.com/sqlite/sqlite-installation.html
[2]: http://www.sqlite.org/download.html