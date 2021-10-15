# 高级应用

## 一、Admin后台管理

### 1、概述

什么是admin？

admin是后台可视化界面管理系统，方便模型的操作与维护

### 2、配置

+ settings.py

  ```django
  INSTALLED_APPS = [
      'django.contrib.admin',
  ]
  ```

  汉化

  ```python
  LANGUAGE_CODE = 'zh-Hans'

  TIME_ZONE = 'Asia/Shanghai'
  ```

+ 创建用户进行访问

  python manage.py createsuperuser

  依次输入用户名、邮箱、密码

+ 路由访问

  127.0.0.1:8000/admin/

### 3、配置后台

班级与学生模型类

+ models.py

  ```python
  from django.db import models
  
  # 创建班级模型
  class Grade(models.Model):
      gname = models.CharField(max_length=10, default='python36')
      gnum = models.SmallIntegerField(default=30)
      gboynum = models.SmallIntegerField(default=15)
      ggirlnum = models.SmallIntegerField(default=15)
      def __str__(self):
          return self.gname
      class Meta:
          db_table = 'grade'
  
  
  # 创建学生模型
  class Student(models.Model):
      sname = models.CharField(max_length=10, default='张三')
      sage = models.SmallIntegerField(default=18)
      ssex = models.BooleanField(default=True)
      # 一对多外键关系 从表随着主表而删除
      # sgrade = models.ForeignKey(Grade, on_delete=models.CASCADE)
      # sgrade = models.ForeignKey(Grade, on_delete=models.SET_NULL, null=True)
      # sgrade = models.ForeignKey(Grade, on_delete=models.PROTECT)
      sgrade = models.ForeignKey(Grade, on_delete=models.SET_DEFAULT, default=1)
  
      def __str__(self):
          return self.sname
      class Meta:
          db_table = 'student'
  ```

+ 设置为中文显示(字段在后台进行中文显示)

  ```python
  class Grade(models.Model):
      ...
      ggirlnum = models.SmallIntegerField(default=15, verbose_name='女生人数')
      class Meta:
          db_table = 'grade'
          verbose_name = '班级'
          # verbose_name_plural = '班级'
  class Student(models.Model):
      ...
  ```

  > 在设置verbose_name后，django admin web界面上显示的内容可能会在末尾多个 S ，这个时候使用verbose_name_plural 设置即可

+ admin.py

  ```python
  from django.contrib import admin
  from App.models import Grade, Student
  
  # class StudentInfo(admin.StackedInline):  # 竖着
  class StudentInfo(admin.TabularInline):  # 横着
      model = Student
      extra = 2
  
  @admin.register(Grade) # 使用装饰器的方式去注册应用
  class GradeAdmin(admin.ModelAdmin):
      inlines = [StudentInfo]
      # 字段显示
      list_display = ['pk', 'gname', 'gnum', 'gboynum', 'ggirlnum']
      # 过滤字段
      list_filter = ['gname']
      # 搜索字段
      search_fields = ['gname']
      # 分页
      list_per_page = 5  # 每页显示五条数据
      # 更改添加修改样式
      # fields = ['gname', 'gnum', 'gboynum', 'ggirlnum']
      # 添加修改分组(fieldsets和fields不能同时存在)
      fieldsets = [
          ('组一', {'fields':['gboynum', 'ggirlnum']}),
          ('组二', {'fields':['gname', 'gnum']})
      ]
  
  
  
  @admin.register(Student)
  class StudentAdmin(admin.ModelAdmin):
      def show_sex(self):
          if self.ssex:
              return '男'
          else:
              return '女'
  
      show_sex.short_description = '性别'
      list_display = ['pk', 'sname', 'sage', show_sex, 'sgrade']
  
    # Register your models here.
    # admin.site.register(Grade, GradeAdmin)
    # admin.site.register(Student, StudentAdmin)
  ```
  
## 二、邮件发送

[更多邮件配置可以查看Django官方文档](https://docs.djangoproject.com/zh-hans/3.2/topics/email/)

### 1、配置

基础阶段里面的邮箱配置

### 2、发送邮件

  ```
```python
def test_send_mail(req):
    from django.core.mail import send_mail
    send_mail(
        '主体',
        '邮件内容',
        settings.EMAIL_HOST_USER,
        ['793390457@qq.com'],
        fail_silently=False,
    )
    return HttpResponse('发送邮件')
  ```

### 3、群发

```python
from django.core.mail import send_mass_mail
message1 = ('Subject here', 'Here is the message', 'from@example.com', ['first@example.com', 'other@example.com'])
message2 = ('Another Subject', 'Here is another message', 'from@example.com', ['second@test.com'])
send_mass_mail((message1, message2), fail_silently=False)
```

> send_mass_mail（）和 send_mail（）之间的主要区别在于，send_mail（）每次执行时都会打开到邮件服务器的连接，而send_mass_mail（）对其所有邮件使用单个连接。这使得send_mass_mail（）的效率略微提高。

### 4、使用文本和html发送

```python
from django.core.mail import EmailMultiAlternatives

subject, from_email, to = 'hello', 'from@example.com', 'to@example.com'
text_content = 'This is an important message.'
html_content = '<p>This is an <strong>important</strong> message.</p>'
msg = EmailMultiAlternatives(subject, text_content, from_email, [to])
msg.attach_alternative(html_content, "text/html")
msg.send()
```

### 5、使用html发送

```python
from django.core.mail import send_mass_mail, EmailMessage
msg = EmailMessage('主体', '<b>邮件内容</b>', settings.EMAIL_HOST_USER, ['793390457@qq.com','948807313@qq.com'])
    msg.content_subtype = "html"  # Main content is now text/html
    msg.send()
```

## 三、分页

### 1、Paginator对象

+ 创建对象

  Paginator()

+ 属性

  count 对象总数

  num_pages 页面总数

  page_range  页码列表

+ 方法

  page(num)

### 2、page对象

通过Paginator的page方法，得到page对象

+ 属性

  object_list  当前页上所有数据

  number  当前页的页码值

  paginator  获取paginator  对象

+ 方法

  has_next  是否有下一页

  has_previous 是否有上一页

  next_page_num  返回下一页页码

  previous_page_num  返回上一页页码

### 3、实例

+ views.py

  ```python
  def index(req):
      nowPage = req.GET.get('page',1)
      stu = Student.objects.all()
      p = Paginator(stu, 2)
      # print(p)

      # 通过页码获取当前页的对象
      try:
          # 判断页码是否合法
          if int(nowPage) >= int(p.num_pages):
              nowPage = p.num_pages
          page = p.page(nowPage)
      except:
          page = p.page(p.num_pages)

      return render(req, 'index.html', {'page': page})
  ```

+ index.html

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css" integrity="sha384-HSMxcRTRxnN+Bdg0JdbxYKrThecOKuH5zCYotlSAcp1+c8xmyTe9GYg1l9a69psu" crossorigin="anonymous">
  </head>
  <body>
  首页
  
  <ol>
      {% for s in page.object_list   %}
          <li>{{ s.sname }}</li>
      {% endfor %}
  </ol>
  
  <nav aria-label="Page navigation">
    <ul class="pagination pagination-lg">
      <li>
        <a {% if page.has_previous %}href="{% url 'App:index' %}?page={{ page.previous_page_number }}"{% endif %} aria-label="Previous">
          <span aria-hidden="true">&laquo;</span>
        </a>
      </li>
        {% for index in page.paginator.page_range %}
            <li {% if index == page.number %}class="active"{% endif %}><a href="{% url 'App:index' %}?page={{ index }}">{{ index }}</a></li>
        {% endfor %}
  
  {#    <li><a href="#">2</a></li>#}
      <li>
        <a {% if page.has_next %}href="{% url 'App:index' %}?page={{ page.next_page_number }}"{% endif %} aria-label="Next">
          <span aria-hidden="true">&raquo;</span>
        </a>
      </li>
    </ul>
  </nav>
  </body>
  </html>
  ```

## 四、中间件

### 1、说明

中间件就是在请求与响应过程中，进行安全验证的功能，每个中间件组件都会负责做一些特定的功能

### 2、本质

python类

### 3、使用场景

+ 白名单、黑名单
+ url访问过滤
+ 缓存

浏览器 --> 中间件 --> url路由地址分发 --> 中间件 --> 视图函数 --> 中间件 --> 响应

### 4、方法

+ process_request(self, req)

  在执行视图之前被调用(分配url匹配视图)，每个请求都会被调用，返回None或者response对象

+ process_view(self, req, view_func, view_args, view_kwargs)

  调用视图之前进行调用，每个表请求都会调用，返回None或者response对象

+ process_response(self, req, response)

  所有响应在返回浏览器之前都会被调用，返回response

+ process_exception(self, req, exception)

  当视图抛出异常时进行调用

### 5、自定义中间件

+ App同级创建middleware文件夹

  ```
  App/
  middleware/
  	myMiddle.py
  ```

+ myMiddle.py中创建MyMiddle类

  ```python
  from django.utils.deprecation import MiddlewareMixin
  from django.shortcuts import HttpResponse, redirect


  class MyMiddle(MiddlewareMixin):
      def process_request(self, req):
          print(req.path)
          print(req.method)
          print(req.META['REMOTE_ADDR'])
          if req.META['REMOTE_ADDR'] == '127.0.0.1':
              return HttpResponse('您被拦截了')

      def process_exception(self, req, exception):
          print(exception)
          return redirect('/a/')
  ```

+ settings.py

  middleware中添加

  ```python
  'middleware.myMiddle.MyMiddle',
  ```




## 五、用户认证

### 1、说明

auth_user 是用来维护用户信息的关系模式

### 2、auth_user

django项目中`python manage.py migrate`后创建的表

```mysql
CREATE TABLE `auth_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `password` varchar(128) NOT NULL,
  `last_login` datetime(6) DEFAULT NULL,
  `is_superuser` tinyint(1) NOT NULL,
  `username` varchar(150) NOT NULL,
  `first_name` varchar(150) NOT NULL,
  `last_name` varchar(150) NOT NULL,
  `email` varchar(254) NOT NULL,
  `is_staff` tinyint(1) NOT NULL,
  `is_active` tinyint(1) NOT NULL,
  `date_joined` datetime(6) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

### 3、用户模型属性方法

+ is_staff

  boolean 用户是否可以访问后台界面，默认False

+ is_active

  boolean   用户是否活跃 / 是否当前删除用户，默认True

+ is_authenticated()

  用户是否通过验证（是否登录）

  request.user.is_authenticated()

+ make_password()

  密码加密，django自带的hash加密

+ check_password()

  验证密码是否正确

+ set_password()

  修改密码

+ authenticated()

  验证用户名和密码是否正确

  authenticated(username=username, password=password)   结果bool

+ create_user()

  创建用户

  User.objects.create_user(username, email, password)

+ request.user

  获取当前用户登录对象

+ login_required

  设置当前视图只能登录后进行访问

+ login()

  处理登录

  login(req, user)

+ logout()

  退出登录

  logout(req)

### 4、注册

+ register.html

  ```html
  <!DOCTYPE html>
  <html>
  <head>
      <title>注册页面</title>
  <style>
          .pg_header{
              position: fixed;
              height: 48px;
              top: 0;
              left: 0;
              right: 0;
              background-color: #2459a2;
              line-height: 48px;
          }
          .pg_header .logo{
              margin: 0 auto;
              float: left;
              width: 200px;
              text-align: center;
              line-height: 48px;
              font-size: 28px;
              color: white;
          }
          .pg_dl{
              left: 400px;
              display: inline-block;
              padding: 0 40px;
              color: white;
          }
          .pg_header .pg_dl:hover{
              background-color: #2459fb;
              cursor: pointer;
          }
          .left{
              margin-top: 20px;
              width: 400px;
              display: inline-block;
              float: left;
          }
          .pg_body{
              margin-top: 50px;
              font-size: 18px;
              display: inline-block;
              width: 200px;
          }
          .pg_body .menu{
              width: 800px;
              padding: 15px;
              float: left;
              font-weight: bold;
          }
          input[type="text"]{
              width: 200px;
              height: 25px;
              border-radius: 6px;
          }
          input[type="password"]{
              width: 200px;
              height: 25px;
              border-radius: 6px;
          }
          input[type="button"]{
              background-color: #555555;
                 border: none;
              color: white;
              padding: 12px 29px;
              text-align: center;
              text-decoration: none;
              display: inline-block;
              font-size: 17px;
              margin: 4px 2px;
              cursor: pointer;
              border-radius: 4px;
          }
          input[type="submit"]{
              background-color: #555555;
                 border: none;
              color: white;
              padding: 12px 29px;
              text-align: center;
              text-decoration: none;
              display: inline-block;
              font-size: 17px;
              margin: 4px 2px;
              cursor: pointer;
              border-radius: 4px;
          }
          .kong{
              margin-top: -54px;
              margin-left: 200px;
              float:left;
              padding: 15px;
          }
          .img{
              width: 50px;
              height: 40px;
          }
          .can{
              width: 1220px;
              height: 40px;
              line-height: 40px;
              margin: 0 auto;
              text-align: center;
              display: inline-block;
          }
          .tian{
              color: red;
              float: right;
              font-size: 12px;
              margin-right: -120px;
              margin-top: -25px;
          }
      </style>
  </head>
  <body id="i88" style="margin: 0">
      <h1>{{ msg }}</h1>
      <div class = "pg_header">
          <a class = "logo">LOGO</a>
          <a class="pg_dl" id="i77">注册</a>
      </div>
      <form name="tijiao" method="post" onsubmit="return check()" action="">
          {% csrf_token %}
          <div class="left"></div>
          <div class="pg_body">
                  <div class="menu">用户名:</div>
                      <div class="kong">
                          <input id="text1" type="text" name="username" placeholder="请输入用户名" onblur="check()"><span id="div1" class="tian" style="margin-top: 4px">*(为必填)</span>
                      </div>

                  <div class="menu">密码:</div>
                      <div class="kong">
                          <input id="text2" type="password" name="userpass" onblur="check()">
                          <span id="div2" class="tian" style="margin-top: 5px">*(为必填)</span>
                      </div>
                  <div class="menu">确认密码:</div>
                      <div class="kong">
                          <input id="text3" type="password" name="01" onblur="check()">
                          <span id="div3" class="tian" style="margin-top: 5px">*(为必填)</span>
                      </div>
                  <div class="menu">邮箱地址:</div>
                      <div class="kong">
                          <input id="text4" type="text" name="email" onblur="check()" required>
                          <span id="div4" class="tian" style="margin-top: 5px">*(为必填)</span>
                      </div>
          </div>
          <div class="can">
              <input id="i111" type="submit" name="002" value="注  册">
              <p style="width: 200px;display: inline-block;"></p>
              <input id="i222" type="button" name="004" value="取  消">
          </div>
      </form>
      <script type="text/javascript">
              //刷新or取消
              document.getElementById('i77').onclick = function(){
                  location.reload();
              }
              document.getElementById('i222').onclick = function(){
                  location.reload();
              }

              //用户名验证
              function checkname(){
                  var div = document.getElementById("div1");
                  div.innerHTML = "";
                  var name1 = document.tijiao.text1.value;
                  if (name1 == "") {
                  div.innerHTML = "用户名不能为空！";
                  document.tijiao.text1.focus();
                  return false;
              }
                  if (name1.length < 4 || name1.length > 16) {
                  div.innerHTML = "长度4-16个字符";
                  document.tijiao.text1.select();
                  return false;
              }
                  var charname1 = name1.toLowerCase();
                  for (var i = 0; i < name1.length; i++) {
                  var charname = charname1.charAt(i);
                  if (!(charname >= 0 && charname <= 9) && (!(charname >= 'a' && charname <= 'z')) && (charname != '_')) {
                      div.innerHTML = "用户名包含非法字符";
                      document.form1.text1.select();
                      return false;
                  }
              }
                  return true;
          }

              //密码验证
              function checkpassword(){
                  var div = document.getElementById("div2");
                  div.innerHTML = "";
                  var password = document.tijiao.text2.value;
                  if (password == "") {
                  div.innerHTML = "密码不能为空";
                  {#document.tijao.text2.focus();#}
                  return false;
              }
                  if (password.length < 4 || password.length > 16) {
                  div.innerHTML = "密码长度为4-16位";
                  document.tijiao.text2.select();
                  return false;
                  }
                  return true;
          }

              function checkrepassword(){
                  var div = document.getElementById("div3");
                  div.innerHTML = "";
                  var password = document.tijiao.text2.value;
                  var repass = document.tijiao.text3.value;
                  if (repass == "") {
                  div.innerHTML = "密码不能为空";
                  document.tijiao.text3.focus();
                  return false;
                  }
                  if (password != repass) {
                  div.innerHTML = "密码不一致";
                  document.tijiao.text3.select();
                  return false;
                  }
                  return true;
              }
          //邮箱验证
          function checkEmail(){
              var div = document.getElementById("div4");
              div.innerHTML = "";
              var email = document.tijiao.text4.value;
              var sw = email.indexOf("@", 0);
              var sw1 = email.indexOf(".", 0);
              var tt = sw1 - sw;
              if (email.length == 0) {
              div.innerHTML = "邮箱不能为空";
              document.tijiao.text5.focus();
              return false;
              }

              if (email.indexOf("@", 0) == -1) {
              div.innerHTML = "必须包含@符号";
              document.tijiao.text5.select();
              return false;
              }

              if (email.indexOf(".", 0) == -1) {
              div.innerHTML = "必须包含.符号";
              document.tijiao.text5.select();
              return false;
              }

              if (tt == 1) {
                  div.innerHTML = "@和.不能一起";
                  document.tijiao.text5.select();
                  return false;
              }

              if (sw > sw1) {
                  div.innerHTML  = "@符号必须在.之前";
                  document.tijiao.text5.select();
                  return false;
              }
              else {
                  return true;
              }
              return ture;
              }

              function check(){
              if (checkname() && checkpassword() && checkrepassword() && checkEmail()) {
              return true;
              }
              else {
              return false;
          }
      }
      </script>
  </body>
  </html>
  ```

+ views.py

  ```python
  # 注册
  def register(req):
      message = ''
      if req.method == 'POST':
          username = req.POST.get('username')
          email = req.POST.get('email')
          userpass = req.POST.get('userpass')
          u = User.objects.filter(username=username).exists()
          if not u:
              User.objects.create_user(username=username, email=email, password=userpass)
              return redirect('/login/')
          else:
              message = '该用户以存在'
      return render(req, 'register.html', {'msg': message})
  ```

  自己完善python正则验证用户名 密码  邮箱

  完善正则验证

### 5、登录

+ views.py

  ```python
  from django.contrib.auth import authenticate, login

  # 登录
  def user_login(req):
      if req.method == 'POST':
          username = req.POST.get('username')
          userpass = req.POST.get('userpass')
          u = authenticate(username=username, password=userpass)
          if u:
              login(req, u)
              return redirect('/')
          else:
              return redirect('/login/')
      return render(req, 'login.html')
  ```

+ login.html

  ```html
  <!DOCTYPE html>
  <html>
  <head>
      <title>登录页面</title>
  <style>
          .pg_header{
              position: fixed;
              height: 48px;
              top: 0;
              left: 0;
              right: 0;
              background-color: #2459a2;
              line-height: 48px;
          }
          .pg_header .logo{
              margin: 0 auto;
              float: left;
              width: 200px;
              text-align: center;
              line-height: 48px;
              font-size: 28px;
              color: white;
          }
          .pg_dl{
              left: 400px;
              display: inline-block;
              padding: 0 40px;
              color: white;
          }
          .pg_header .pg_dl:hover{
              background-color: #2459fb;
              cursor: pointer;
          }
          .left{
              margin-top: 20px;
              width: 400px;
              display: inline-block;
              float: left;
          }
          .pg_body{
              margin-top: 50px;
              font-size: 18px;
              display: inline-block;
              width: 200px;
          }
          .pg_body .menu{
              width: 800px;
              padding: 15px;
              float: left;
              font-weight: bold;
          }
          input[type="text"]{
              width: 200px;
              height: 25px;
              border-radius: 6px;
          }
          input[type="password"]{
              width: 200px;
              height: 25px;
              border-radius: 6px;
          }
          input[type="button"]{
              background-color: #555555;
                 border: none;
              color: white;
              padding: 12px 29px;
              text-align: center;
              text-decoration: none;
              display: inline-block;
              font-size: 17px;
              margin: 4px 2px;
              cursor: pointer;
              border-radius: 4px;
          }
          input[type="submit"]{
              background-color: #555555;
                 border: none;
              color: white;
              padding: 12px 29px;
              text-align: center;
              text-decoration: none;
              display: inline-block;
              font-size: 17px;
              margin: 4px 2px;
              cursor: pointer;
              border-radius: 4px;
          }
          .kong{
              margin-top: -54px;
              margin-left: 200px;
              float:left;
              padding: 15px;
          }
          .img{
              width: 50px;
              height: 40px;
          }
          .can{
              width: 1220px;
              height: 40px;
              line-height: 40px;
              margin: 0 auto;
              text-align: center;
              display: inline-block;
          }
          .tian{
              color: red;
              float: right;
              font-size: 12px;
              margin-right: -120px;
              margin-top: -25px;
          }
      </style>
  </head>
  <body id="i88" style="margin: 0">
      <div class = "pg_header">
          <a class = "logo">LOGO</a>
          <a class="pg_dl" id="i77">登录</a>
      </div>
      <form name="tijiao" method="post" onsubmit="return check()" action="">
          {% csrf_token %}
          <div class="left"></div>
          <div class="pg_body">
                  <div class="menu">用户名:</div>
                      <div class="kong">
                          <input id="text1" type="text" name="username" placeholder="请输入用户名" onblur="check()"><span id="div1" class="tian" style="margin-top: 4px">*(为必填)</span>
                      </div>

                  <div class="menu">密码:</div>
                      <div class="kong">
                          <input id="text2" type="password" name="userpass" onblur="check()">
                          <span id="div2" class="tian" style="margin-top: 5px">*(为必填)</span>
                      </div>
                 {% comment %} <div class="menu">确认密码:</div>
                      <div class="kong">
                          <input id="text3" type="password" name="01" onblur="check()">
                          <span id="div3" class="tian" style="margin-top: 5px">*(为必填)</span>
                      </div>
                  <div class="menu">邮箱地址:</div>
                      <div class="kong">
                          <input id="text4" type="text" name="email" onblur="check()" required>
                          <span id="div4" class="tian" style="margin-top: 5px">*(为必填)</span>
                      </div>
          {% endcomment %}
          </div>
          <div class="can">
              <input id="i111" type="submit" name="002" value="注  册">
              <p style="width: 200px;display: inline-block;"></p>
              <input id="i222" type="button" name="004" value="取  消">
          </div>
      </form>
      <script type="text/javascript">
              //刷新or取消
              document.getElementById('i77').onclick = function(){
                  location.reload();
              }
              document.getElementById('i222').onclick = function(){
                  location.reload();
              }

              //用户名验证
              function checkname(){
                  var div = document.getElementById("div1");
                  div.innerHTML = "";
                  var name1 = document.tijiao.text1.value;
                  if (name1 == "") {
                  div.innerHTML = "用户名不能为空！";
                  document.tijiao.text1.focus();
                  return false;
              }
                  if (name1.length < 4 || name1.length > 16) {
                  div.innerHTML = "长度4-16个字符";
                  document.tijiao.text1.select();
                  return false;
              }
                  var charname1 = name1.toLowerCase();
                  for (var i = 0; i < name1.length; i++) {
                  var charname = charname1.charAt(i);
                  if (!(charname >= 0 && charname <= 9) && (!(charname >= 'a' && charname <= 'z')) && (charname != '_')) {
                      div.innerHTML = "用户名包含非法字符";
                      document.form1.text1.select();
                      return false;
                  }
              }
                  return true;
          }

              //密码验证
              function checkpassword(){
                  var div = document.getElementById("div2");
                  div.innerHTML = "";
                  var password = document.tijiao.text2.value;
                  if (password == "") {
                  div.innerHTML = "密码不能为空";
                  {#document.tijao.text2.focus();#}
                  return false;
              }
                  if (password.length < 4 || password.length > 16) {
                  div.innerHTML = "密码长度为4-16位";
                  document.tijiao.text2.select();
                  return false;
                  }
                  return true;
          }

              function checkrepassword(){
                  var div = document.getElementById("div3");
                  div.innerHTML = "";
                  var password = document.tijiao.text2.value;
                  var repass = document.tijiao.text3.value;
                  if (repass == "") {
                  div.innerHTML = "密码不能为空";
                  document.tijiao.text3.focus();
                  return false;
                  }
                  if (password != repass) {
                  div.innerHTML = "密码不一致";
                  document.tijiao.text3.select();
                  return false;
                  }
                  return true;
              }
          //邮箱验证
          function checkEmail(){
              var div = document.getElementById("div4");
              div.innerHTML = "";
              var email = document.tijiao.text4.value;
              var sw = email.indexOf("@", 0);
              var sw1 = email.indexOf(".", 0);
              var tt = sw1 - sw;
              if (email.length == 0) {
              div.innerHTML = "邮箱不能为空";
              document.tijiao.text5.focus();
              return false;
              }

              if (email.indexOf("@", 0) == -1) {
              div.innerHTML = "必须包含@符号";
              document.tijiao.text5.select();
              return false;
              }

              if (email.indexOf(".", 0) == -1) {
              div.innerHTML = "必须包含.符号";
              document.tijiao.text5.select();
              return false;
              }

              if (tt == 1) {
                  div.innerHTML = "@和.不能一起";
                  document.tijiao.text5.select();
                  return false;
              }

              if (sw > sw1) {
                  div.innerHTML  = "@符号必须在.之前";
                  document.tijiao.text5.select();
                  return false;
              }
              else {
                  return true;
              }
              return ture;
              }

              function check(){
              if (checkname() && checkpassword() && checkrepassword() && checkEmail()) {
              return true;
              }
              else {
              return false;
          }
      }
      </script>
  </body>
  </html>
  ```

+ index.html

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  <h1>首页</h1>
  <hr>
  <h3>
  {% if request.user.is_authenticated %}
      <span>欢迎：{{ request.user.username }}|<a href="/logout/">退出登录</a>|<a href="/update_password/">修改密码</a></span>
  {% else %}
      <span><a href="/login/">登录</a>|<a href="/register/">注册</a></span>
  {% endif %}
  </h3>
  </body>
  </html>
  ```

### 6、退出登录

+ views.py

  ```python
  from django.contrib.auth import logout
  
  # 退出登录
  def user_logout(req):
      logout(req)
      return redirect('/')
  ```

### 7、修改密码

- 方法一

```python
@login_required(login_url='/login/')
def update_password(req):
    if req.method == 'POST':
        old_userpass = req.POST.get('old_userpass')
        new_userpass = req.POST.get('new_userpass')
        username = req.user.username
        u = authenticate(username=username, password=old_userpass)
        if u:
            # 修改密码
            u.set_password(new_userpass)
            u.save()
            return redirect('/logout/')
        else:
            return redirect('/update_password/')
    return render(req, 'update_password.html')
```

- 方法二

配置全局login_url

settings.py

```
LOGIN_URL = '/login/'
```

views.py

```python
@login_required  # 不在装饰器后写路径的话就需要配置全局login_url
def update_password(req):
    if req.method == 'POST':
        old_userpass = req.POST.get('old_userpass')
        new_userpass = req.POST.get('new_userpass')
        username = req.user.username
        u = authenticate(username=username, password=old_userpass)
        if u:
            # 修改密码
            u.set_password(new_userpass)
            u.save()
            return redirect('/logout/')
        else:
            return redirect('/update_password/')
    return render(req, 'update_password.html')
```

### 8、配置优美前端页面

安装扩展库：

+ pip install django-bootstrap3

bootstrap官网：https://www.bootcss.com/

### 9、message组件（消息闪现）

+ 说明

  当某个动作处理完进行消息通知

+ 方法

  + message.debug
  + message.info
  + message.success
  + message.warning
  + message.error

+ 导入

  ```
  from django.contrib import messages
  ```

+ 使用

  ```python
  def index(req):
      messages.success(req, 'index的消息')
      return render(req, 'index.html')
  ```

### 10 、自定义用户表

- 给 auth_user 表添加字段

- models.py

  ```python
  from django.contrib.auth.models import AbstractUser
  from django.db import models
  # Create your models here.

  class User(AbstractUser):
      iphone = models.CharField(max_length=11, default='111111')
      icon = models.CharField(max_length=20, default='default.jpg')
      class Meta:
          db_table = 'user'
  ```

- 配置、将模型类设置为自定义的类

  ```python
  AUTH_USER_MODEL = 'App.User'
  ```

- 执行迁移

  python manage.py makemigrations

  python manage.py migrate

- 添加自定义认证 （可以使用用户名和手机号码都可以登录）

  App 下创建 auth.py

  ```python
  from django.contrib.auth.backends import ModelBackend
  from django.db.models import Q
  from App.models import User


  class MyBackend(ModelBackend):
      def authenticate(self, request, username=None, password=None, **kwargs):
          try:
              u = User.objects.get(Q(username=username)|Q(iphone=username)|Q(email=username))
          except:
              return None
          if u.check_password(password):
              return u
  ```

- settings.py

  ```
  AUTHENTICATION_BACKENDS = ['App.auth.MyBackend']
  ```

- views.py

  ```python
  from App.models import User
  User.objects.create_user(username=username, email=email, password=userpass, iphone='12345678901')
  ```



## 六、文件上传

### 1、概述

文件存储在服务器端，存储在 request.FILES 中

### 2、注意

- form表单更改enctype

  enctype="multipart/form-data"

- 文件上传必须使用 POST 方式

- 注意 name 名称

- 限制文件上传类型

  https://blog.csdn.net/u013379933/article/details/77119796

  ```html
  <input type="file" name="pic" id="pic" accept="image/gif, image/jpeg" />
  ```

- 多文件上传

  multiple

### 3、配置文件上传路径

- 在 settings.py

  ```python
  STATICFILE_DIRS = [os.path.join(BASE_DIR, 'static')] # (前面模板中配置静态资源文件时就进行配置的)
  # 文件上传路径
  MEDIA_ROOT = os.path.join(BASE_DIR, r'App/static/upload')
  # 文件类型限制
  ALLOWED_FILES = ['jpg', 'jpeg', 'png']
  ```

### 4、文件上传的属性和方法

**属性**

- file.name 文件名称
- file.size 文件大小

**方法**

- file.read() 读取文件上传的所有数据，当前方法适合于小文件上传

- file.chunks() 按块返回文件，在循环迭代中，将大块文件拆分成小块写入到文件中

- file.multiple_chunks()

  房钱方法根据文件大小返回 True 或 False，当 fileSize 大于 2.5M，返回 True；否则返回 False，大小可调整

### 5、原始文件上传（单文件上传）

- upload_file.html

  ```
  {% extends 'common/base.html' %}
  {% block title %}
  文件上传
  {% endblock %}
  {% block content %}
      <h1>文件上传</h1>
      <form action="" method="post" enctype="multipart/form-data">
      {% csrf_token %}
          <input type="file" name="file" accept="image/*" required>
          <br>
          <button  type="submit" class="btn btn-success">上传</button>
      </form>

  {% endblock %}
  ```

- views.py

  ```python
  import os
  from django.conf import settings
  from django.contrib import messages
  from datetime import datetime
  from PIL import Image
  from django.shortcuts import redirect, render
  
  
  # 文件类型限制
  def allowed_file(suffix):
      if suffix in settings.ALLOWED_FILES:
          return True
      else:
          return False
      # return suffix in settings.ALLOWED_FILES
  
  
  # 图片重命名
  def rename_file(suffix):
      # 获取当前时间
      nowTime = datetime.now().strftime("%Y%m%d_%H%M%S")
      # 格式为 "20211013_161620.jpg"
      return nowTime + '.' + suffix
  
  
  # 图片缩放
  def image_zoom_rename(path, perfix='django_', w=200, h=150):
      # 打开图片
      img = Image.open(path)
      # 重新设置图片宽高
      img2 = img.resize((w, h))
      # 取消结尾文件名，只保留路径
      fileList = os.path.split(path)
      print(fileList)
      # 拼凑缩放后图片的名称，格式为 "/路径/django_20211013_161620.jpg"
      newFile = os.path.join(fileList[0], perfix + fileList[1])
      # 保存图片
      img2.save(newFile)
      return True
  
  
  # 上传单个文件
  def uploadOne(request):
      if request.method == 'POST':
          # 获取文件上传对象
          file = request.FILES.get('file')
          # print(file.name)
          # print(file.size)
          # 后缀获取
          suffix = file.name.split('.')[-1]
          # 类型限定
          if not allowed_file(suffix):
              messages.error(request, '文件类型不允许上传')
              return redirect('/upload_one/')
          # 生成随机文件名
          randomFileName = rename_file(suffix)
          # 拼接文件保存路径
          path = os.path.join(settings.MEDIA_ROOT, randomFileName)
          try:
              with open(path, 'wb') as f:
                  if file.multiple_chunks():
                      for ff in file.multiple_chunks():
                          f.write(ff)
                  else:
                      f.write(file.read())
              # 图片缩放处理
              image_zoom_rename(path)
              # 指定宽高和前缀
              # image_zoom_rename(path, perfix='python_', w=200, h=300)
              messages.success(request, '文件上传成功！')
          except:
              messages.error(request, '文件上传失败！')
      return render(request, 'upload_file.html')

### 6、多文件上传

- views.py


  ```python
# 上传多个文件
def uploadMany(request):
    if request.method == 'POST':
        # 获取文件上传对象
        fileList = request.FILES.getlist('file')
        print(fileList)
        try:
            for file in fileList:
                # 拼接文件保存路径
                path = os.path.join(settings.MEDIA_ROOT, file.name)
                print(path)
                with open(path, 'wb') as fw:
                    if file.multiple_chunks():
                        for ff in file.chunks():
                            fw.write(ff)
                    else:
                        fw.write(file.read())
            messages.success(request, '文件上传成功！')
        except:
            messages.error(request, '文件上传失败！')
    return render(request, 'upload_many.html')
  ```

- upload_many.html

  ```python
  {% extends 'common/base.html' %}
  {% block title %}
  文件上传
  {% endblock %}
  {% block content %}
      <h1>多文件上传</h1>
      <form action="" method="post" enctype="multipart/form-data">
      {% csrf_token %}
          <input type="file" name="file" accept="image/*" required multiple>
          <br>
          <button  type="submit" class="btn btn-success">上传</button>
      </form>
  
  {% endblock %}
  ```

> 可以结合登录注册制作一个人中心，登录后点击右上角的头像即打开个人中心，展示自己的用户名，性别，头像信息，修改密码选项



### 7、使用模块类上传

+ models.py

  不使用 upload_to，上传到 settings.py 中的 MEDIA_ROOT指定的目录

  ```python
  from django.contrib.auth.models import AbstractUser
  from django.db import models
  # Create your models here.
      
  class Upload(models.Model):
      # 不指定路径则为settings.py MEDIA_ROOT
      img = models.ImageField(max_length=200)
      file = models.FileField(max_length=200)
      class Meta:
          db_table = 'upload'
  ```

  指定upload_to，则上传到 settings.py 中的 MEDIA_ROOT指定的目录之后再次拼接路径

  ```python
  from django.db import models
  
  class Upload(models.Model):
      # 上传到settings.py MEDIA_ROOT路径/年/月/日
      # img = models.ImageField(max_length=200, upload_to='%Y/%m/%d/')
      # file = models.FileField(max_length=200, upload_to='%Y/%m/%d/')
      img = models.ImageField(max_length=200, upload_to='test/')
      file = models.FileField(max_length=200, upload_to='test/')
      class Meta:
          db_table = 'upload'
  ```

   > 使用 upload_to 指定目录，将文件名存入数据库表中，会将 upload_to 指定的路径 + 文件名一起存储在表中
   >
   > ```python
   > mysql> select * from upload;
   > +----+-------------------------------+-----------------------------+
   > | id | img                           | file                        |
   > +----+-------------------------------+-----------------------------+
   > |  1 | 20210922083503.jpg            | 面试问题.txt                |
   > |  2 | 2021/10/14/20210922083503.jpg | 2021/10/14/面试问题.txt     |
   > |  3 | test/20210922083503.jpg       | test/面试问题.txt           |
   > +----+-------------------------------+-----------------------------+
   > 
   > ```

+ 注意

  如果使用了ImageField，需要安装pillow

+ views.py

  ```python
  def upload_one(req):
      if req.method == 'POST':
          img = req.FILES.get('img')
          file = req.FILES.get('file')
          up = Upload(img=img, file=file)
          # 对图片重命名
          # up.img.name = random.randint(1232)
          up.save()
          print(up.img)
          print(up.img.name)
          print(up.img.url)
          messages.success(req, '上传成功')
      return render(req, 'own_center/upload.html')
  ```
  

- models_upload.html

  ```html
  {% extends 'common/base.html' %}
  {% block title %}
      文件上传
  {% endblock %}
  {% block content %}
      <h1>模型类单文件上传</h1>
      <form action="" method="post" enctype="multipart/form-data">
          {% csrf_token %}
          <input type="file" name="img" accept="image/*" required>
          <br>
          <br>
          <br>
          <input type="file" name="file" required>
          <br>
          <button type="submit" class="btn btn-success">上传</button>
      </form>
  
  {% endblock %}
  ```

  > 作业：
  >
  > 完成如果上传的是文件   保存到upload/file/y/m/d/中  如果是图片则保存到 upload/image/y/m/d/
  >
  > file.content_type   获取文件上传类型
  >
  > 我这里是用了 upload_to 的设置，去限制了图片和文件分别存储的目录，因为前端文件中对选项也做了限制，第一个选项只能选择图片类型
  >
  > - 模型类
  >
  >   models.py
  >
  > ```python
  > from django.db import models
  > 
  > # 使用 upload_to
  > class Upload(models.Model):
  >     # 上传到settings.py MEDIA_ROOT路径/年/月/日
  >     # img = models.ImageField(max_length=200, upload_to='%Y/%m/%d/')
  >     # img = models.ImageField(max_length=200, upload_to='test/')
  >     # file = models.FileField(max_length=200, upload_to='test/')
  >     # file = models.FileField(max_length=200, upload_to='%Y/%m/%d/')
  >     img = models.ImageField(max_length=200, upload_to='images/%Y/%m/%d/')
  >     file = models.FileField(max_length=200, upload_to='file/%Y/%m/%d/')
  > 
  > 
  >     class Meta:
  >         db_table = 'upload'
  > ```
  >
  > - 视图文件
  >
  >   views.py
  >
  >   ```python
  >   from django.contrib import messages
  >   from django.shortcuts import render
  >   from App.models.models_upload_file import Upload
  >     
  >     def uploadOne(request):
  >         if request.method == 'POST':
  >             img = request.FILES.get('img')
  >             file = request.FILES.get('file')
  >             # 图片类型存到images目录下，文件存到file目录下
  >             # if img.content_type != 'image/jpeg':
  >             #     messages.error(request, '类型不允许')
  >             up = Upload(img=img, file=file)
  >             up.save()
  >             # print(up.img)
  >             # print(up.img.url)
  >             messages.success(request, '上传成功')
  >         return render(request, 'file_upload/models_upload.html')
  > 

 
