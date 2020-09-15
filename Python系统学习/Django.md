## django入门
地址：https://zhuanlan.zhihu.com/p/98788776

安装好django后，在某个目录下
### 创建项目
```python
>>>django-admin startproject 项目名(比如说django_news)
```
* 生成的项目结构为：
    ```
    django_news
    ├── django_news              // 项目全局文件目录
    │   ├── __init__.py
    │   ├── settings.py          // 全局配置
    │   ├── urls.py              // 全局路由
    │   └── wsgi.py              // WSGI服务接口（暂时不用纠结这个是神马）
    └── manage.py                // 项目管理脚本
    ```
### 创建应用(app)
* Django App有三大类：
    * 内置：即 Django 框架自带的应用，包括 admin（后台管理）、auth（身份鉴权）、sessions（会话管理）等等
    * 自定义：即用来实现自身业务逻辑的应用，这里我们将创建一个新闻（news）展示应用
    * 第三方：即社区提供的应用，数量极其丰富，功能涵盖几乎所有方面，能够大大减少开发成本
* 所有的Django应用都在django_news/django_news/settings.py 的 INSTALLED_APPS 列表中定义:
    ```python
    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
        'news', # 新生成的应用需注册在这里
    ]
    ```
* 可以用```>>>python manage.py startapp news```来生成news应用，
* 生成之后的news应用文件夹结构：
    ```
    news                     // news 应用目录
    ├── __init__.py          // 初始化模块
    ├── admin.py             // 后台管理配置
    ├── apps.py              // 应用配置
    ├── migrations           // 数据库迁移文件目录
    │   └── __init__.py      // 数据库迁移初始化模块
    ├── models.py            // 数据模型
    ├── tests.py             // 单元测试
    └── views.py             // 视图
    ```
* Django使用MTV（对应MVC）,其中的V(views.py)对应MVC中的controller，T(templates)对应MVC中的view。
* 在 settings.py 中将 news 应用加入 INSTALLED_APPS 中。
### Django的路由
* 分为全局路由表(django_news/django_news/urls.py)和子应用路由表(django_news/news/urls.py)
### 编写第一个简单应用
#### 编写第一个视图
在django_news/news/views.py中添加：
```python
from django.http import HttpResponse
# 这是基于函数的视图FBV，还有基于类的视图CBV
def index(request):
    return HttpResponse('Hello World!')
```
### 将视图接入路由
1. 创建子应用路由表django_news/news/urls.py：
```python
from django.urls import path

from . import views
# 每一个Django路由表模块(urls.py)都约定有一个urlpatterns列表来存放路由映射表
# 列表中都使用django.urls.path 函数封装好的路由映射
# 3个参数：
# route：必须，即实际的访问路由，空字符串等于 /，即空路由
# view：必须，该路由将要访问的视图
# name：可选，该路由的名称，方便后续在模板中使用
urlpatterns = [
    path('', views.index, name='index'),
]
```
2. 在全局路由表（django_news/django_news/urls.py）中添加路由：
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('news.urls')),
]
```
3. 运行项目```>>>python manage.py runserver (端口号,默认8000)```这样访问locahost:8000就可以django_news/news/views.py中index函数返回的视图。
>注意：添加路由规则时顺序是很重要的，因为在尝试匹配时会按照从上到下的顺序进行，因此应该把最模糊的路由（即空路由）放在最下面。
###  Django的模板（就像jsp）
* 通过在一对花括号 {{}} 放入一个表达式，就能够在视图中传入表达式中变量的内容，并最终渲染成包含变量具体内容的 HTML 代码。
```html
<!-- 单个变量 -->
{{ variable }}

<!-- 获取字典的键或对象的属性 -->
{{ dict.key }}
{{ object.attribute }}

<!-- 获取列表中的某个元素 -->
{{ list.0 }}
```
* 条件语句
```html
{% if is_true %}
  <h1>It is true!</h1>
{% else %}
  <h1>It is false!</h1>
{% endif %}
```
* 循环语句
```html
{% for elem in some_list %}
  <p>{{ elem }}</p>
{% endfor %}
```
* 实现第一个Django模板
在news目录下面创建templates/news目录，在里面创建index.html
>注意：
听上去很麻烦，只创建 news/templates，然后把模板放里面不就好了，为什么还要再创建一个 news 目录？这是由于 Django 的模板查找机制会将所有应用里面的模板全部收集到一起，如果两个模板的名字冲突，就会导致其中一个模板不能被正确访问。如果放在 news 子文件夹里面，就能够通过 news/index.html 访问，通过命名空间的机制避免了冲突。
模板index.html的代码为：
```html
{% if news_list %}
  <ul>
  {% for elem in news_list %}
    <li>
      <h3>{{ elem.title }}</h3>
      <p>{{ elem.content }}</p>
    </li>
  {% endfor %}
  </ul>
{% else %}
  <p>暂无新闻</p>
{% endif %}
```
修改news/views.py：
```python
from django.shortcuts import render


def index(request):
    context = {
        'news_list': [
            {
                "title": "图雀写作工具推出了新的版本",
                "content": "随随便便就能写出一篇好教程，真的很神奇",
            },
            {
                "title": "图雀社区正式推出快速入门系列教程",
                "content": "一杯茶的功夫，让你快速上手，绝无担忧",
            },
        ]
    }
    # 调用django.shortcuts.render()来渲染模板，接受三个参数：
    # request：请求对象，直接把视图的参数 request 传进来就可以
    # template_name：模板名称，这里就是我们刚刚创建的 news/index.html
    # context：传入模板的上下文对象，必须是一个字典，字典中的每个键对应模板中的变量。
    return render(request, 'news/index.html', context=context)
```
### 与数据库(默认sqlite，也可以使用Mysql,Oracle等)联动，从数据库中取数据渲染模板
* Django ORM的例子：
```python
# 查询所有模型
# 等价于 SELECT * FROM Blog
Blog.objects.all()

# 查询单个模型
# 等价于 SELECT * FROM Blog WHERE ID=1
Blog.objects.get(id=1)

# 添加单个模型
# 等价于 INSERT INTO Blog (title, content) VALUES ('hello', 'world')
blog = Blog(title='hello', content='world')
blog.save()
```
1. 在news/models.py中定义数据模型Post:
```python
from django.db import models


class Post(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()

    def __str__(self):
        return self.title
```
2. 定义好之后，运行```>>>python manage.py makemigrations```来创建迁移文件：可以看到自动创建了 news/migrations/0001_initial.py 迁移脚本。
3. 接着运行```>>>python manage.py migrate```来进行数据库迁移
4. 可以运行```>>>python manage.py createsuperuser```来创建超级用户，运行项目```>>>python manage.py runserver (端口号,默认8000)```登录后没看到Post模型
5. 配置后台管理接口，在news/admin.py填入如下代码，就可以看到news应用和Post模型了，就可以在页面上添加数据
```python
from django.contrib import admin

from .models import Post

admin.site.register(Post)
```

6. 在视图中添加数据查询，在news.views.py填入代码：
```python
from django.shortcuts import render

from .models import Post


def index(request):
    # 查询数据库
    context = { 'news_list': Post.objects.all() }
    return render(request, 'news/index.html', context=context)
```
7. 登录，就可以看到看看在后台配置的Post数据