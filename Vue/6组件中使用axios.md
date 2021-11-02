# axios的使用

在组件中使用axios获取来自服务端的数据

ajax `async javascript and xml`，中文译音：阿贾克斯]，是一种异步页面刷新技术（或叫局部页面刷新技术），默认情况下由浏览器中原生javascript提供了一个XMLHttpRequest对象允许我们在用户不知情的情况下，发送http请求，访问服务端获取数据。

## 原生ajax的使用

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
    .weather table{
        width: 800px;
        border-collapse: collapse;
    }
    .weather table td{
        width: 120px;
        height: 40px;
        line-height: 40px;
        border: 1px solid red;
        text-align: center;
    }
    </style>
</head>
<body>
    <input type="text" id="city" value="北京">
    <button onclick="get_weather()">获取数据</button>
    <div class="weather">
        <table>
            <tr>
                <td>date</td>
                <td>fengxiang~fengli</td>
                <td>wendu</td>
                <td>type</td>
            </tr>
        </table>
    </div>
    <script>
        // 原生ajax请求获取数据
        let city = document.querySelector("#city");
        let table = document.querySelector(".weather table");
        let html  = table.innerHTML;
        const get_weather = ()=>{
            console.log(`获取${city.value}的天气预报！`);
            // 实例化ajax对象
            const xhr = new XMLHttpRequest();
            console.log(xhr.readyState); // 0，ajax对象实例化
            // 设置http请求信息
            xhr.open("GET", `http://wthrcdn.etouch.cn/weather_mini?city=${city.value}`);
            // xhr.setRequestHeader() // 设置请求头信息

            // 发送http请求
            xhr.send()  // 请求体： username=xioaming&password=123456
            console.log(xhr.readyState); // 1 构建完成http请求，并发送
            // 通过ajax的监听事件，监听是否ajax是否已经获取服务端返回的数据结果
            xhr.onreadystatechange = ()=>{
                console.log(xhr.readyState); // 2: http请求已经被服务端接收,3: http请求已经被服务端处理中,
                // 当ajax发生状态改变时，会自动执行当前事件
                if(xhr.readyState==4){ // 4: http请求被服务端处理完成并返回结果给客户端ajax
                    if(xhr.status == 200){ // 判断http响应状态码
                        // 接收结果
                        console.log(xhr.response); // 得到的服务端数据是文本格式的，所以一般还要转换数据的格式
                        data = JSON.parse(xhr.response)
                        console.log(data.data.forecast);
                        let content = "";
                        for(let item of data.data.forecast){
                            console.log(item.date);
                            content += `
                                <tr>
                                    <td>${item.date}</td>
                                    <td>${item.fengxiang}${item.fengli}</td>
                                    <td>${item.low} ~ ${item.high}</td>
                                    <td>${item.type}</td>
                                </tr>
                            `
                        }
                        table.innerHTML = html+content;
                    }
                }
            }
        }
    </script>
</body>
</html>
```

默认情况下，vue中并没有对ajax进行封装的，所以我们需要下载安装axios包。

在项目**根目录**中使用 npm或者yarn安装包

```bash
# npm install axios
yarn add axios
```

接着在src目录下创建一个utils/http.js脚本中，导入axios并通过create方法实例化一个http请求对象，这样我们才能在vue的所有中组件中直接使用。

src/utils/http.js，代码：

```js
import axios from "axios";  // 要导入安装的包，则直接填写包名即可。不需要使用路径

// 实例化
const http = axios.create({
    baseURL: 'http://wthrcdn.etouch.cn/',    // 请求的公共路径,一般填写服务端的默认的api地址，这个地址在具体使用的时候覆盖
    timeout: 8000,                           // 最大请求超时时间，请求超过这个时间则报错，有文件上传的站点不要设置这个参数
    headers: {'X-Custom-Header': 'foobar'}   // 默认的预定义请求头，一般工作中这里填写隐藏了客户端身份的字段
});

export default http;

```



## 在组件中使用axios获取数据

Register.vue，代码：

```vue
<template>
  <h1>注册页面</h1>
  <Menu message="Register"></Menu>
  <button @click="get_weather_info">ajax获取数据</button>
</template>

<script>
import Menu from "../components/Menu";
import httptool from "../http";
export default {
  name: "Register",
  components:{
    Menu,
  },
  methods:{
    get_weather_info1(){
      // 发送get请求查询数据，可以使用delete方法删除服务端数据，参数和返回值与get一样
      // 参数1：url，地址
      // 参数2：config，请求配置，params表示查询字符串，headers表示请求头
      // 返回值是一个Promise对象.可以通过then方法接收返回的结果，也可以通过catch方法接收错误
      httptool.delete("http://wthrcdn.etouch.cn/weather_mini", {
        params: { // 查询字符串参数，?号后面的参数
          city: "北京"
        },
        headers:{
          // 自定义请求头
        }
      }).then(response=>{ // response 就是服务端返回http响应对象
        // console.log(response.data) // 获取服务端返回的响应体数据
        console.log(response.data.data)
      }).catch(error=>{
        console.log(error) // 打印错误信息【客户端报错的情况下】
        console.log(error.response) // 获取来自服务端的错误信息
      })
    },
    get_weather_info(){
      // 使用post方法提交数据，也可以使用put/patch来更新数据，参数与返回值于post一模一样
      // 参数1：url地址，数据提交的服务端地址
      // 参数2：data，请求体数据，json对象
      // 参数3：config，请求配置，params表示查询字符串，headers表示请求头
      // 返回值是一个Promise对象.可以通过then方法接收返回的结果，也可以通过catch方法接收错误
      httptool.post("http://127.0.0.1:8080/home", {
        user: 'root',
        age: 17,
        sex: true,
        money: 1000
      },{
        // params:{},
        // headers:{},
      }).then(response=>{
        console.log(response.data)
      }).catch(error=>{
        console.log(error)
      })
    }
  }
}
</script>

<style scoped>

</style>
```

效果：

![1552202922168](6%E7%BB%84%E4%BB%B6%E4%B8%AD%E4%BD%BF%E7%94%A8axios.assets/1552202922168.png)

使用的时候，因为本质上来说，我们使用axios发起http请求，本质上还是javascript提供的ajax，所以也会和普通的javascript受到<mark>同源策略</mark>的影响。

所谓的同源策略，实际上是浏览器为了保护客户端用户在服务端中的数据的安全性所设置的一种网络安全机制/安全策略，

同源策略，可以禁止不同源下的客户端的js在没有得到服务端授权的情况下操作或调用服务端的数据。

所谓的同源，实际上就是判断通信的客户端服务端，是否处于同一个协议，同一个域名/IP，同一个端口下，如果三者相同，则服务端和客户端属于同源，则同源策略不会拦截客户端的js请求服务端数据，如果三者中任意1个信息不一致，则属于不同源，不同源的客户端js无法访问和操作服务端的数据。

解决同源策略的方式有3种：jsonp，服务端代理，CORS服务端授权

jsonp：使用json作为参数，通过HTML内置的支持跨域的标签，调用服务端的函数在客户端执行。

​            HTML中内置的支持跨域对的标签：img、link、script、媒体标签

CORS，翻译：跨域资源共享



## 常用的axios请求方法

在工作中，我们基本都是通过axios来发起http请求来读取或者上传数据到服务端的。

axios针对http请求提供了多个不同的请求方法。根据是否有请求体，写法上可以分2类：

```javascript
// delete和get用法，参数一致。
axios.get("url地址",{  // url地址如果没有加上协议，则axios会自动根据实例化的baseURL参数自动拼接。
    params:{
        "":"",  // 键值对，查询字符串   地址栏?号后面的信息
    },
    headers:{
        "":"",  // 键值对，自定义请求头
    }
}).then(response=>{
    // 请求成功时，可以通过response.data来获取来自服务端的请求体数据
}).catch(error=>{
    // 异常发生时会自动调用这里的代码，同时第一个参数表示异常对象
    // error.response;   // 获取来自服务端的异常信息，如果服务端没有异常，则response的值为unidentified
});

// post, put, patch 三者用法一致，参数一样。
axios.post("url地址",{
    "":"",  // 请求体数据
},{
    params:{  // 查询字符串
        "":"",
    },
    headers:{ // 自定义请求头
       "":"",
    }
}).then(response=>{
    // 请求成功时，可以通过response.data来获取来自服务端的请求体数据
}).catch(error=>{
    // 异常发生时会自动调用这里的代码，同时第一个参数表示异常对象
    // error.response;   // 获取来自服务端的异常信息，如果服务端没有异常，则response的值为unidentified
});
```

#### GET请求

http.js，代码：

```javascript
import axios from "axios"; // 要导入安装的包，则直接填写包名即可。不需要使用路径

// 实例化
const httptool = axios.create({
      baseURL: 'http://wthrcdn.etouch.cn/', // 请求的公共路径,一般填写服务端的api地址
      timeout: 1000,                           // 最大请求超时时间，请求超过这个时间则报错
      headers: {'X-Custom-Header': 'foobar'}   // 预定义请求头，一般工作中这里填写隐藏了客户端身份的字段
});

export default httptool;
```

Register.vue，代码：

```vue
<template>
  <h1>注册页面</h1>
  <button @click="send">点击发送请求</button>
</template>

<script>
import httptool from "../http";
export default {
  name: "Register",
  components:{

  },
  methods:{
    send(){
      // http://127.0.0.1:8000/student/students/?ordering=-id&age=18
      // http://127.0.0.1:8000/ 填写在http.js中的BaseURL
      // student/students/ 填写在axios的http方法的第一个参数里面
      // ordering=-id 查询字符串
      // httptool.get("url地址",{
      //   params:{},  // 查询字符串
      //   headers:{}, // 自定义请求头
      // });
      // http://wthrcdn.etouch.cn/weather_mini?city=深圳
      httptool.get("weather_mini",{
        params:{    // 查询字符串
          city:"北京",
        },
        headers:{"Company": "vue Company"}, // 自定义请求头
      }).then(response=>{
        console.log("response", response); // http响应头
        console.log("response.data", response.data); // http响应的请求体数据
        console.log("response.data.data", response.data.data); // http响应的请求体数据[.data.data是表示对方提供的数据还有一个data,这个最后的data是服务端自定义的。]
      }).catch(error=>{
        console.log(error);          // error可以是本地错误信息，也可以是服务端的错误信息
        console.log(error.response); // 接收来自服务端的响应错误，如果服务端没有错误，则没有response属性
      });
    }
  }
}
</script>

<style scoped>

</style>
```



#### POST请求

我们先使用原来的server.py，在本地安装了aiohttp和aiohttp_cors以后，运行当前服务器。

```python
pip install aiohttp
pip install aiohttp_cors
```

```python
from aiohttp import web
import aiohttp_cors

# 路由对象
routes = web.RouteTableDef()

@routes.post("/{name}")
async def index(request):
    # 获取url地址的一部分，这个叫路由参数
    name = request.match_info.get('name', "Anonymous")
    # 接受客户端上传过来的请求体数据
    data = await request.json()
    print("接收客户端发送的请求体数据：", data)
    # 返回json数据
    return web.json_response({"name":name,"age": 19,**data})

@routes.get("/")
async def index(request):
    # 返回json数据
    data = {"name":"xiaoming","age": 19}
    return web.json_response(data)

class Application(web.Application):
    def __init__(self, routes):
        """
        :param routes: url和服务端处理程序之间的绑定关系, 这种绑定关系，我们一般称之为路由，当然，在服务端开发中，我们也会经常把具有路由功能功能的类/对象，称之为"路由类"或"路由对象"
        """
        # 先让官方的aiohttp的web框架先进行初始化
        super(Application, self).__init__()
        # 注册路由信息到web框架中,routes是一个list列表
        self.add_routes(routes)
        # 这里内部完成了一个for循环，把routes里面每一个路由信息都了遍历，添加一个返回值用于实现跨域资源共享[CORS]。
        cors = aiohttp_cors.setup(self, defaults={
            "*": aiohttp_cors.ResourceOptions(
                allow_credentials=True,
                expose_headers="*",
                allow_headers="*",
            )
        })

        # Configure CORS on all routes.
        for route in list(self.router.routes()):
            cors.add(route)

if __name__ == '__main__':
    web.run_app(Application(routes), host="127.0.0.1", port=8000)
```

注意：我们需要把当前vue客户端运行在8000端口下，而server.py则运行在8080端口下。

Register.vue，代码：

```vue
<template>
  <Menu :current="1" msg="注册"></Menu>
  <h1>注册页面</h1>
  <button @click="user_register">注册</button>
  <hr>
  <input type="text" v-model="city">
  <button @click="get_weather">获取数据</button>
  <div class="weather">
    <table>
      <tr>
        <td>date</td>
        <td>fengxiang~fengli</td>
        <td>wendu</td>
        <td>type</td>
      </tr>
      <tr v-for="item,key in weathers" :key="key">
        <td>{{item.date}}</td>
        <td>{{item.fengxiang}}{{item.fengli}}</td>
        <td>{{item.low}} ~ {{item.high}}</td>
        <td>{{item.type}}</td>
      </tr>
    </table>
</div>
</template>

<script>
import Menu from "../components/Menu";
import http from "../utils/http";
export default {
  name: "Register",
  data(){
    return {
      city: "北京",
      weathers: [],
    }
  },
  methods:{
    get_weather(){
      // 获取指定城市的天气信息
      // http.get(`http://wthrcdn.etouch.cn/weather_mini?city=${this.city}`).then(response=>{
      http.get(`http://wthrcdn.etouch.cn/weather_mini`,{
        params:{
          city: this.city,
        },
        headers:{
          // 自定义请求头
        }
      }).then(response=>{
        if(response.status === 200){
          if(response?.data?.data?.forecast){
            this.weathers = response.data.data.forecast;
          }
        }
      }).catch(error=>{
        // ajax请求或者响应处理过程中，出现异常，则会自动调用这里
        console.log(error) // 打印错误信息【客户端报错的情况下】
        console.log(error.response) // 获取来自服务端的错误信息
      })
    },
    user_register(){
      // 使用post方法提交数据，也可以使用put/patch来更新数据，参数与返回值于post一模一样
      // 参数1：url地址，数据提交的服务端地址
      // 参数2：data，请求体数据，json对象
      // 参数3：config，请求配置，params表示查询字符串，headers表示请求头
      // 返回值是一个Promise对象.可以通过then方法接收返回的结果，也可以通过catch方法接收错误
      http.post("http://127.0.0.1:8000/xiaoming", {
        name: 'xiaoming',
        sex: true,
        age: 17,
        classmate: "301",
        description: "巴拉巴拉...."
      },{
        // params:{},
        // headers:{},
      }).then(response=>{
        console.log(response.data)
      }).catch(error=>{
        console.log(error)
      })
    }
  },
  components:{
    Menu
  }
}
</script>

<style scoped>
    .weather table{
        width: 800px;
        border-collapse: collapse;
    }
    .weather table td{
        width: 120px;
        height: 40px;
        line-height: 40px;
        border: 1px solid red;
        text-align: center;
    }
</style>
```



## 请求和响应拦截器

```
https://github.com/axios/axios#interceptors
```

官方举例代码：

```javascript
import axios from "axios"; // 要导入安装的包，则直接填写包名即可。不需要使用路径


// 实例化
const httptool = axios.create({
    // baseURL: 'http://wthrcdn.etouch.cn/', // 请求的公共路径,一般填写服务端的api地址
    timeout: 1000 * 2.5,                  // 最大请求超时时间，请求超过这个时间则报错
    headers: {                            // 预定义请求头，一般工作中这里填写隐藏了客户端身份的字段
        'X-Custom-Header': 'foobar'
    }
})

// 新增 一个 请求 拦截器
httptool.interceptors.request.use(function (config) {
    // 编写 一些代码 在 请求 发送之前
    console.log("loading...")
    return config;
  }, function (error) {
    // 编写 一些在 在 请求 发生错误时
    alert("报错！报错！");
    return Promise.reject(error);
  });

// 新增 一个 响应 拦截器
httptool.interceptors.response.use(function (response) {
    // 编写 一些在 服务端响应数据成功时
    console.log("关闭loding...")
    return response;
}, function (error) {
    // 编写 一些在 服务端响应错误时
    console.log("服务端告诉我们，出错了。")
    return Promise.reject(error);
});

export default httptool;
```

