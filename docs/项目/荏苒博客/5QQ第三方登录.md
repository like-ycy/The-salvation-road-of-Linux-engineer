## QQ登录

我们所说的第三方登录，是指用户可以不在本项目中输入密码，而直接通过第三方的验证，成功登录荏苒网。

要开发QQ第三方登录功能，必须要成为QQ互联开发者后，创建应用，即获取当前项目与QQ互联的应用ID，创建应用的方法参考链接<http://wiki.connect.qq.com/__trashed-2>

![1578390841684](5QQ%E7%AC%AC%E4%B8%89%E6%96%B9%E7%99%BB%E5%BD%95.assets/1578390841684.png)

申请创建开发应用以后, 需要经过人工审核,这个时间一般是一个工作日或者半个工作日. 

审核的状态会影响我们开发者开发第三方登录,状态有三种:

```
1. 不通过和审核中的应用,是无法开发QQ第三方登录功能
2. 只有已通过的应用才可以开发QQ第三方登录功能
```

点击刚才申请的引用,进入到项目里面获取APPID和APP KEY.

![1578391088823](5QQ%E7%AC%AC%E4%B8%89%E6%96%B9%E7%99%BB%E5%BD%95.assets/1578391088823.png)

QQ登录开发文档连接<http://wiki.connect.qq.com/%E5%87%86%E5%A4%87%E5%B7%A5%E4%BD%9C_oauth2-0>

### QQ第三方登录的实现流程

![QQ第三方登录的实现流程.png](5QQ%E7%AC%AC%E4%B8%89%E6%96%B9%E7%99%BB%E5%BD%95.assets/QQ%E7%AC%AC%E4%B8%89%E6%96%B9%E7%99%BB%E5%BD%95%E7%9A%84%E5%AE%9E%E7%8E%B0%E6%B5%81%E7%A8%8B.png)



### 创建模型类

创建一个新的应用oauth，用来实现QQ第三方认证登录。

```python
cd renranapi/apps
python ../../manage.py startapp oauth
```

注册子应用 settings/dev.py,代码:

```python
INSTALLED_APPS = [

    'oauth',
]
```



在models.py中定义QQ身份（openid）与用户模型类User的关联关系

```python
from django.db import models
from renranapi.utils.models import BaseModel
from users.models import User
# Create your models here.
class OauthUser(BaseModel):
    """第三方登录用户信息"""
    NAME_TYPE = (
        (1, "QQ"),
        (2, "微信"),
        (3, "微博"),
    )
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name="openuser", verbose_name='用户')
    openid = models.CharField(max_length=64, unique=True, verbose_name='openid')
    name = models.IntegerField(choices=NAME_TYPE, default=1, verbose_name="社交平台名称")
    access_token = models.CharField(max_length=500,verbose_name="临时访问票据", help_text="有效期:3个月")
    refresh_token = models.CharField(max_length=500,verbose_name="刷新访问票据的token", help_text="当access_token过期以后,可以使用refresh_token来重新获取新的access_token和refresh_token")
    class Meta:
        db_table = 'rr_oauth_user'
        verbose_name = '第三方社交账号登录用户数据'
        verbose_name_plural = verbose_name
```



进行数据库迁移

```shell
python manage.py makemigrations
python manage.py migrate
```



### urllib使用说明

在python中常用于发送http请求的包：`urllib`和`requests`，其中urllib属于python内置的标准包，而requests则是第三方开发出来用于替换urllib包的。

```bash
如果要想在项目中使用requests发送http请求，需要安装requests包
pip install requests
```



在后端接口中，我们需要向QQ服务器发送请求，查询用户的QQ信息，Python提供了标准模块urllib可以帮助我们发送http请求。

- urllib.parse.urlencode(query)

  将query字典转换为url路径中的查询字符串格式

  ```python
  from urllib.parse import urlencode
  query_string = {"参数1":"参数值","参数2":"参数值"}
  query = urlencode(query_string)
  print(query)   # 参数1=参数值&参数2=参数值
  ```

  

- urllib.parse.parse_qs(qs)

  将qs查询字符串格式数据转换为python的字典

  ```python
  参数1=参数值&参数2=参数值     ====> {"参数1":["参数值"],"参数2":["参数值"]}
  ```

  

- urllib.request.urlopen(url, data=None)

  发送http请求

  url表示服务端的url地址，data表示本次http协议的请求体数据。

  如果data为None，则表示发送GET请求，如果data不为None，发送POST请求

  返回值是一个响应对象，可以通过对象.read()读取响应体数据，需要注意读取出的响应体数据为bytes类型



### 代码实现

1.  在settings/dev.py中增加QQ登录相关配置

```python
# QQ登录配置参数
QQ_CONNECT = {
    "app_id": "101403367",
    "app_key": "93112df14c10d6fde74baa62f5de95ab",
    "redirect_uri": "http://www.moluo.net:8080/oauth_callback.html",
    "state": "/", # 用于保存登录成功后的跳转页面路径,查询字符串的参数信息,
}
```

2.  服务端提供QQ登录的链接地址给客户端。

```python
from django.conf import settings
from rest_framework.response import Response
from .utils import OAuthQQ
class QQUserLoginViewSet(ViewSet):
    def url(self,request):
        """获取QQ登录地址"""
        qq = OAuthQQ(**settings.QQ_CONNECT)
        url = qq.url()
        return Response(url)
```

在oauth子应用下，创建utils.py文件，编写QQ登录的辅助类

```python
from urllib.parse import urlencode

class OAuthQQ(object):
    """QQ登录工具类"""
    def __init__(self,app_id,app_key,redirect_uri,state="/"):
        self.app_id = app_id
        self.app_key = app_key
        self.redirect_uri = redirect_uri
        self.state = state

    def url(self):
        """生成QQ登录地址"""
        query_dict = {
            "response_type":"code",
            "client_id": self.app_id,
            "redirect_uri": self.redirect_uri,
            "state": self.state,
        }
        query_string = urlencode(query_dict)

        return "https://graph.qq.com/oauth2.0/authorize?" + query_string
```

子应用路由,代码:

```python
from django.urls import path
from . import views
urlpatterns = [
    path("qq/", views.QQUserLoginViewSet.as_view({"get":"url"}))
]
```

总路由，代码：

```python
    path('oauth/', include("oauth.urls")),
```



#### 客户端请求获取QQ第三方登录地址

修改Login.vue，，在methods中增加qq_login方法

```vue
<template>
    <div class="sign">
    <div class="logo"><router-link to="/"><img src="/static/image/nav-logo.png" alt="Logo"></router-link></div>
    <div class="main">


<h4 class="title">
  <div class="normal-title">
    <router-link class="active" to="/login">登录</router-link>
    <b>·</b>
    <router-link id="js-sign-up-btn" class="" to="/register">注册</router-link>
  </div>
</h4>
<div class="js-sign-in-container">
  <form id="new_session" action="" method="post">
      <div class="input-prepend restyle js-normal">
        <input placeholder="手机号或邮箱" type="text" v-model="username" id="session_email_or_mobile_number">
        <i class="iconfont ic-user"></i>
      </div>

    <div class="input-prepend">
      <input placeholder="密码" type="password" v-model="password" id="session_password">
      <i class="iconfont ic-password"></i>
    </div>
    <div class="remember-btn">
      <input type="checkbox" v-model="remember_me" id="session_remember_me"><span>记住我</span>
    </div>
    <div class="forget-btn">
      <a class="" data-toggle="dropdown" href="">登录遇到问题?</a>
    </div>
    <button class="sign-in-button" id="sign-in-form-submit-btn" type="button" @click="show_captcha">
      <span id="sign-in-loading"></span>
      登录
    </button>
</form>
  <!-- 更多登录方式 -->
  <div class="more-sign">
    <h6>社交帐号登录</h6>
    <ul>
  <li id="weibo-link-wrap" class="">
    <a class="weibo" id="weibo-link">
      <i class="iconfont ic-weibo"></i>
    </a>
  </li>
  <li><a id="weixin" class="weixin" target="_blank" href=""><i class="iconfont ic-wechat"></i></a></li>
  <li><a id="qq" class="qq" target="_blank" href="" @click.prevent.stop="qq_login"><i class="iconfont ic-qq_connect"></i></a></li>
</ul>
  </div>
</div>

    </div>
  </div>
</template>

<script>
    export default {
        name: "Login",
        data(){
          return {
            username: "",
            password: "",
            remember_me: "", // 设置是否要记录登录状态
          }
        },
        methods:{
            qq_login(){
                // QQ登录跳转
                this.$axios.get(`${this.$settings.Host}/oauth/qq/`).then(response=>{
                    // 页面跳转
                    location.href=response.data;
                    // open(response.data,"_blank");
                }).catch(error=>{
                    this.$message.error("对不起,QQ第三方登录暂时没有开通!");
                })
            },
            show_captcha(){
                // 显示验证码

                // 判断手机号或者密码是否为空！
                if(this.username.length<1 || this.password.length<1){
                    // 阻止代码继续往下执行
                    return false;
                }


                var captcha1 = new TencentCaptcha(this.$settings.TC_captcha.app_id, res=>{
                    // 用户操作验证码成功以后的回调函数，这个函数将会在对象创建以后，在页面那种进行监听用户的操作
                    // res就是用户操作成功以后，验证码服务器返回的内容
                    /**
                   res:
                    appid: "2086888489"  # 验证码的APPID
                    randstr: "@G0V"      # 随机字符串，防止重复
                    ret: 0               # 0表示用户操作成功，2表示用户主动关闭验证码窗口
                    ticket: ""           # 验证通过以后的票据，提供给python后端，将来到验证码服务器中进行
                    */
                    this.$axios.get(`${this.$settings.Host}/users/captcha/`,{
                        params:{
                          ticket: res.ticket,
                          randstr: res.randstr
                        }
                    }).then(response=>{
                        if(response.data.detail){
                            // 继续进行登录处理
                            this.loginhandler();
                        }
                    }).catch(error=>{
                        this.$message.error("对不起，验证码校验不通过！");
                    });
                });
                captcha1.show(); // 显示验证码
            },
            loginhandler(){
                // 登录处理
                this.$axios.post(`${this.$settings.Host}/users/login/`,{
                  username: this.username,
                  password: this.password,
                }).then(response=>{
                    // 根据remember_me的值来保存登录状态到本地[vuex，本地存储]
                    if(this.remember_me){
                        // 永久存储
                        localStorage.user_token = response.data.token;
                        localStorage.user_id    = response.data.id;
                        localStorage.user_name  = response.data.username;
                        localStorage.user_avatar= response.data.avatar;
                        sessionStorage.removeItem("user_token");
                        sessionStorage.removeItem("user_id");
                        sessionStorage.removeItem("user_name");
                        sessionStorage.removeItem("user_avatar");
                    }else{
                        // 临时存储
                        sessionStorage.user_token = response.data.token;
                        sessionStorage.user_id = response.data.id;
                        sessionStorage.user_name = response.data.username;
                        sessionStorage.user_avatar = response.data.avatar;
                        localStorage.removeItem("user_token");
                        localStorage.removeItem("user_id");
                        localStorage.removeItem("user_name");
                        localStorage.removeItem("user_avatar");
                    }
                    // 页面跳转
                    this.$confirm('是否跳转到个人中心', '登录成功', {
                      confirmButtonText: '个人中心',
                      cancelButtonText: '返回首页',
                      type: 'success'
                    }).then(() => {
                      // 前往个人中心
                      this.$router.push("/users");
                    }).catch(() => {
                      // 返回站点首页
                      this.$router.push("/");
                    });
                }).catch(error=>{
                    this.$message.error("登录失败！");
                    this.username = "";
                    this.password = "";
                });
            }
        }
    }
</script>

```

然后用户此时就可以在页面中通过点击跳转到QQ第三方登录页面了。



##### BUG

如果实现上面代码以后QQ页面出现如下错误信息：

`对不起，该网站尚未开通QQ帐号登录（错误码：100008）`

则表示当前站点应用没有通过审核。这表示我们的代码没有问题了，但是注册的应用是没通过或者审核中的。

解决方案：使用以下QQ登录应用进行测试开发。

```python
# 1. 在settings/dev.py文件中修改QQ第三方登录配置。
	# QQ登录参数
    QQ_APP_ID = '101403367'
    QQ_APP_KEY = '93112df14c10d6fde74baa62f5de95ab'
    QQ_REDIRECT_URL = 'http://www.moluo.net:8080/oauth_callback.html'
    QQ_STATE = "/" # 用于保存登录成功后的跳转页面路径
    
# 2. 在/etc/hosts/中添加如下配置信息：
    # sudo vim /etc/hosts
    127.0.0.1   www.moluo.net
	# :wq

# 3. 在客户端项目中临时修改config/index.js中的host域名为：www.moluo.net
#    并重启客户端项目

# 4. 在api服务端项目的配置文件settings/dev.py中，添加www.moluo.net到CORS_ORIGIN_WHITELIST列表中，并重启服务端项目
     CORS_ORIGIN_WHITELIST = (
    	'http://www.moluo.net:8080',
     )
```

经过上面的提供的操作，用户在QQ登录成功后，QQ会将用户重定向回我们配置的回调域网址，我们申请QQ登录开发资质时配置的回调地址为：`http://www.renran.cn:8080/login/qq_callback`或者`http://www.moluo.net:8080/oauth_callback.html`。接下来，我们要在客户端对提供一个QQCallBack组件页面，地址绑定为`oauth_callback.html`。

QQCallBack.vue，代码：

```vue
<template>
  <div>QQ登录的回调页面</div>
</template>

<script>
    export default {
        name: "QQCallBack",
    }
</script>

<style scoped>

</style>

```

路由router/index.js中绑定路由

```javascript
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)

import Home from "../components/Home"
import Login from "../components/Login"
import Register from "../components/Register"
import QQCallBack from "../components/QQCallBack"


export default new Router({
  mode: "history",
  routes: [
    {
      path: '/',
      name: "Home",
      component: Home,
    },{
      path: '/login',
      name: "Login",
      component: Login,
    },{
      path: '/register',
      name: "Register",
      component: Register,
    },{
      path: '/oauth_callback.html',
      name: "QQCallBack",
      component: QQCallBack,
    },
  ]
})

```



接收地址栏上面的code参数转发到服务端，服务端再次请求QQ服务器提取访问QQ用户信息的access_token。

客户端代码：

```vue
<template>
    <div>
      QQ回调地址
    </div>
</template>

<script>
    export default {
        name: "QQCallBack",
        data(){

        },
        created(){
            this.get_user_info();
        },
        methods:{
            get_user_info(){

                // 转发code提供给服务端
                this.$axios.get(`${this.$settings.Host}/oauth/qq/info/`+location.search
                ).then(response=>{
                   //
                    console.log("ok");
                }).catch(error=>{
                   this.$message.error("网络错误!无法使用QQ第三方登陆");
                });
            }
        }
    }
</script>

<style scoped>

</style>

```



服务端在oauth/utils.py的OAuthQQ辅助类中新增获取access_token和根据access_token提取openid的方法：

```python
from urllib.parse import urlencode,parse_qs
from urllib.request import urlopen
import json
class QQNetworkError(Exception):
    pass

class OAuthQQ(object):
    """QQ登录工具类"""
    def __init__(self,app_id,app_key,redirect_uri,state="/"):
        self.app_id = app_id
        self.app_key = app_key
        self.redirect_uri = redirect_uri
        self.state = state

    def url(self):
        """生成QQ登录地址"""
        query_dict = {
            "response_type":"code",
            "client_id": self.app_id,
            "redirect_uri": self.redirect_uri,
            "state": self.state,
        }
        query_string = urlencode(query_dict)
        return "https://graph.qq.com/oauth2.0/authorize?" + query_string

    def get_access_token(self,code):
        """根据授权码获取access_token临时访问票据"""
        query_dict = {
            "grant_type":"authorization_code",
            "client_id": self.app_id,
            "client_secret": self.app_key,
            "code": code,
            "redirect_uri": self.redirect_uri,
        }
        query_string = urlencode(query_dict)
        url = "https://graph.qq.com/oauth2.0/token?" + query_string
        try:
            result = urlopen(url)
            content_string = result.read().decode()
            content_dict = parse_qs(content_string)
        except:
            raise QQNetworkError

        return content_dict["access_token"][0],content_dict["refresh_token"][0]

    def get_open_id(self,access_token):
        """根据临时访问票据获取QQ用户的openID"""
        query_dict = {
            "access_token": access_token,
            "fmt": "json",
        }
        query_string = urlencode(query_dict)
        url = "https://graph.qq.com/oauth2.0/me?" + query_string
        try:
            result = urlopen(url)
            content_string = result.read().decode()
            content_dict = json.loads(content_string)
        except:
            raise QQNetworkError

        return content_dict["openid"]
```

接下来根据openID是否保存在数据库中，用户会存在以下三种情况：

>   1.  用户有账号，也非第一次使用QQ登录，则直接保存用户的登录状态完成QQ登录即可。[生成jwt]
>   2.  用户有账号，属于第一次使用QQ登录，则直接进入用户账号和QQ的绑定阶段[密码登录]
>   3.  用户是游客，则让用户注册平台账号并在服务器自动完成新账号和QQ的绑定。[注册]

接下来，我们可以先到数据表rr_auth_qq里面创建一个数据记录。模拟用户已经有账号并且非第一次使用QQ登录。

```
openid   4F65D2442D41B0D1639FD7FC14123B11
user_id  1

# 上面的数据，需要执行视图代码，打印出来的openID，不能照抄上面的。
```

效果：![1583809888836](5QQ%E7%AC%AC%E4%B8%89%E6%96%B9%E7%99%BB%E5%BD%95.assets/1583809888836.png)

#### 服务端对非第一次登录的QQ用户生成登录的jwt

视图调用辅助类完成返回jwt的逻辑代码，oauth/views.py，代码:

```python
from django.shortcuts import render

# Create your views here.
from rest_framework.viewsets import ViewSet
from django.conf import settings
from rest_framework.response import Response
from .utils import OAuthQQ,QQNetworkError
from .models import OauthUser
from users.serializers import UserModelSerializer
from rest_framework import status
class QQUserLoginViewSet(ViewSet):
    def url(self,request):
        """获取QQ登录地址"""
        qq = OAuthQQ(**settings.QQ_CONNECT)
        url = qq.url()
        return Response(url)

    def get_qq_user(self,request):
        """获取登录用户信息"""
        # 1. 获取客户端提交过来的code授权码
        code = request.query_params.get("code")
        state = request.query_params.get("state")
        if code is None:
            return Response({"detail":"对不起,QQ登录授权码丢失,请重新使用QQ登录!"})

        qq = OAuthQQ(**settings.QQ_CONNECT)
        try:
            # 2. 根据授权码获取access_token临时访问票据
            access_token,refresh_token = qq.get_access_token(code)
            # 3. 根据临时访问票据获取QQ用户的openID
            openid = qq.get_open_id(access_token)
            # print(openid)  # 4F65D2442D41B0D1639FD7FC14123B11
        except QQNetworkError:
            return Response({"detail":"网络发生异常,QQ登录失败!"}, status=status.HTTP_500_INTERNAL_SERVER_ERROR)

        # 4. 根据openid到数据库是否有QQ用户信息绑定记录
        try:
            qq_user = OauthUser.objects.get(name=1, openid=openid)
            """表示当前用户已经和QQ进行登录绑定,生成jwt状态即可"""
            # 返回jwt登录状态
            from rest_framework_jwt.settings import api_settings

            jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
            jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

            payload = jwt_payload_handler(qq_user.user)
            qq_user.user.token = jwt_encode_handler(payload)
            serializer = UserModelSerializer(instance=qq_user.user)
            return Response(serializer.data)

        except OauthUser.DoesNotExist:
            """表示当前用户没有进行QQ登录绑定,则让客户端显示绑定QQ和用户信息的表单页面"""

        # 5. 根据数据库查询结果返回登录状态或者进行QQ与用户信息绑定流程
        return Response("ok")
```



#### 客户端实现平台用户非第一次QQ登录的情况.

callbcak.vue组件代码：

```vue
<template>

</template>

<script>
    export default {
        name: "QQCallBack",
        data(){
            return {

            }
        },
        created() {
            this.get_user_info();
        },
        methods:{
            get_user_info(){
              // 接受QQ服务端的回调参数code和state参数
              this.$axios.get(`${this.$settings.Host}/oauth/qq/user/`,{
                  params:{
                      code: this.$route.query.code,
                      state: this.$route.query.state,
                  }
              }).then(response=>{
                  if(response.data.token){
                    // 接受服务端返回的jwt
                    sessionStorage.user_token = response.data.token;
                    sessionStorage.user_id = response.data.id;
                    sessionStorage.user_name = response.data.username;
                    sessionStorage.user_avatar = response.data.avatar;
                    localStorage.removeItem("user_token");
                    localStorage.removeItem("user_id");
                    localStorage.removeItem("user_name");
                    localStorage.removeItem("user_avatar");

                    // 页面跳转
                    this.$confirm('是否跳转到个人中心', '登录成功', {
                      confirmButtonText: '个人中心',
                      cancelButtonText: '返回首页',
                      type: 'success'
                    }).then(() => {
                      // 前往个人中心
                      this.$router.push("/users");
                    }).catch(() => {
                      // 返回站点首页
                      this.$router.push("/");
                    });
                  }

              }).catch(error=>{
                  this.$message({
                      type: "error",
                      duration: 3000,
                      message:"QQ登录失败!",
                      onClose(){
                          this.$router.push("/login");
                      }
                  });
              });
            }
        },
    }
</script>

<style scoped>

</style>
```



接下来,我们实现用户没有绑定QQ账号的情况.

#### 实现用户已有账号，第一次使用QQ登录的账号绑定

接下来，我们先把oauth_qq数据表之前添加的记录删除。删除记录以后，经过上面的代码编写，可以看到此时使用QQ，返回客户端时，页面空白，所以我们需要客户端显示提供用户账号QQ账号的页面。

```vue
<template>
    <div v-show="is_show_page">
      <div class="sign">
        <div class="logo"><a href="/"><img src="/static/image/nav-logo.png" alt="Logo"></a></div>
        <div class="main">
          <h4 class="title">
            <div class="normal-title">
                <a :class="status==1?'active':''" @click="status=1">已有账号</a>
                <b>·</b>
                <a :class="status==2?'active':''" @click="status=2">没有账号</a>
            </div>
          </h4>
          <div class="js-sign-in-container" v-if="status==1">
            <form action="" method="post">
                <div class="input-prepend restyle js-normal">
                  <input placeholder="登录账号或手机号或邮箱" type="text" v-model="username">
                  <i class="iconfont ic-user"></i>
                </div>
                <div class="input-prepend">
                  <input placeholder="密码" type="password" v-model="password">
                  <i class="iconfont ic-password"></i>
                </div>
                <div class="forget-btn">
                  <router-link to="/find_password">通过邮箱找回密码?</router-link>
                </div>
                <button class="sign-in-button" type="button" @click.prevent="show_captcha">
                  <span></span>登录
                </button>
            </form>
          </div>
          <div class="js-sign-in-container" v-if="status==2">
            <form class="new_user" id="new_user" action="" accept-charset="UTF-8" method="post">
              <div class="input-prepend restyle">
                  <input placeholder="你的昵称" type="text" value="" v-model="nickname" id="user_nickname">
                <i class="iconfont ic-user"></i>
              </div>
                <div class="input-prepend restyle no-radius js-normal">
                    <input placeholder="手机号" type="tel" v-model="mobile" id="user_mobile_number">
                  <i class="iconfont ic-phonenumber"></i>
                </div>
              <div class="input-prepend restyle no-radius security-up-code js-security-number" v-if="is_show_sms_code">
                  <input type="text" v-model="sms_code" id="sms_code" placeholder="手机验证码">
                <i class="iconfont ic-verify"></i>
                <a tabindex="-1" class="btn-up-resend js-send-code-button"  href="javascript:void(0);" id="send_code" @click.prevent="send_sms">{{sms_code_text}}</a>
              </div>
              <input type="hidden" name="security_number" id="security_number">
              <div class="input-prepend">
                <input placeholder="设置密码" type="password" v-model="password" id="user_password">
                <i class="iconfont ic-password"></i>
              </div>
              <input type="submit" name="commit" value="注册" class="sign-up-button" id="sign_up_btn" @click.prevent="registerHandler">
              <p class="sign-up-msg">点击 “注册” 即表示您同意并愿意遵守荏苒<br> <a target="_blank" href="">用户协议</a> 和 <a target="_blank" href="">隐私政策</a> 。</p>
            </form>
          </div>
        </div>
      </div>
    </div>
</template>

<script>
    export default {
        name: "QQCallBack",
        data(){
          return {
              is_show_page: false, // 是否显示页面
              status: 1, // 当前用户是否拥有了平台账号
              username: "",
              password: "",
              nickname:"",
              mobile:"",
              sms_code:"",
              sms_code_text:"发送验证码",
              is_show_sms_code:false,
          }
        },
        created(){
            this.get_user_info();
        },
        methods:{
            get_user_info(){
                // 转发code授权码，获取用户信息
                this.$axios.get(`${this.$settings.Host}/oauth/qq/info/`+location.search
                ).then(response=>{
                    if(response.data.token){
                        // 当前用户非第一次使用QQ登录，保存登录状态
                        //3. 处理响应信息
                       sessionStorage.user_token = response.data.token;
                       sessionStorage.user_id = response.data.id;
                       sessionStorage.user_name = response.data.username;
                       sessionStorage.user_avatar = response.data.avatar;
                       sessionStorage.user_nickname = response.data.nickname;
                       let name = response.data.nickname || response.data.username;
                       this.$confirm(`欢迎加入${this.$settings.Website}, ${name}`, '注册成功', {
                          confirmButtonText: '跳转到首页',
                          cancelButtonText: '跳转到个人中心',
                          type: 'success'
                        }).then(() => {
                          this.$router.push("/");
                        }).catch(() => {
                          this.$router.pust("/users");
                        });
                    }else{
                        // 显示绑定账号的页面
                        this.is_show_page = true;
                    }
                }).catch(error=>{
                   this.$message.error("网络错误!无法使用QQ第三方登陆");
                });
            },
            show_captcha(){
                // 显示验证码

                if(this.username.length<1 || this.password.length<1){
                    this.$message.error("对不起,账号或密码不能为空!");
                    return ;
                }

                var captcha1 = new TencentCaptcha(this.$settings.TC_captcha.app_id, res=>{
                    /*
                    * ret	      Int	      验证结果，0：验证成功。2：用户主动关闭验证码。
                      ticket	  String	  验证成功的票据，当且仅当 ret = 0 时 ticket 有值。
                      appid	    String	  场景 ID。
                      bizState	Any	      自定义透传参数。
                      randstr	  String	 本次验证的随机串，请求后台接口时需带上。
                    *
                    * */
                    if(res.ret === 0){
                      this.$axios.get(`${this.$settings.Host}/users/captcha/`,{
                          params:{
                              ret: res.ret,
                              ticket: res.ticket,
                              randstr: res.randstr,
                          }
                      }).then(response=>{
                          if(response.data.message){
                              // 验证成功
                              this.loginHandler();
                          }else{
                              this.$meesage.error("验证码验证失败!请重新操作验证码");
                              captcha1.destroy();
                          }
                      }).catch(error=>{
                          console.log("发生错误!", error);
                      })
                    }
                });
                captcha1.show();
            },
            loginHandler(){
                // 登陆账号完成QQ的绑定

            }
        }
    }
</script>

<style scoped>
input{
  outline: none;
}
*, :after, :before {
    box-sizing: border-box;
}
.sign {
	height: 100%;
	min-height: 750px;
	text-align: center;
	font-size: 14px;
	background-color: #f1f1f1
}

.sign:before {
	content: "";
	display: inline-block;
	height: 85%;
	vertical-align: middle
}

.sign .disable,.sign .disable-gray {
	opacity: .5;
	pointer-events: none
}

.sign .disable-gray {
	background-color: #969696
}

.sign .tooltip-error {
	font-size: 14px;
	line-height: 25px;
	white-space: nowrap;
	background: none
}

.sign .tooltip-error .tooltip-inner {
	max-width: 280px;
	color: #333;
	border: 1px solid #ea6f5a;
	background-color: #fff
}

.sign .tooltip-error .tooltip-inner i {
	position: static;
	margin-right: 5px;
	font-size: 20px;
	color: #ea6f5a;
	vertical-align: middle
}

.sign .tooltip-error .tooltip-inner span {
	vertical-align: middle;
	display: inline-block;
	white-space: normal;
	max-width: 230px
}

.sign .tooltip-error.right .tooltip-arrow-border {
	border-right-color: #ea6f5a
}

.sign .tooltip-error.right .tooltip-arrow-bg {
	left: 2px;
	border-right-color: #fff
}

.sign .slide-error {
	position: relative;
	padding: 10px 0;
	border: 1px solid #c8c8c8;
	border-radius: 4px
}

.sign .slide-error i {
	position: static!important;
	margin-right: 10px;
	color: #ea6f5a!important;
	vertical-align: middle
}

.sign .slide-error span {
	font-size: 15px;
	vertical-align: middle
}

.sign .slide-error div {
	margin-top: 10px;
	font-size: 13px
}

.sign .slide-error a {
	color: #3194d0
}

.sign .js-sign-up-forbidden {
	color: #999;
	padding: 80px 0 100px
}

.sign .js-sign-up-container .slide-error {
	border-bottom: none;
	border-radius: 0
}

.sign .logo {
	position: absolute;
	top: 56px;
	margin-left: 50px
}

.sign .logo img {
	width: 100px
}

.sign .main {
	width: 400px;
	margin: 60px auto 0;
	padding: 50px 50px 30px;
	background-color: #fff;
	border-radius: 4px;
	box-shadow: 0 0 8px rgba(0,0,0,.1);
	vertical-align: middle;
	display: inline-block
}

.sign .reset-title,.sign .title {
	margin: 0 auto 50px;
	padding: 10px;
	font-weight: 400;
	color: #969696
}

.sign .reset-title a,.sign .title a {
	padding: 10px;
	color: #969696
}

.sign .reset-title a:hover,.sign .title a:hover {
	border-bottom: 2px solid #ea6f5a
}

.sign .reset-title .active,.sign .title .active {
	font-weight: 700;
	color: #ea6f5a;
	border-bottom: 2px solid #ea6f5a
}

.sign .reset-title b,.sign .title b {
	padding: 10px
}

.sign .reset-title {
	color: #333;
	font-weight: 700
}

.sign form {
	margin-bottom: 30px
}

.sign form .input-prepend {
	position: relative;
	width: 100%
}

.sign form .input-prepend input {
	width: 100%;
	height: 50px;
	margin-bottom: 0;
	padding: 4px 12px 4px 35px;
	border: 1px solid #c8c8c8;
	border-radius: 0 0 4px 4px;
	background-color: hsla(0,0%,71%,.1);
	vertical-align: middle
}

.sign form .input-prepend i {
	position: absolute;
	top: 14px;
	left: 10px;
	font-size: 18px;
	color: #969696
}

.sign form .input-prepend span {
	color: #333
}

.sign form .input-prepend .ic-show {
	top: 18px;
	left: auto;
	right: 8px;
	font-size: 12px
}

.sign form .geetest-placeholder {
	height: 44px;
	border-radius: 4px;
	background-color: hsla(0,0%,71%,.1);
	text-align: center;
	line-height: 44px;
	font-size: 14px;
	color: #999
}

.sign form .restyle {
	margin-bottom: 0
}

.sign form .restyle input {
	border-bottom: none;
	border-radius: 4px 4px 0 0
}

.sign form .no-radius input {
	border-radius: 0
}

.sign form .slide-security-placeholder {
	height: 32px;
	background-color: hsla(0,0%,71%,.1);
	border-radius: 4px
}

.sign form .slide-security-placeholder p {
	padding-top: 7px;
	color: #999;
	margin-right: -7px
}

.sign .overseas-btn {
	font-size: 14px;
	color: #999
}

.sign .overseas-btn:hover {
	color: #2f2f2f
}

.sign .remember-btn {
	float: left;
	margin: 15px 0
}

.sign .remember-btn span {
	margin-left: 5px;
	font-size: 15px;
	color: #969696;
	vertical-align: middle
}

.sign .forget-btn {
	float: right;
	position: relative;
	margin: 15px 0;
	font-size: 14px
}

.sign .forget-btn a {
	color: #999
}

.sign .forget-btn a:hover {
	color: #333
}

.sign .forget-btn .dropdown-menu {
	top: 20px;
	left: auto;
	right: 0;
	border-radius: 4px
}

.sign .forget-btn .dropdown-menu a {
	padding: 10px 20px;
	color: #333
}

.sign #sign-in-loading {
	position: relative;
	width: 20px;
	height: 20px;
	vertical-align: middle;
	margin-top: -4px;
	margin-right: 2px;
	display: none
}

.sign #sign-in-loading:after {
	content: "";
	position: absolute;
	left: 0;
	top: 0;
	width: 100%;
	height: 100%;
	background-color: transparent
}

.sign #sign-in-loading:before {
	content: "";
	position: absolute;
	top: 50%;
	left: 50%;
	width: 20px;
	height: 20px;
	margin: -10px 0 0 -10px;
	border-radius: 10px;
	border: 2px solid #fff;
	border-bottom-color: transparent;
	vertical-align: middle;
	-webkit-animation: rolling .8s infinite linear;
	animation: rolling .8s infinite linear;
	z-index: 1
}

.sign .sign-in-button,.sign .sign-up-button {
	margin-top: 20px;
	width: 100%;
	padding: 9px 18px;
	font-size: 18px;
	border: none;
	border-radius: 25px;
	color: #fff;
	background: #42c02e;
	cursor: pointer;
	outline: none;
	display: block;
	clear: both
}

.sign .sign-in-button:hover,.sign .sign-up-button:hover {
	background: #3db922
}

.sign .sign-in-button {
	background: #3194d0
}

.sign .sign-in-button:hover {
	background: #187cb7
}

.sign .btn-in-resend,.sign .btn-up-resend {
	position: absolute;
	top: 7px;
	right: 7px;
	width: 100px;
	height: 36px;
	font-size: 13px;
	color: #fff;
	background-color: #42c02e;
	border-radius: 20px;
	line-height: 36px
}

.sign .btn-in-resend {
	background-color: #3194d0
}

.sign .sign-up-msg {
	margin: 10px 0;
	padding: 0;
	text-align: center;
	font-size: 12px;
	line-height: 20px;
	color: #969696
}

.sign .sign-up-msg a,.sign .sign-up-msg a:hover {
	color: #3194d0
}

.sign .overseas input {
	padding-left: 110px!important
}

.sign .overseas .overseas-number {
	position: absolute;
	top: 0;
	left: 0;
	width: 100px;
	height: 50px;
	font-size: 18px;
	color: #969696;
	border-right: 1px solid #c8c8c8
}

.sign .overseas .overseas-number span {
	margin-top: 17px;
	padding-left: 35px;
	text-align: left;
	font-size: 14px;
	display: block
}

.sign .overseas .dropdown-menu {
	width: 100%;
	max-height: 285px;
	font-size: 14px;
	border-radius: 0 0 4px 4px;
	overflow-y: auto
}

.sign .overseas .dropdown-menu li .nation-code {
	width: 65px;
	display: inline-block
}

.sign .overseas .dropdown-menu li a {
	padding: 6px 20px;
	font-size: 14px;
	line-height: 20px
}

.sign .overseas .dropdown-menu li a::hover {
	color: #fff;
	background-color: #f5f5f5
}

.sign .more-sign {
	margin-top: 50px
}

.sign .more-sign h6 {
	position: relative;
	margin: 0 0 10px;
	font-size: 12px;
	color: #b5b5b5
}

.sign .more-sign h6:before {
	left: 30px
}

.sign .more-sign h6:after,.sign .more-sign h6:before {
	content: "";
	border-top: 1px solid #b5b5b5;
	display: block;
	position: absolute;
	width: 60px;
	top: 5px
}

.sign .more-sign h6:after {
	right: 30px
}

.sign .more-sign ul {
	margin-bottom: 10px;
	list-style: none
}

.sign .more-sign ul li {
	margin: 0 5px;
	display: inline-block
}

.sign .more-sign ul a {
	width: 50px;
	height: 50px;
	line-height: 50px;
	display: block
}

.sign .more-sign ul i {
	font-size: 28px
}

.sign .more-sign .ic-weibo {
	color: #e05244
}

.sign .more-sign .ic-wechat {
	color: #00bb29
}

.sign .more-sign .ic-qq_connect {
	color: #498ad5
}

.sign .more-sign .ic-douban {
	color: #00820f
}

.sign .more-sign .ic-more {
	color: #999
}

.sign .more-sign .weibo-loading {
	pointer-events: none;
	cursor: pointer;
	position: relative
}

.sign .more-sign .weibo-loading:after {
	content: "";
	position: absolute;
	left: 0;
	top: 0;
	width: 100%;
	height: 100%;
	background-color: #fff
}

body.reader-night-mode .sign .more-sign .weibo-loading:after {
	background-color: #3f3f3f
}

.sign .more-sign .weibo-loading:before {
	content: "";
	position: absolute;
	top: 50%;
	left: 50%;
	width: 20px;
	height: 20px;
	margin: -10px 0 0 -10px;
	border-radius: 10px;
	border: 2px solid #e05244;
	border-bottom-color: transparent;
	vertical-align: middle;
	-webkit-animation: rolling .8s infinite linear;
	animation: rolling .8s infinite linear;
	z-index: 1
}

@keyframes rolling {
	0% {
		-webkit-transform: rotate(0deg);
		transform: rotate(0deg)
	}

	to {
		-webkit-transform: rotate(1turn);
		transform: rotate(1turn)
	}
}

@-webkit-keyframes rolling {
	0% {
		-webkit-transform: rotate(0deg)
	}

	to {
		-webkit-transform: rotate(1turn)
	}
}

.sign .reset-password-input {
	border-radius: 4px!important
}

.sign .return {
	margin-left: -8px;
	color: #969696
}

.sign .return:hover {
	color: #333
}

.sign .return i {
	margin-right: 5px
}

.sign .icheckbox_square-green {
	display: inline-block;
	*display: inline;
	vertical-align: middle;
	margin: 0;
	padding: 0;
	width: 18px;
	height: 18px;
	background: url(/static/image/green.png) no-repeat;
	border: none;
	cursor: pointer;
	background-position: 0 0
}

.sign .icheckbox_square-green.hover {
	background-position: -20px 0
}

.sign .icheckbox_square-green.checked {
	background-position: -40px 0
}

.sign .icheckbox_square-green.disabled {
	background-position: -60px 0;
	cursor: default
}

.sign .icheckbox_square-green.checked.disabled {
	background-position: -80px 0
}


.geetest_panel_box>* {
	box-sizing: content-box
}

@media (max-width:768px) {
	body {
		min-width: 0
	}

	.sign {
		height: auto;
		min-height: 0;
		background-color: transparent
	}

	.sign .logo {
		display: none
	}

	.sign .main {
		position: absolute;
		left: 50%;
		margin: 0 0 0 -200px;
		box-shadow: none
	}
}
</style>
```

客户端接收服务端因为没有绑定QQ账号而返回的QQ用户信息,里面隐藏了一个data标识openid

```vue
<template>
    <div v-show="is_show_page">
      <div class="sign">
        <div class="logo"><a href="/"><img src="/static/image/nav-logo.png" alt="Logo"></a></div>
        <div class="main">
          <h4 class="title">
            <div class="normal-title">
                <a :class="status==1?'active':''" @click="status=1">已有账号</a>
                <b>·</b>
                <a :class="status==2?'active':''" @click="status=2">没有账号</a>
            </div>
          </h4>
          <div class="js-sign-in-container" v-if="status==1">
            <form action="" method="post">
                <div class="input-prepend restyle js-normal">
                  <input placeholder="登录账号或手机号或邮箱" type="text" v-model="username">
                  <i class="iconfont ic-user"></i>
                </div>
                <div class="input-prepend">
                  <input placeholder="密码" type="password" v-model="password">
                  <i class="iconfont ic-password"></i>
                </div>
                <div class="forget-btn">
                  <router-link to="/find_password">通过邮箱找回密码?</router-link>
                </div>
                <button class="sign-in-button" type="button" @click.prevent="show_captcha">
                  <span></span>登录
                </button>
            </form>
          </div>
          <div class="js-sign-in-container" v-if="status==2">
            <form class="new_user" id="new_user" action="" accept-charset="UTF-8" method="post">
              <div class="input-prepend restyle">
                  <input placeholder="你的昵称" type="text" value="" v-model="nickname" id="user_nickname">
                <i class="iconfont ic-user"></i>
              </div>
                <div class="input-prepend restyle no-radius js-normal">
                    <input placeholder="手机号" type="tel" v-model="mobile" id="user_mobile_number">
                  <i class="iconfont ic-phonenumber"></i>
                </div>
              <div class="input-prepend restyle no-radius security-up-code js-security-number" v-if="is_show_sms_code">
                  <input type="text" v-model="sms_code" id="sms_code" placeholder="手机验证码">
                <i class="iconfont ic-verify"></i>
                <a tabindex="-1" class="btn-up-resend js-send-code-button"  href="javascript:void(0);" id="send_code" @click.prevent="send_sms">{{sms_code_text}}</a>
              </div>
              <input type="hidden" name="security_number" id="security_number">
              <div class="input-prepend">
                <input placeholder="设置密码" type="password" v-model="password" id="user_password">
                <i class="iconfont ic-password"></i>
              </div>
              <input type="submit" name="commit" value="注册" class="sign-up-button" id="sign_up_btn" @click.prevent="registerHandler">
              <p class="sign-up-msg">点击 “注册” 即表示您同意并愿意遵守荏苒<br> <a target="_blank" href="">用户协议</a> 和 <a target="_blank" href="">隐私政策</a> 。</p>
            </form>
          </div>
        </div>
      </div>
    </div>
</template>

<script>
    export default {
        name: "QQCallBack",
        data(){
          return {
              is_show_page: false, // 是否显示页面
              status: 1, // 当前用户是否拥有了平台账号
              username: "",
              password: "",
              nickname:"",
              mobile:"",
              sms_code:"",
              sms_code_text:"发送验证码",
              is_show_sms_code:false,
              openid:"", // QQ登录用户的openid
              access_token:"", // QQ登录用户的access_token
              refresh_token:"", // QQ登录用户的refresh_token
          }
        },
        created(){
            this.get_user_info();
        },
        methods:{
            get_user_info(){
                // 转发code授权码，获取用户信息
                this.$axios.get(`${this.$settings.Host}/oauth/qq/info/`+location.search
                ).then(response=>{
                    if(response.data.token){
                        // 当前用户非第一次使用QQ登录，保存登录状态
                        //3. 处理响应信息
                       sessionStorage.user_token = response.data.token;
                       sessionStorage.user_id = response.data.id;
                       sessionStorage.user_name = response.data.username;
                       sessionStorage.user_avatar = response.data.avatar;
                       sessionStorage.user_nickname = response.data.nickname;
                       let name = response.data.nickname || response.data.username;
                       this.$confirm(`欢迎加入${this.$settings.Website}, ${name}`, '注册成功', {
                          confirmButtonText: '跳转到首页',
                          cancelButtonText: '跳转到个人中心',
                          type: 'success'
                        }).then(() => {
                          this.$router.push("/");
                        }).catch(() => {
                          this.$router.pust("/users");
                        });
                    }else{
                        // 显示绑定账号的页面
                        this.is_show_page = true;
                        this.openid = response.data.openid;
                        this.access_token = response.data.access_token;
                        this.refresh_token = response.data.refresh_token;
                    }
                }).catch(error=>{
                   this.$message.error("网络错误!无法使用QQ第三方登陆");
                });
            },
            show_captcha(){
                // 显示验证码

                if(this.username.length<1 || this.password.length<1){
                    this.$message.error("对不起,账号或密码不能为空!");
                    return ;
                }

                var captcha1 = new TencentCaptcha(this.$settings.TC_captcha.app_id, res=>{
                    /*
                    * ret	      Int	      验证结果，0：验证成功。2：用户主动关闭验证码。
                      ticket	  String	  验证成功的票据，当且仅当 ret = 0 时 ticket 有值。
                      appid	    String	  场景 ID。
                      bizState	Any	      自定义透传参数。
                      randstr	  String	 本次验证的随机串，请求后台接口时需带上。
                    *
                    * */
                    if(res.ret === 0){
                      this.$axios.get(`${this.$settings.Host}/users/captcha/`,{
                          params:{
                              ret: res.ret,
                              ticket: res.ticket,
                              randstr: res.randstr,
                          }
                      }).then(response=>{
                          if(response.data.message){
                              // 验证成功
                              this.loginHandler();
                          }else{
                              this.$meesage.error("验证码验证失败!请重新操作验证码");
                              captcha1.destroy();
                          }
                      }).catch(error=>{
                          console.log("发生错误!", error);
                      })
                    }
                });
                captcha1.show();
            },
            loginHandler(){
                // 登陆账号完成QQ的绑定

            }
        }
    }
</script>
```



客户端发送ajax请求服务端绑定QQ用户

```vue
<template>
    <div v-show="is_show_page">
      <div class="sign">
        <div class="logo"><a href="/"><img src="/static/image/nav-logo.png" alt="Logo"></a></div>
        <div class="main">
          <h4 class="title">
            <div class="normal-title">
                <a :class="status==1?'active':''" @click="status=1">已有账号</a>
                <b>·</b>
                <a :class="status==2?'active':''" @click="status=2">没有账号</a>
            </div>
          </h4>
          <div class="js-sign-in-container" v-if="status==1">
            <form action="" method="post">
                <div class="input-prepend restyle js-normal">
                  <input placeholder="登录账号或手机号或邮箱" type="text" v-model="username">
                  <i class="iconfont ic-user"></i>
                </div>
                <div class="input-prepend">
                  <input placeholder="密码" type="password" v-model="password">
                  <i class="iconfont ic-password"></i>
                </div>
                <div class="forget-btn">
                  <router-link to="/find_password">通过邮箱找回密码?</router-link>
                </div>
                <button class="sign-in-button" type="button" @click.prevent="show_captcha">
                  <span></span>登录
                </button>
            </form>
          </div>
          <div class="js-sign-in-container" v-if="status==2">
            <form class="new_user" id="new_user" action="" accept-charset="UTF-8" method="post">
              <div class="input-prepend restyle">
                  <input placeholder="你的昵称" type="text" value="" v-model="nickname" id="user_nickname">
                <i class="iconfont ic-user"></i>
              </div>
                <div class="input-prepend restyle no-radius js-normal">
                    <input placeholder="手机号" type="tel" v-model="mobile" id="user_mobile_number">
                  <i class="iconfont ic-phonenumber"></i>
                </div>
              <div class="input-prepend restyle no-radius security-up-code js-security-number" v-if="is_show_sms_code">
                  <input type="text" v-model="sms_code" id="sms_code" placeholder="手机验证码">
                <i class="iconfont ic-verify"></i>
                <a tabindex="-1" class="btn-up-resend js-send-code-button"  href="javascript:void(0);" id="send_code" @click.prevent="send_sms">{{sms_code_text}}</a>
              </div>
              <input type="hidden" name="security_number" id="security_number">
              <div class="input-prepend">
                <input placeholder="设置密码" type="password" v-model="password" id="user_password">
                <i class="iconfont ic-password"></i>
              </div>
              <input type="submit" name="commit" value="注册" class="sign-up-button" id="sign_up_btn" @click.prevent="registerHandler">
              <p class="sign-up-msg">点击 “注册” 即表示您同意并愿意遵守荏苒<br> <a target="_blank" href="">用户协议</a> 和 <a target="_blank" href="">隐私政策</a> 。</p>
            </form>
          </div>
        </div>
      </div>
    </div>
</template>


<script>
    export default {
        name: "QQCallBack",
        data(){
            return {
              is_show_page: false, // 控制是否显示账号和QQ的绑定表单,默认不显示
              status: 1,  // 绑定QQ用户的是否已经拥有网站账号,1表示拥有,2表示没有
              username: "",
              password: "",
              nickname:"",
              mobile:"",
              sms_code:"",
              sms_code_text:"发送验证码",
              is_show_sms_code:false,
              openid:"", // QQ登录用户的openid
              access_token:"", // QQ登录用户的access_token
              refresh_token:"", // QQ登录用户的refresh_token
            }
        },
        watch:{
          mobile(){
            if(/^1[3-9]\d{9}$/.test(this.mobile)){
              // 发送ajax请求到服务器验证手机号是否已经注册了
              this.check_mobile();
            }else{
              this.is_show_sms_code = false;
            }
          }
        },
        created() {
            this.get_user_info();
        },
        methods:{
            get_user_info(){
              // 接受QQ服务端的回调参数code和state参数
              this.$axios.get(`${this.$settings.Host}/oauth/qq/user/`,{
                  params:{
                      code: this.$route.query.code,
                      state: this.$route.query.state,
                  }
              }).then(response=>{
                  if(response.data.token){
                    // 接受服务端返回的jwt
                    sessionStorage.user_token = response.data.token;
                    sessionStorage.user_id = response.data.id;
                    sessionStorage.user_name = response.data.username;
                    sessionStorage.user_avatar = response.data.avatar;
                    localStorage.removeItem("user_token");
                    localStorage.removeItem("user_id");
                    localStorage.removeItem("user_name");
                    localStorage.removeItem("user_avatar");

                    // 页面跳转
                    this.$confirm('是否跳转到个人中心', '登录成功', {
                      confirmButtonText: '个人中心',
                      cancelButtonText: '返回首页',
                      type: 'success'
                    }).then(() => {
                      // 前往个人中心
                      this.$router.push("/users");
                    }).catch(() => {
                      // 返回站点首页
                      this.$router.push("/");
                    });
                  }else{
                      // 进入QQ和账号绑定阶段,显示绑定表单
                      this.is_show_page = true;
                      this.openid = response.data.openid;
                      this.access_token = response.data.access_token;
                      this.refresh_token = response.data.refresh_token;
                  }

              }).catch(error=>{
                  let self = this;
                  self.$message({
                      type: "error",
                      duration: 2000,
                      message:"QQ登录失败!请尝试其他登录方式,页面跳转中....",
                      onClose(){
                          self.$router.push("/login");
                      }
                  });
              });
            },
            show_captcha(){
                // 显示验证码

                // 判断手机号或者密码是否为空！
                if(this.username.length<1 || this.password.length<1){
                    // 阻止代码继续往下执行
                    return false;
                }


                var captcha1 = new TencentCaptcha(this.$settings.TC_captcha.app_id, res=>{
                    // 用户操作验证码成功以后的回调函数，这个函数将会在对象创建以后，在页面那种进行监听用户的操作
                    // res就是用户操作成功以后，验证码服务器返回的内容
                    /**
                   res:
                    appid: "2086888489"  # 验证码的APPID
                    randstr: "@G0V"      # 随机字符串，防止重复
                    ret: 0               # 0表示用户操作成功，2表示用户主动关闭验证码窗口
                    ticket: ""           # 验证通过以后的票据，提供给python后端，将来到验证码服务器中进行
                    */
                    this.$axios.get(`${this.$settings.Host}/users/captcha/`,{
                        params:{
                          ticket: res.ticket,
                          randstr: res.randstr
                        }
                    }).then(response=>{
                        if(response.data.detail){
                            // 继续进行登录处理
                            if(this.status==1){
                              this.loginhandler();
                            }else{
                              this.registerHandler();
                            }
                        }
                    }).catch(error=>{
                        this.$message.error("对不起，验证码校验不通过！");
                    });
                });
                captcha1.show(); // 显示验证码
            },
            loginhandler(){
                // 实现QQ和已有账号绑定
                this.$axios.post(`${this.$settings.Host}/oauth/qq/login/`,{
                    username: this.username,
                    password: this.password,
                    openid: this.openid,
                    access_token: this.access_token,
                    refresh_token: this.refresh_token,
                }).then(response=>{
                  if(response.data.token){
                    // 接受服务端返回的jwt
                    sessionStorage.user_token = response.data.token;
                    sessionStorage.user_id = response.data.id;
                    sessionStorage.user_name = response.data.username;
                    sessionStorage.user_avatar = response.data.avatar;
                    localStorage.removeItem("user_token");
                    localStorage.removeItem("user_id");
                    localStorage.removeItem("user_name");
                    localStorage.removeItem("user_avatar");

                    // 页面跳转
                    this.$confirm('是否跳转到个人中心', '登录成功', {
                      confirmButtonText: '个人中心',
                      cancelButtonText: '返回首页',
                      type: 'success'
                    }).then(() => {
                      // 前往个人中心
                      this.$router.push("/users");
                    }).catch(() => {
                      // 返回站点首页
                      this.$router.push("/");
                    });
                  }else{
                      // 进入QQ和账号绑定阶段,显示绑定表单
                      this.is_show_page = true;
                      this.openid = response.data.openid;
                      this.access_token = response.data.access_token;
                      this.refresh_token = response.data.refresh_token;
                  }
                }).catch(error=>{
                  let self = this;
                  self.$message({
                      type: "error",
                      duration: 2000,
                      message:"QQ登录失败!请尝试其他登录方式,页面跳转中....",
                      onClose(){
                          self.$router.push("/login");
                      }
                  });
                })
            },
            send_sms(){
                // 发送短信

            },
            check_mobile(){
                // 验证手机是否已经被注册了
                this.$axios.get(`${this.$settings.Host}/users/mobile/${this.mobile}/`).then(response=>{
                    console.log(response.data);
                    this.is_show_sms_code = true;  // 显示短信验证码框
                }).catch(error=>{
                    this.$message.error("对不起,手机号码已经被注册了!");
                    this.is_show_sms_code = false; // 关闭短信验证码框
                })
            },
            registerHandler(){
              // 进行注册处理并绑定QQ
              
            }
      },
    }
</script>

```



服务端提供绑定QQ用户的API接口

```python
from django.shortcuts import render

# Create your views here.
from rest_framework.viewsets import ViewSet
from django.conf import settings
from rest_framework.response import Response
from .utils import OAuthQQ,QQNetworkError
from .models import OauthUser
from users.serializers import UserModelSerializer
from rest_framework import status
from users.utils import CustomAuthUserModelBackend

class QQUserLoginViewSet(ViewSet):
    def url(self,request):
        """获取QQ登录地址"""
        qq = OAuthQQ(**settings.QQ_CONNECT)
        url = qq.url()
        return Response(url)

    def get_qq_user(self,request):
        """获取登录用户信息"""
        # 1. 获取客户端提交过来的code授权码
        code = request.query_params.get("code")
        state = request.query_params.get("state")
        if code is None:
            return Response({"detail":"对不起,QQ登录授权码丢失,请重新使用QQ登录!"})

        qq = OAuthQQ(**settings.QQ_CONNECT)
        try:
            # 2. 根据授权码获取access_token临时访问票据
            access_token,refresh_token = qq.get_access_token(code)
            # 3. 根据临时访问票据获取QQ用户的openID
            openid = qq.get_open_id(access_token)
            # print(openid)  # 4F65D2442D41B0D1639FD7FC14123B11
        except QQNetworkError:
            return Response({"detail":"网络发生异常,QQ登录失败!"}, status=status.HTTP_500_INTERNAL_SERVER_ERROR)

        # 4. 根据openid到数据库是否有QQ用户信息绑定记录
        try:
            qq_user = OauthUser.objects.get(name=1, openid=openid)
            """表示当前用户已经和QQ进行登录绑定,生成jwt状态即可"""
            # 返回jwt登录状态
            from rest_framework_jwt.settings import api_settings

            jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
            jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

            payload = jwt_payload_handler(qq_user.user)
            qq_user.user.token = jwt_encode_handler(payload)
            serializer = UserModelSerializer(instance=qq_user.user)
            return Response(serializer.data)

        except OauthUser.DoesNotExist:
            """表示当前用户没有进行QQ登录绑定,则让客户端显示绑定QQ和用户信息的表单页面"""
            # 5. 根据数据库查询结果返回登录状态或者进行QQ与用户信息绑定流程
            return Response({
                "access_token":access_token,
                "refresh_token":refresh_token,
                "openid":openid,
            })

    def qq_user_login(self,request):
        """网站用户和QQ进行绑定"""
        # 接收客户端提交
        username = request.data.get("username")
        password = request.data.get("password")
        # 单点调试
        # 查询用户是否存在
        ModelBackend = CustomAuthUserModelBackend()
        user = ModelBackend.authenticate(request, username, password)
        if user is None:
            return Response({"detail":"用户信息不存在或密码错误!"}, status=status.HTTP_400_BAD_REQUEST)
        else:
            """绑定账号和QQ"""
            return self._qq_user_bind(request,user)

    def _qq_user_bind(self,request,user):
        """平台账号绑定QQ信息"""
        access_token = request.data.get("access_token")
        openid = request.data.get("openid")
        refresh_token = request.data.get("refresh_token")
        try:
            OauthUser.objects.create(
                user=user,
                openid=openid,
                name=1,
                access_token=access_token,
                refresh_token=refresh_token,
            )

            # 返回jwt登录状态
            from rest_framework_jwt.settings import api_settings

            jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
            jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

            payload = jwt_payload_handler(user)
            user.token = jwt_encode_handler(payload)
            serializer = UserModelSerializer(instance=user)
            return Response(serializer.data)
        except:
            return Response({"detail": "绑定QQ账号失败!请联系客服工作人员!"}, status=status.HTTP_500_INTERNAL_SERVER_ERROR)

```

指定路由，代码：

```python
from django.urls import path
from . import views
urlpatterns = [
    path("qq/", views.QQUserLoginViewSet.as_view({"get":"url"})),
    path("qq/user/", views.QQUserLoginViewSet.as_view({"get":"get_qq_user"})),
    path("qq/login/", views.QQUserLoginViewSet.as_view({"post":"qq_user_login"})),
]
```



#### 游客使用QQ登录

服务端提供注册平台账号并让新账号绑定QQ的api接口

```python
from django.shortcuts import render

# Create your views here.
from rest_framework.viewsets import ViewSet
from django.conf import settings
from rest_framework.response import Response
from .utils import OAuthQQ,QQNetworkError
from .models import OauthUser
from users.serializers import UserModelSerializer
from rest_framework import status
from users.utils import CustomAuthUserModelBackend

class QQUserLoginViewSet(ViewSet):
    def url(self,request):
        """获取QQ登录地址"""
        qq = OAuthQQ(**settings.QQ_CONNECT)
        url = qq.url()
        return Response(url)

    def get_qq_user(self,request):
        """获取登录用户信息"""
        # 1. 获取客户端提交过来的code授权码
        code = request.query_params.get("code")
        state = request.query_params.get("state")
        if code is None:
            return Response({"detail":"对不起,QQ登录授权码丢失,请重新使用QQ登录!"})

        qq = OAuthQQ(**settings.QQ_CONNECT)
        try:
            # 2. 根据授权码获取access_token临时访问票据
            access_token,refresh_token = qq.get_access_token(code)
            # 3. 根据临时访问票据获取QQ用户的openID
            openid = qq.get_open_id(access_token)
            # print(openid)  # 4F65D2442D41B0D1639FD7FC14123B11
        except QQNetworkError:
            return Response({"detail":"网络发生异常,QQ登录失败!"}, status=status.HTTP_500_INTERNAL_SERVER_ERROR)

        # 4. 根据openid到数据库是否有QQ用户信息绑定记录
        try:
            qq_user = OauthUser.objects.get(name=1, openid=openid)
            """表示当前用户已经和QQ进行登录绑定,生成jwt状态即可"""
            # 返回jwt登录状态
            from rest_framework_jwt.settings import api_settings

            jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
            jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

            payload = jwt_payload_handler(qq_user.user)
            qq_user.user.token = jwt_encode_handler(payload)
            serializer = UserModelSerializer(instance=qq_user.user)
            return Response(serializer.data)

        except OauthUser.DoesNotExist:
            """表示当前用户没有进行QQ登录绑定,则让客户端显示绑定QQ和用户信息的表单页面"""
            # 5. 根据数据库查询结果返回登录状态或者进行QQ与用户信息绑定流程
            return Response({
                "access_token":access_token,
                "refresh_token":refresh_token,
                "openid":openid,
            })

    def qq_user_login(self,request):
        """网站用户和QQ进行绑定"""
        # 接收客户端提交
        username = request.data.get("username")
        password = request.data.get("password")
        # 单点调试
        # 查询用户是否存在
        ModelBackend = CustomAuthUserModelBackend()
        user = ModelBackend.authenticate(request, username, password)
        if user is None:
            return Response({"detail":"用户信息不存在或密码错误!"}, status=status.HTTP_400_BAD_REQUEST)
        else:
            """绑定账号和QQ"""
            return self._qq_user_bind(request,user)

    def qq_user_register(self,request):
        """游客注册账号和QQ进行绑定"""
        # 接收客户端提交
        nickname = request.data.get("nickname")
        mobile = request.data.get("mobile")
        password = request.data.get("password")
        sms_code = request.data.get("sms_code")

        serializers = UserModelSerializer(data={
            "nickname":nickname,
            "mobile":mobile,
            "password":password,
            "sms_code":sms_code,
        })

        serializers.is_valid(raise_exception=True)
        user = serializers.save()
        return self._qq_user_bind(request,user)

    def _qq_user_bind(self,request,user):
        """平台账号绑定QQ信息"""
        access_token = request.data.get("access_token")
        openid = request.data.get("openid")
        refresh_token = request.data.get("refresh_token")
        try:
            OauthUser.objects.create(
                user=user,
                openid=openid,
                name=1,
                access_token=access_token,
                refresh_token=refresh_token,
            )

            # 返回jwt登录状态
            from rest_framework_jwt.settings import api_settings

            jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
            jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

            payload = jwt_payload_handler(user)
            user.token = jwt_encode_handler(payload)
            serializer = UserModelSerializer(instance=user)
            return Response(serializer.data)
        except:
            return Response({"detail": "绑定QQ账号失败!请联系客服工作人员!"}, status=status.HTTP_500_INTERNAL_SERVER_ERROR)

```

路由代码：

```python
from django.urls import path
from . import views
urlpatterns = [
    path("qq/", views.QQUserLoginViewSet.as_view({"get":"url"})),
    path("qq/user/", views.QQUserLoginViewSet.as_view({"get":"get_qq_user"})),
    path("qq/login/", views.QQUserLoginViewSet.as_view({"post":"qq_user_login"})),
    path("qq/reg/", views.QQUserLoginViewSet.as_view({"post":"qq_user_register"})),
]
```



前端发送ajax，提供注册信息和openID。代码：

```vue
<template>
    <div v-show="is_show_page">
      <div class="sign">
        <div class="logo"><a href="/"><img src="/static/image/nav-logo.png" alt="Logo"></a></div>
        <div class="main">
          <h4 class="title">
            <div class="normal-title">
                <a :class="status==1?'active':''" @click="status=1">已有账号</a>
                <b>·</b>
                <a :class="status==2?'active':''" @click="status=2">没有账号</a>
            </div>
          </h4>
          <div class="js-sign-in-container" v-if="status==1">
            <form action="" method="post">
                <div class="input-prepend restyle js-normal">
                  <input placeholder="登录账号或手机号或邮箱" type="text" v-model="username">
                  <i class="iconfont ic-user"></i>
                </div>
                <div class="input-prepend">
                  <input placeholder="密码" type="password" v-model="password">
                  <i class="iconfont ic-password"></i>
                </div>
                <div class="forget-btn">
                  <router-link to="/find_password">通过邮箱找回密码?</router-link>
                </div>
                <button class="sign-in-button" type="button" @click.prevent="show_captcha">
                  <span></span>登录
                </button>
            </form>
          </div>
          <div class="js-sign-in-container" v-if="status==2">
            <form class="new_user" id="new_user" action="" accept-charset="UTF-8" method="post">
              <div class="input-prepend restyle">
                  <input placeholder="你的昵称" type="text" value="" v-model="nickname" id="user_nickname">
                <i class="iconfont ic-user"></i>
              </div>
                <div class="input-prepend restyle no-radius js-normal">
                    <input placeholder="手机号" type="tel" v-model="mobile" id="user_mobile_number">
                  <i class="iconfont ic-phonenumber"></i>
                </div>
              <div class="input-prepend restyle no-radius security-up-code js-security-number" v-if="is_show_sms_code">
                  <input type="text" v-model="sms_code" id="sms_code" placeholder="手机验证码">
                <i class="iconfont ic-verify"></i>
                <a tabindex="-1" class="btn-up-resend js-send-code-button"  href="javascript:void(0);" id="send_code" @click.prevent="send_sms">{{sms_code_text}}</a>
              </div>
              <input type="hidden" name="security_number" id="security_number">
              <div class="input-prepend">
                <input placeholder="设置密码" type="password" v-model="password" id="user_password">
                <i class="iconfont ic-password"></i>
              </div>
              <input type="submit" name="commit" value="注册" class="sign-up-button" id="sign_up_btn" @click.prevent="registerHandler">
              <p class="sign-up-msg">点击 “注册” 即表示您同意并愿意遵守荏苒<br> <a target="_blank" href="">用户协议</a> 和 <a target="_blank" href="">隐私政策</a> 。</p>
            </form>
          </div>
        </div>
      </div>
    </div>
</template>


<script>
    export default {
        name: "QQCallBack",
        data(){
            return {
              is_show_page: false, // 控制是否显示账号和QQ的绑定表单,默认不显示
              status: 1,  // 绑定QQ用户的是否已经拥有网站账号,1表示拥有,2表示没有
              username: "",
              password: "",
              nickname:"",
              mobile:"",
              sms_code:"",
              sms_code_text:"发送验证码",
              is_show_sms_code:false,
              openid:"", // QQ登录用户的openid
              access_token:"", // QQ登录用户的access_token
              refresh_token:"", // QQ登录用户的refresh_token
            }
        },
        watch:{
          mobile(){
            if(/^1[3-9]\d{9}$/.test(this.mobile)){
              // 发送ajax请求到服务器验证手机号是否已经注册了
              this.check_mobile();
            }else{
              this.is_show_sms_code = false;
            }
          }
        },
        created() {
            this.get_user_info();
        },
        methods:{
            get_user_info(){
              // 接受QQ服务端的回调参数code和state参数
              this.$axios.get(`${this.$settings.Host}/oauth/qq/user/`,{
                  params:{
                      code: this.$route.query.code,
                      state: this.$route.query.state,
                  }
              }).then(response=>{
                  if(response.data.token){
                    // 接受服务端返回的jwt
                    sessionStorage.user_token = response.data.token;
                    sessionStorage.user_id = response.data.id;
                    sessionStorage.user_name = response.data.username;
                    sessionStorage.user_avatar = response.data.avatar;
                    localStorage.removeItem("user_token");
                    localStorage.removeItem("user_id");
                    localStorage.removeItem("user_name");
                    localStorage.removeItem("user_avatar");

                    // 页面跳转
                    this.$confirm('是否跳转到个人中心', '登录成功', {
                      confirmButtonText: '个人中心',
                      cancelButtonText: '返回首页',
                      type: 'success'
                    }).then(() => {
                      // 前往个人中心
                      this.$router.push("/users");
                    }).catch(() => {
                      // 返回站点首页
                      this.$router.push("/");
                    });
                  }else{
                      // 进入QQ和账号绑定阶段,显示绑定表单
                      this.is_show_page = true;
                      this.openid = response.data.openid;
                      this.access_token = response.data.access_token;
                      this.refresh_token = response.data.refresh_token;
                  }

              }).catch(error=>{
                  let self = this;
                  self.$message({
                      type: "error",
                      duration: 2000,
                      message:"QQ登录失败!请尝试其他登录方式,页面跳转中....",
                      onClose(){
                          self.$router.push("/login");
                      }
                  });
              });
            },
            show_captcha(){
                // 显示验证码

                // 判断手机号或者密码是否为空！
                if(this.username.length<1 || this.password.length<1){
                    // 阻止代码继续往下执行
                    return false;
                }


                var captcha1 = new TencentCaptcha(this.$settings.TC_captcha.app_id, res=>{
                    // 用户操作验证码成功以后的回调函数，这个函数将会在对象创建以后，在页面那种进行监听用户的操作
                    // res就是用户操作成功以后，验证码服务器返回的内容
                    /**
                   res:
                    appid: "2086888489"  # 验证码的APPID
                    randstr: "@G0V"      # 随机字符串，防止重复
                    ret: 0               # 0表示用户操作成功，2表示用户主动关闭验证码窗口
                    ticket: ""           # 验证通过以后的票据，提供给python后端，将来到验证码服务器中进行
                    */
                    this.$axios.get(`${this.$settings.Host}/users/captcha/`,{
                        params:{
                          ticket: res.ticket,
                          randstr: res.randstr
                        }
                    }).then(response=>{
                        if(response.data.detail){
                            // 继续进行登录处理
                            if(this.status==1){
                              this.loginhandler();
                            }else{
                              this.registerHandler();
                            }
                        }
                    }).catch(error=>{
                        this.$message.error("对不起，验证码校验不通过！");
                    });
                });
                captcha1.show(); // 显示验证码
            },
            loginhandler(){
                // 实现QQ和已有账号绑定
                this.$axios.post(`${this.$settings.Host}/oauth/qq/login/`,{
                    username: this.username,
                    password: this.password,
                    openid: this.openid,
                    access_token: this.access_token,
                    refresh_token: this.refresh_token,
                }).then(response=>{
                  if(response.data.token){
                    // 接受服务端返回的jwt
                    sessionStorage.user_token = response.data.token;
                    sessionStorage.user_id = response.data.id;
                    sessionStorage.user_name = response.data.username;
                    sessionStorage.user_avatar = response.data.avatar;
                    localStorage.removeItem("user_token");
                    localStorage.removeItem("user_id");
                    localStorage.removeItem("user_name");
                    localStorage.removeItem("user_avatar");

                    // 页面跳转
                    this.$confirm('是否跳转到个人中心', '登录成功', {
                      confirmButtonText: '个人中心',
                      cancelButtonText: '返回首页',
                      type: 'success'
                    }).then(() => {
                      // 前往个人中心
                      this.$router.push("/users");
                    }).catch(() => {
                      // 返回站点首页
                      this.$router.push("/");
                    });
                  }else{
                      // 进入QQ和账号绑定阶段,显示绑定表单
                      this.is_show_page = true;
                      this.openid = response.data.openid;
                      this.access_token = response.data.access_token;
                      this.refresh_token = response.data.refresh_token;
                  }
                }).catch(error=>{
                  let self = this;
                  self.$message({
                      type: "error",
                      duration: 2000,
                      message:"QQ登录失败!请尝试其他登录方式,页面跳转中....",
                      onClose(){
                          self.$router.push("/login");
                      }
                  });
                })
            },
            send_sms(){
                // 发送短信
                if(this.mobile.length<1){
                    this.$message.error("对不起,手机号码不能为空!");
                    return false;
                }

                this.$axios.get(`${this.$settings.Host}/users/sms/${this.mobile}/`).then(response=>{
                    this.$message.info(response.data.detail);
                }).catch(error=>{
                    this.$message.error("短信发送错误!请联系客服工作人员!");
                });
            },
            check_mobile(){
                // 验证手机是否已经被注册了
                this.$axios.get(`${this.$settings.Host}/users/mobile/${this.mobile}/`).then(response=>{
                    console.log(response.data);
                    this.is_show_sms_code = true;  // 显示短信验证码框
                }).catch(error=>{
                    this.$message.error("对不起,手机号码已经被注册了!");
                    this.is_show_sms_code = false; // 关闭短信验证码框
                })
            },
            registerHandler(){
              // 进行注册处理并绑定QQ
              this.$axios.post(`${this.$settings.Host}/oauth/qq/reg/`,{
                  mobile: this.mobile,
                  nickname: this.nickname,
                  sms_code: this.sms_code,
                  password: this.password,
                  openid: this.openid,
                  access_token: this.access_token,
                  refresh_token: this.refresh_token,
              }).then(response=>{
                if(response.data.token){
                  // 接受服务端返回的jwt
                  sessionStorage.user_token = response.data.token;
                  sessionStorage.user_id = response.data.id;
                  sessionStorage.user_name = response.data.username;
                  sessionStorage.user_avatar = response.data.avatar;
                  localStorage.removeItem("user_token");
                  localStorage.removeItem("user_id");
                  localStorage.removeItem("user_name");
                  localStorage.removeItem("user_avatar");

                  // 页面跳转
                  this.$confirm('是否跳转到个人中心', '注册成功', {
                    confirmButtonText: '个人中心',
                    cancelButtonText: '返回首页',
                    type: 'success'
                  }).then(() => {
                    // 前往个人中心
                    this.$router.push("/users");
                  }).catch(() => {
                    // 返回站点首页
                    this.$router.push("/");
                  });
                }else{
                    // 进入QQ和账号绑定阶段,显示绑定表单
                    this.is_show_page = true;
                    this.openid = response.data.openid;
                    this.access_token = response.data.access_token;
                    this.refresh_token = response.data.refresh_token;
                }
              }).catch(error=>{
                let self = this;
                self.$message({
                    type: "error",
                    duration: 2000,
                    message:"QQ登录失败!请尝试其他登录方式,页面跳转中....",
                    onClose(){
                        self.$router.push("/login");
                    }
                });
              })
            }
      },
    }
</script>
```

