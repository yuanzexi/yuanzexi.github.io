# 【学习笔记】Flask

----

[TOC]

----
## 什么是Flask
Flask是一个基于python语言的轻量级的web应用框架，它的Python服务器网关接口（WSGI）采用的是Werkzeug，模板引擎则使用Jinja2。Flask也被称为微框架(microframework)，因为它使用简单的核心，用extension增加其他功能。Flask不会替你做任何决定，比如使用何种数据库，何种模板引擎，给予了开发者充分发挥的自由空间。也就是说，默认情况下，Flask不包括数据库抽象层，表单验证或者其他已有的库可以处理的东西。但Flask可以通过扩展为你的应用添加这些功能，就如同这些功能是 Flask 生的一样。 大量的扩展用以支持数据库整合、表单验证、上传处理和各种开放验证等等。Flask 可能是 “微小”的，但它已经为满足您的各种生产需要做出了充足的准备。

## 为什么用Flask
- Flask是一种微框架，核心简单又可扩展。因其轻量而又简单，Flask可以实现web快速开发的目的；同时又因其扩展性，Flask也可在后期实现多功能扩展的目的。
- Flask是基于Python语言的，开发者若拥有Python基础即可迅速上手。
- Python语言的胶水特性也为Web应用使用其他语言工具包（例如C++）提供了便利性。
## Flask使用简介
###  安装及环境配置
- Python环境 （Python 3.6）
    - 推荐安装Anaconda来管理Python环境。
    - 如果不使用Anaconda来管理Python环境，建议使用虚拟环境venv。
    ```windows
    创建一个项目文件夹，然后创建一个虚拟环境。
    创建完成后项目文件夹中会有一个 venv 文件夹：
    mkdir myproject
    cd myproject
    python3 -m venv venv
    在 Windows 下：
    py -3 -m venv venv
    在老版本的 Python 中要使用下面的命令创建虚拟环境：
    virtualenv venv
    在 Windows 下：
    \Python27\Scripts\virtualenv.exe venv
    激活虚拟环境
    在开始工作前，先要激活相应的虚拟环境：
    . venv/bin/activate
    在 Windows 下：
    venv\Scripts\activate
    激活后，你的终端提示符会显示虚拟环境的名称。
    退出虚拟环境
    结束工作后，可退出虚拟环境
    deactivate
    ```

- 依赖包
    - Flask：一般默认会在安装Python时自动安装。若无，可自行安装。
    ```
    pip install Flask
    ```
- IDE
    - Visual Studio对HTML编写比较友好。
    - Pycharm 对Python编程比较友好
### 最小的应用（Helloworld）
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return '<h1>Hello World!</h1>'

if __name__ == '__main__':
    app.run(debug=True)
```
1. app = Flask(__name__)   
创建Flask对象app，Flask类的构造函数只有一个必须指定的参数，即程序主模块或包的名字。在大多数程序中，Python的__name__变量就是所需要的值。  
2. @app.route('/')  
客户端（例如web浏览器）把请求发送给Web服务器,Web服务器再把请求发送给Flask程序实例。程序实例需要知道对每个URL请求运行哪些代码，所以保存了一个URL到Python函数的映射关系。处理URL和函数之间的关系的程序称为路由。  
在Flask程序中定义路由的最简便方式，是使用程序实例提供的app.route修饰器，把修饰的函数注册为路由。  
修饰器是Python语言的标准特性，可以使用不同的方式修改函数的行为。惯常用法是使用修饰器把函数注册为事件的处理程序。
3. def index(): 视图函数  
index()函数放在@app.route('/')后面，所以就是把index()函数注册为路由。如果部署程序的服务器域名为 `www.example.com`,在浏览器中访问`www.example.com`后，会触发服务器执行index()函数。
4. app.run(debug=True)  
程序实例用run方法启动Flask继承的开发Web服务器。
服务器启动后，会进入轮询，等待并处理请求。轮询会一直进行，直到程序停止，比如按Ctrl-C键。
debug=True表示启用调试模式。方便我们调试。
5. 启动运行  
将程序保存为 `app.py`, 命令行执行python app.py启动程序。然后打开浏览器，访问 http://localhost:5000 即可访问程序主页。
```
* Serving Flask app "app.py"
* Environment: production
WARNING: Do not use the development server in a production environment.
Use a production WSGI server instead.
* Debug mode: off
* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```

### 模板
在一般的 Web 程序里，访问一个地址通常会返回一个包含各类信息的 HTML 页面。因为我们的程序是动态的，页面中的某些信息需要根据不同的情况来进行调整，比如对登录和未登录用户显示不同的信息，所以页面需要在用户访问时根据程序逻辑动态生成。我们把包含变量和运算逻辑的 HTML 或其他格式的文本叫做模板，执行这些变量替换和逻辑计算工作的过程被称为渲染，渲染由模板渲染引擎——Jinja2 来完成。  
按照默认的设置，Flask 会从程序实例所在模块同级目录的 templates 文件夹中寻找模板，所以我们要在项目根目录创建templates文件夹。
Jinja2的语法与Python大致相同，在模板里，需要添加特定的定界符将Jinja2语句与变量标记出来，以下为三种常见的定界符：
- `{{...}}` 用来标记变量
- `{%...%}` 用来标记语句，比如if和for语句
- `{#...#}` 用来写注释   

模板中使用的变量需要在渲染的时候传递进去。
#### 模板及模板继承
`templates/base.html`：基模板
```html
<html>
<head>
{% block head %}
<title>{% block title %}{% endblock %} - My Application</title>
{% endblock %}
</head>
<body>
{% block body %}
{% endblock %}
</body>
</html>
```
`templates/index.html`：主页模板
```html
{% extends "base.html" %}
{% block title %}Index{% endblock %}
{% block head %}
{{ super() }}
<style>
</style>
{% endblock %}
{% block body %}
<h1>Hello, World!</h1>
{% endblock %}
```
extends 指令声明这个模板衍生自base.html。在extends 指令之后，基模板中的3 个块被重新定义，模板引擎会将其插入适当的位置。注意新定义的head 块，在基模板中其内容不是空的，所以使用super() 获取原来的内容。
#### Flask-Bootstrap扩展
为了使页面美观，可引用Bootstrap的CSS和Javascript组件。Flask提供了Bootstrap扩展[Flask-Bootstrap](https://flask-bootstrap-zh.readthedocs.io/zh/latest/)，简化集成的过程。
Flask-Bootstrap使用pip安装：
```
pip install flask-bootstrap
```
Flask 扩展一般都在创建程序实例时初始化。
```
from flask_bootstrap import Bootstrap
## ...
bootstrap = Bootstrap(app)
```
初始化Flask-Bootstrap 之后，就可以在程序中使用一个包含所有Bootstrap 文件的基模板。  
`templates/base.html`：使用Flask-Bootstrap 的模板
```html
{% extends "bootstrap/base.html" %}
{% block title %}Flasky{% endblock %}
{% block navbar %}
<div class="navbar navbar-inverse" role="navigation">
<div class="container">
<div class="navbar-header">
<button type="button" class="navbar-toggle"
data-toggle="collapse" data-target=".navbar-collapse">
<span class="sr-only">Toggle navigation</span>
<span class="icon-bar"></span>
<span class="icon-bar"></span>
<span class="icon-bar"></span>
</button>
<a class="navbar-brand" href="/">Flasky</a>
</div>
<div class="navbar-collapse collapse">
<ul class="nav navbar-nav">
<li><a href="/">Home</a></li>
</ul>
</div>
</div>
</div>
{% endblock %}
{% block content %}
<div class="container">
{% block page_content %}{% endblock %}
</div>
{% endblock %}
```
Jinja2 中的extends 指令从Flask-Bootstrap 中导入bootstrap/base.html， 从而实现模板继承。Flask-Bootstrap 中的基模板提供了一个网页框架，引入了Bootstrap 中的所有CSS 和JavaScript 文件。  
上面这个base.html 模板定义了3 个块，分别名为title、navbar 和content。这些块都是基模板提供的，可在衍生模板中重新定义。title 块的作用很明显，其中的内容会出现在渲染后的HTML 文档头部，放在< title> 标签中。navbar 和content 这两个块分别表示页面中的导航条和主体内容。
在这个模板中，navbar 块使用Bootstrap 组件定义了一个简单的导航条。
基于base.html，可简单编写index.html和user.html。

`templates/index.html`
```html
{% extends "base.html" %}
{% block title %}Flasky{% endblock %}
{% block page_content %}
<div class="page-header">
<h1>Hello, World!</h1>
</div>
{% endblock %}
```
`templates/user.html`
```html
{% extends "base.html" %}
{% block title %}Flasky{% endblock %}
{% block page_content %}
<div class="page-header">
<h1>Hello, {{ name }}!</h1>
</div>
{% endblock %}
```
#### 渲染主页模板

使用 render_template() 函数可以把模板渲染出来，必须传入的参数为模板文件名（相对于templates 根目录的文件路径），这里即 'index.html' 。为了让模板正确渲染，我们还要把模板内部使用的变量通过关键字参数传入这个函数，如下所示：  
`app.py`：返回渲染好的模板作为响应
```python
from flask import Flask, render_template
## ...
@app.route('/')
def index():
return render_template('index.html')

@app.route('/user/<name>')
def index(name):
return render_template('user.html', name=name)
```
在传入 render_template() 函数的关键字参数中，左边的 movies 是模板中使用的变量名称，右边的 movies 则是该变量指向的实际对象。这里传入模板的 name 是字符串， movies 是列表，但能够在模板里使用的不只这两种 Python 数据结构，你也可以传入元组、字典、函数等。  
render_template() 函数在调用时会识别并执行 index.html 里所有的 Jinja2 语句，返回渲染好的模板内容。在返回的页面中，变量会被替换为实际的值（包括定界符），语句（及定界符）则会在执行后被移除（注释也会一并移除）。  
#### Jinja2变量过滤器
可以使用过滤器修改变量，过滤器名添加在变量名之后，中间使用竖线分隔。例如，下述模板以首字母大写形式显示变量name 的值：
```html
Hello, {{ name|capitalize }}
```
|过滤器名|说　　明|
|-------|-------|
|safe| 渲染值时不转义|
|capitalize| 把值的首字母转换成大写，其他字母转换成小写|
|lower| 把值转换成小写形式|
|upper| 把值转换成大写形式|
|title| 把值中每个单词的首字母都转换成大写|
|trim| 把值的首尾空格去掉|
|striptags| 渲染之前把值中所有的HTML 标签都删掉|
### 表单
[Flask-WTF](http://pythonhosted.org/Flask-WTF/)扩展可以把处理Web 表单的过程变成一种愉悦的体验。这个扩展对独立的[WTForms](http://wtforms.simplecodes.com)包进行了包装，方便集成到Flask 程序中。  
Flask-WTF 及其依赖可使用pip 安装：
```
pip install flask-wtf
```
#### 设置密钥
默认情况下，Flask-WTF 能保护所有表单免受跨站请求伪造（Cross-Site Request Forgery，CSRF）的攻击。恶意网站把请求发送到被攻击者已登录的其他网站时就会引发CSRF 攻击。为了实现CSRF 保护，Flask-WTF 需要程序设置一个密钥。Flask-WTF 使用这个密钥生成加密令牌，再用令牌验证请求中表单数据的真伪。
```python
app = Flask(__name__)
app.config['SECRET_KEY'] = 'hard to guess string'
```
app.config 字典可用来存储框架、扩展和程序本身的配置变量。使用标准的字典句法就能把配置值添加到app.config 对象中。这个对象还提供了一些方法，可以从文件或环境中导入配置值。  
SECRET_KEY 配置变量是通用密钥，可在Flask 和多个第三方扩展中使用。

#### 表单定义
使用Flask-WTF 时，每个Web 表单都由一个继承自FlaskForm 的类表示。这个类定义表单中的一组字段，每个字段都用对象表示。字段对象可附属一个或多个验证函数。验证函数用来验证用户提交的输入值是否符合要求。
```python
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField
from wtforms.validators import DataRequired
class NameForm(FlaskForm):
name = StringField('What is your name?', validators=[DataRequired()])
submit = SubmitField('Submit')
```
这个表单中的字段都定义为类变量，类变量的值是相应字段类型的对象。在这个示例中，NameForm 表单中有一个名为name 的文本字段和一个名为submit 的提交按钮。StringField类表示属性为type="text" 的< input> 元素。SubmitField 类表示属性为type="submit" 的< input> 元素。字段构造函数的第一个参数是把表单渲染成HTML 时使用的标号。  
StringField 构造函数中的可选参数validators 指定一个由验证函数组成的列表，在接受用户提交的数据之前验证数据。验证函数DataRequired() 确保提交的字段不为空。
#### 表单渲染成HTML
表单字段是可调用的，在模板中调用后会渲染成HTML。假设视图函数把一个NameForm 实例通过参数form 传入模板，在模板中可以生成一个简单的表单，如下所示：
```html
<form method="POST">
{{ form.hidden_tag() }}
{{ form.name.label }} {{ form.name(id='my-text-field') }}
{{ form.submit() }}
</form>
```
Flask-Bootstrap 提供了一个非常高端的辅助函数，可以使用Bootstrap 中预先定义好的表单样式渲染整个Flask-WTF 表单，而这些操作只需一次调用即可完成。使用Flask-Bootstrap，上述表单可使用下面的方式渲染：
```html
{% import "bootstrap/wtf.html" as wtf %}
{{ wtf.quick_form(form) }}
```
import 指令的使用方法和普通Python 代码一样，允许导入模板中的元素并用在多个模板中。导入的bootstrap/wtf.html 文件中定义了一个使用Bootstrap 渲染Falsk-WTF 表单对象的辅助函数。wtf.quick_form() 函数的参数为Flask-WTF 表单对象，使用Bootstrap 的默认样式渲染传入的表单。
#### 表单处理
```python
@app.route('/', methods=['GET', 'POST'])
def index():
    name = None
    form = NameForm()
    if form.validate_on_submit():
        name = form.name.data
        form.name.data = ''
    return render_template('index.html', form=form, name=name)
```
app.route 修饰器中添加的methods 参数告诉Flask 在URL 映射中把这个视图函数注册为GET 和POST 请求的处理程序。
局部变量name 用来存放表单中输入的有效名字，如果没有输入，其值为None。如上述代码所示，在视图函数中创建一个NameForm 类实例用于表示表单。提交表单后，如果数据能被所有验证函数接受，那么validate_on_submit() 方法的返回值为True，否则返回False。这个函数的返回值决定是重新渲染表单还是处理表单提交的数据。  
用户第一次访问程序时，服务器会收到一个没有表单数据的GET 请求，所以validate_on_submit() 将返回False。if 语句的内容将被跳过，通过渲染模板处理请求，并传入表单对象和值为None 的name 变量作为参数。用户会看到浏览器中显示了一个表单。用户提交表单后，服务器收到一个包含数据的POST 请求。validate_on_submit() 会调用name 字段上附属的Required() 验证函数。如果名字不为空，就能通过验证，validate_on_submit() 返回True。现在，用户输入的名字可通过字段的data 属性获取。在if 语句中，把名字赋值给局部变量name，然后再把data 属性设为空字符串，从而清空表单字段。最后一行调用render_template() 函数渲染模板，但这一次参数name 的值为表单中输入的名字，因此会显示一个针对该用户的欢迎消息。
#### 重定向和用户会话
用户输入名字后提交表单，然后点击浏览器的刷新按钮，会看到一个莫名其妙的警告，要求在再次提交表单之前进行确认。之所以出现这种情况，是因为刷新页面时浏览器会重新发送之前已经发送过的最后一个请求。如果这个请求是一个包含表单数据的POST 请求，刷新页面后会再次提交表单。大多数情况下，这并不是理想的处理方式。使用重定向作为POST 请求的响应，而不是使用常规响应。重定向是一种特殊的响应，响应内容是URL，而不是包含HTML 代码的字符串。浏览器收到这种响应时，会向重定向的URL 发起GET 请求，显示页面的内容。这个技巧称为Post/ 重定向/Get 模式。
但这种方法会带来另一个问题。程序处理POST 请求时，使用form.name.data 获取用户输入的名字，可是一旦这个请求结束，数据也就丢失了。因为这个POST 请求使用重定向处理，所以程序需要保存输入的名字，这样重定向后的请求才能获得并使用这个名字，从而构建真正的响应。程序可以把数据存储在用户会话中，在请求之间“记住”数据。用户会话是一种私有存储，存在于每个连接到服务器的客户端中。  
重定向和用户会话
```python
from flask import Flask, render_template, session, redirect, url_for
@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        session['name'] = form.name.data
        return redirect(url_for('index'))
    return render_template('index.html', form=form, name=session.get('name'))
```
现在，包含合法表单数据的请求最后会调用redirect() 函数。redirect() 是个辅助函数，用来生成HTTP 重定向响应。redirect() 函数的参数是重定向的URL，这里使用的重定向URL 是程序的根地址，因此重定向响应本可以写得更简单一些，写成redirect('/')，但却会使用Flask 提供的URL 生成函数url_for()。推荐使用url_for() 生成URL，因为这个函数使用URL 映射生成URL，从而保证URL 和定义的路由兼容，而且修改路由名字后依然可用。  
url_for() 函数的第一个且唯一必须指定的参数是端点名，即路由的内部名字。默认情况下，路由的端点是相应视图函数的名字。在这个示例中，处理根地址的视图函数是index()，因此传给url_for() 函数的名字是index。  
session.get('name') 直接从会话中读取name 参数的值。和普通的字典一样，这里使用get() 获取字典中键对应的值以避免未找到键的异常情况，因为对于不存在的键，get() 会返回默认值None。

#### Flash 消息
请求完成后，有时需要让用户知道状态发生了变化。这里可以使用确认消息、警告或者错误提醒。一个典型例子是，用户提交了有一项错误的登录表单后，服务器发回的响应重新渲染了登录表单，并在表单上面显示一个消息，提示用户用户名或密码错误。flash() 函数可实现这种效果。
```python
from flask import Flask, render_template, session, redirect, url_for, flash
@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        old_name = session.get('name')
        if old_name is not None and old_name != form.name.data:
            flash('Looks like you have changed your name!')
        session['name'] = form.name.data
        return redirect(url_for('index'))
    return render_template('index.html',
        form = form, name = session.get('name'))
```
仅调用flash() 函数并不能把消息显示出来，程序使用的模板要渲染这些消息。最好在基模板中渲染Flash 消息，因为这样所有页面都能使用这些消息。Flask 把get_flashed_messages() 函数开放给模板，用来获取并渲染消息:
```html
{% block content %}
<div class="container">
{% for message in get_flashed_messages() %}
<div class="alert alert-warning">
<button type="button" class="close" data-dismiss="alert">&times;</button>
{{ message }}
</div>
{% endfor %}
{% block page_content %}{% endblock %}
</div>
{% endblock %}
```


### 静态文件
静态文件（static files）和我们的模板概念相反，指的是内容不需要动态生成的文件。比如图片、CSS 文件和 JavaScript 脚本等。  
在 Flask 中，我们需要创建一个 static 文件夹来保存静态文件，它应该和程序模块、templates 文件夹在同一目录层级。

#### 生成静态文件URL
在 HTML 文件里，引入这些静态文件需要给出资源所在的 URL。为了更加灵活，这些文件的 URL可以通过 Flask 提供的 url_for() 函数来生成。传入端点值（视图函数的名称）和参数，它会返回对应的 URL。对于静态文件，需要传入的端点值是 static ，同时使用 filename 参数来传入相对于 static 文件夹的文件路径。  
在 Python 脚本里， url_for() 函数需要从 flask 包中导入，而在模板中则可以直接使用，因为 Flask 把一些常用的函数和对象添加到了模板上下文（环境）里。
```html

<head>
<!-- 添加Favicon，显示在标签页和书签栏的网站头像 -->
<link rel="icon" href="{{ url_for('static', filename='favicon.ico') }}">
<!-- 添加CSS，页面样式 -->
<link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}" type="t
ext/css">
</head>
<!-- 添加图片，url = "/static/foo.jpg" -->
<img src="{{ url_for('static', filename='foo.jpg') }}">
```

### 错误页面
`templates/error.html` 错误页面模板 
```html
{% extends "base.html" %}
{% block title %}Flasky - Page Not Found{% endblock %}
{% block page_content %}
<div class="page-header">
<h1>Not Found</h1>
</div>
{% endblock %}
```
`app.py` 错误页面逻辑处理 
```python
@app.errorhandler(404)
def page_not_found(e):
    return render_template('error.html', message='404 找不到页面'), 404
```
## 常见问题
1. WTF-Form的API？  
详见 https://flask-wtf.readthedocs.io/en/latest/api.html#module-flask_wtf
2. 如何服务器上启动？  
用对应的主机地址启动
```python
app.run(host="192.168.1.123", debug=True)
```

## 相关文献
[Flask入门教程](http://helloflask.com/tutorial/)  
[Flask Web开发实战](http://helloflask.com/book/)
