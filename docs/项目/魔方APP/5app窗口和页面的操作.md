## 在APP进行窗口和页面操作

APP中针对用户看待的界面，有份窗口和页面的说法。在APICloud中提供了2种类型,3种方式给开发者打开或者新建窗口/页面.

### 1. window 窗口

window是APICloud提供的最顶级的界面单位.一个APP至少会存在一个以上的window窗口,在用户打开APP应用,APP应用在初始化的时候默认就会创建了一个name=root 的顶级window窗口显示当前APP配置的默认首页.（默认首页就是config.xml中content配置项指定的）

如果APP中所有的窗口都关闭了，则APP就被系统退出了。注意：手机的物理返回键，控制的就是退出APP的当前窗口。

#### 新建window窗口

注意: 如果之前已经存在同名的窗口,则APP不会再次创建新窗口,而是把对应名称的窗口进行置顶显示给用户看.

代码:

```javascript
api.openWin({
    name: 'page1',        // 自定义窗口名称
    bounces: false,       // 窗口是否上下拉动
    reload: false,        // 如果页面已经在之前被打开了,是否要重新加载当前窗口中的页面
    url: './page1.html',  // 窗口创建时展示的html页面的本地路径[相对于当前代码所在文件的路径]
    animation:{           // 打开新建窗口时的过渡动画效果
    	type:"none",                //动画类型（详见动画类型常量）
    	subType:"from_right",       //动画子类型（详见动画子类型常量）
    	duration:300                //动画过渡时间，默认300毫秒
    },
    pageParam: {          // 传递给下一个窗口使用的参数.将来可以在新窗口中通过 api.pageParam.name 获取
        name: 'test'      // name只是举例, 将来可以传递更多自定义数据的.
    }
});
```

#### 关闭指定名称的窗口

如果当前APP中只有剩下一个顶级窗口root,则无法通过当前方法关闭! 也有部分手机直接退出APP了

```javascript
//关闭当前window，使用默认动画
api.closeWin();

//关闭指定名称的window，若待关闭的window不在最上面，则无动画
api.closeWin({
    name: 'page1'
});
```

接下来我们可以把打开和关闭窗口的代码封装到主程脚本main.js中.

```javascript
class Game{
	constructor(bg_music){
		// 构造函数
		this.init();
    	this.play_music(bg_music);
	}
	init(){
		// 初始化
		console.log("系统初始化");
    	this.rem();
	}
	print(data, type=true){
		// 打印数据
		if(type){
			api.alert({msg:JSON.stringify(data)});
		}
		console.log("打印开始: >>>>> ");
		console.log(JSON.stringify(data));
		console.log("打印结束: <<<<< ");
	}
	rem(){
		if(window.innerWidth<1200){
				this.UIWidth = document.documentElement.clientWidth;
		this.UIHeight = document.documentElement.clientHeight;
		document.documentElement.style.fontSize = (0.01*this.UIWidth*3)+"px";
				document.querySelector("#app").style.height=this.UIHeight+"px"
		}
		window.onresize = ()=>{
		if(window.innerWidth<1200){
			this.UIWidth = document.documentElement.clientWidth;
			this.UIHeight = document.documentElement.clientHeight;
			document.documentElement.style.fontSize = (0.01*this.UIWidth*3)+"px";
		}
		}
	}
	stop_music(){
		this.print("停止")
		document.body.removeChild(this.audio);
	}
  	play_music(src){
		this.audio = document.createElement("audio");
		this.source = document.createElement("source");
		this.source.type = "audio/mp3";
		this.audio.autoplay = "autoplay";
		this.source.src=src;
		this.audio.appendChild(this.source);
		document.body.appendChild(this.audio);
		var t = setInterval(()=>{
		if(this.audio.readyState > 0){
			if(this.audio.ended){
			clearInterval(t);
			document.body.removeChild(this.audio);
			}
		}
		},100);
	}

	goWin(name,url,pageParam){
		// 打开窗口
		api.openWin({
		    name: name,            // 自定义窗口名称
		    bounces: false,        // 窗口是否上下拉动
		    reload: true,          // 如果页面已经在之前被打开了,是否要重新加载当前窗口中的页面
		    url: url,              // 窗口创建时展示的html页面的本地路径[相对于当前代码所在文件的路径]
		    animation:{            // 打开新建窗口时的过渡动画效果
		    	type: "push",                //动画类型（详见动画类型常量）
		    	subType: "from_right",       //动画子类型（详见动画子类型常量）
		    	duration:300                //动画过渡时间，默认300毫秒
		    },
		    pageParam: pageParam   // 传递给下一个窗口使用的参数.将来可以在新窗口中通过 api.pageParam.name 获取
		});
	}
	outWin(name){
		// 关闭窗口
		api.closeWin({
			name: name
		});
	}


}

```

这里，我们就可以利用上面的关闭窗口，在用户按了手机设备的返回键时，判断用户是否要退出当前APP，调用quit方法。

main.js，代码：

```javascript
class Game{
	// 主程序
	constructor(bg_music){
		// 构造函数，类似于python的 __init__()
		this.init();
    	this.play_music(bg_music);
		this.quit();
	}
	init(){
		// 初始化
		console.log("系统初始化");
    	this.rem();
	}
	quit(){
		// 监听设备中的keyback按钮是否被按了
		api.addEventListener({
			name: 'keyback'
		}, (ret, err) => {
			// 弹出一个确认框
			var windows = api.windows()
			if(windows.length>1){
				// 如果当前APP打开了多个窗口，则直接关闭窗口
				this.outWin();
			}else{
				// 如果当前APP只打开了一个窗口，则弹窗确认用户是否退出当前APP
				api.confirm({
					title: '系统提示',
					msg: '您确认退出魔方APP吗？',
					buttons: ['退出', '取消']
				}, (ret, err) => {
					var index = ret.buttonIndex;
					if(index === 1) {
						this.outWin();
					}
				});
			}
		});
	}
	print(data,show=false){
		// 打印数据
		if(show){
			// 以弹窗的方式打印数据
			api.alert({"msg": JSON.stringify(data)});
		}else{
			// 以终端的方式打印数据
			console.log(JSON.stringify(data));
		}
	}
	rem(){
		// 页面自适配方法
		if(window.innerWidth<1200){
			this.UIWidth = document.documentElement.clientWidth;
			this.UIHeight = document.documentElement.clientHeight;
			document.documentElement.style.fontSize = (0.01*this.UIWidth*3)+"px";
			document.querySelector("#app").style.height=this.UIHeight+"px"
		}
		window.onresize = ()=>{
			if(window.innerWidth<1200){
				this.UIWidth = document.documentElement.clientWidth;
				this.UIHeight = document.documentElement.clientHeight;
				document.documentElement.style.fontSize = (0.01*this.UIWidth*3)+"px";
			}
		}
	}
	stop_music(){
		this.print("停止音乐")
		try {
			document.body.removeChild(this.audio);
		} catch (error) {
			this.print(error)
		}
	}
	play_music(src){
		this.print("播放音乐")
		try {
			this.audio = document.createElement("audio");
			this.source = document.createElement("source");
			this.source.type = "audio/mp3";
			this.audio.autoplay = "autoplay";
			this.source.src=src;
			this.audio.appendChild(this.source);
			document.body.appendChild(this.audio);
			var t = setInterval(()=>{
				if(this.audio.readyState > 0){
					if(this.audio.ended){
						clearInterval(t);
						document.body.removeChild(this.audio);
					}
				}
			},100);
		} catch (error) {
			this.print(error);
		}
	}

	goWin(name,url,pageParam){
		// 打开窗口
		api.openWin({
		    name: name,            // 自定义窗口名称
		    bounces: false,        // 窗口是否上下拉动
		    reload: true,          // 如果页面已经在之前被打开了,是否要重新加载当前窗口中的页面
		    url: url,              // 窗口创建时展示的html页面的本地路径[相对于当前代码所在文件的路径]
		    animation:{            // 打开新建窗口时的过渡动画效果
		    	type: "push",                //动画类型（详见动画类型常量）
		    	subType: "from_right",       //动画子类型（详见动画子类型常量）
		    	duration:300                //动画过渡时间，默认300毫秒
		    },
		    pageParam: pageParam   // 传递给下一个窗口使用的参数.将来可以在新窗口中通过 api.pageParam.name 获取
		});
	}
	outWin(name){
		// 关闭窗口
		api.closeWin({
			name: name
		});
	}
}

```



### 2. frame 帧页面

```
如果APP中所有的页面全部窗口进行展开,则APP需要耗费大量的内存来维护这个窗口列表,从而导致, 用户操作APP时,APP响应缓慢甚至卡顿的现象.所以APP中除了通过新建窗口的方式展开页面以外, 还提供了帧的方式来展开页面.
帧,代表的就是一个窗口下开打的某个页面记录.所谓的帧就有点类似于浏览器中窗口通过地址栏新建的一个页面一样.
使用的时候注意: 
    1. APP每一个window窗口都可以打开1到多个帧.新建窗口的时候,系统会默认顺便创建第一帧页面出来.
    2. 每一帧代表的都是一个html页面, 如果一个windows窗口什么页面都没有，则系统会提示错误，并且窗口默认是透明的。
    3. 默认情况下, APP的window的窗口会自动默认满屏展示.而帧可以设置矩形的宽高.如果顶层的帧页面没有满屏显示,则用户可以看到当前这一帧下的其他帧的内容.
```

#### 新建帧页面

```javascript
api.openFrame({
    name: 'page2',        // 帧页面的名称
    url: './page2.html',  // 帧页面打开的url地址
    data: '',             // 可选参数,如果填写了data,则不要使用url, data表示页面数据,可以是html代码
    bounces:false,        // 页面是否可以下拉拖动
    reload: true,         // 帧页面如果已经存在,是否重新刷新加载
    useWKWebView:true,
    historyGestureEnabled:true,
    animation:{
        type:"push",             //动画类型（详见动画类型常量）
    	subType:"from_right",    //动画子类型（详见动画子类型常量）
    	duration:300             //动画过渡时间，默认300毫秒
    },
    rect: {               // 当前帧的宽高范围
        // 方式1,设置矩形大小宽高
        x: 0,             // 左上角x轴坐标
        y: 0,             // 左上角y轴坐标
        w: 'auto',        // 当前帧页面的宽度, auto表示满屏
        h: 'auto'         // 当前帧页面的高度, auto表示满屏
		// 方式2,设置矩形大小宽高
        marginLeft:,    //相对父页面左外边距的距离，数字类型
        marginTop:,     //相对父页面上外边距的距离，数字类型
        marginBottom:,  //相对父页面下外边距的距离，数字类型
        marginRight:    //相对父页面右外边距的距离，数字类型
    },
    pageParam: {          // 要传递新建帧页面的参数,在新页面可通过 api.pageParam.name 获取
        name: 'test'      // name只是举例, 可以传递任意自定义参数
    }
});
```

#### 关闭帧页面

```javascript
// 关闭当前 frame页面
api.closeFrame();

// 关闭指定名称的frame页面
api.closeFrame({
    name: 'page2'
});
```

接下来我们可以把打开和关闭frame页面的代码封装到主程脚本main.js中.

```javascript
class Game{
	// 主程序
	constructor(bg_music){
		// 构造函数，类似于python的 __init__()
		this.init();
    	this.play_music(bg_music);
		this.quit();
	}
	init(){
		// 初始化
		console.log("系统初始化");
    	this.rem();
	}
	quit(){
		// 监听设备中的keyback按钮是否被按了
		api.addEventListener({
			name: 'keyback'
		}, (ret, err) => {
			// 弹出一个确认框
			var windows = api.windows()
			if(windows.length>1){
				// 如果当前APP打开了多个窗口，则直接关闭窗口
				this.outWin();
			}else{
				// 如果当前APP只打开了一个窗口，则弹窗确认用户是否退出当前APP
				api.confirm({
					title: '系统提示',
					msg: '您确认退出魔方APP吗？',
					buttons: ['退出', '取消']
				}, (ret, err) => {
					var index = ret.buttonIndex;
					if(index === 1) {
						this.outWin();
					}
				});
			}
		});
	}
	print(data,show=false){
		// 打印数据
		if(show){
			// 以弹窗的方式打印数据
			api.alert({"msg": JSON.stringify(data)});
		}else{
			// 以终端的方式打印数据
			console.log(JSON.stringify(data));
		}
	}
	rem(){
		// 页面自适配方法
		if(window.innerWidth<1200){
			this.UIWidth = document.documentElement.clientWidth;
			this.UIHeight = document.documentElement.clientHeight;
			document.documentElement.style.fontSize = (0.01*this.UIWidth*3)+"px";
			document.querySelector("#app").style.height=this.UIHeight+"px"
		}
		window.onresize = ()=>{
			if(window.innerWidth<1200){
				this.UIWidth = document.documentElement.clientWidth;
				this.UIHeight = document.documentElement.clientHeight;
				document.documentElement.style.fontSize = (0.01*this.UIWidth*3)+"px";
			}
		}
	}
	stop_music(){
		this.print("停止音乐")
		try {
			document.body.removeChild(this.audio);
		} catch (error) {
			this.print(error)
		}
	}
	play_music(src){
		this.print("播放音乐")
		try {
			this.audio = document.createElement("audio");
			this.source = document.createElement("source");
			this.source.type = "audio/mp3";
			this.audio.autoplay = "autoplay";
			this.source.src=src;
			this.audio.appendChild(this.source);
			document.body.appendChild(this.audio);
			var t = setInterval(()=>{
				if(this.audio.readyState > 0){
					if(this.audio.ended){
						clearInterval(t);
						document.body.removeChild(this.audio);
					}
				}
			},100);
		} catch (error) {
			this.print(error);
		}
	}

	goWin(name,url,pageParam){
		// 打开窗口
		api.openWin({
		    name: name,            // 自定义窗口名称
		    bounces: false,        // 窗口是否上下拉动
		    reload: true,          // 如果页面已经在之前被打开了,是否要重新加载当前窗口中的页面
		    url: url,              // 窗口创建时展示的html页面的本地路径[相对于当前代码所在文件的路径]
		    animation:{            // 打开新建窗口时的过渡动画效果
		    	type: "push",                //动画类型（详见动画类型常量）
		    	subType: "from_right",       //动画子类型（详见动画子类型常量）
		    	duration:300                //动画过渡时间，默认300毫秒
		    },
		    pageParam: pageParam   // 传递给下一个窗口使用的参数.将来可以在新窗口中通过 api.pageParam.name 获取
		});
	}
	outWin(name){
		// 关闭窗口
		api.closeWin({
			name: name
		});
	}

	goFrame(name,url, pageParam){
		// 打开帧页面
		api.openFrame({
				name: name,
				url: url,
				rect: {
						x: 0,
						y: 0,
						w: 'auto',
						h: 'auto'
				},
				useWKWebView: true,
				historyGestureEnabled: true,
				bounces: false,
				animation:{
							type:"push",
							subType: "from_right",
							duration:300
				},
				pageParam: pageParam
		});
	}

	outFrame(name){
		// 关闭帧页面
		api.closeFrame({
		    name: name,
		});
	}

}

```

在index.html页面中,当用户点击左上角第一个抽奖按钮时，暂时先设置让系统跳转到登陆页面。

index.html，代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>首页</title>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width,minimum-scale=1.0,maximum-scale=1.0,user-scalable=no">
    <meta name="format-detection" content="telephone=no,email=no,date=no,address=no">
    <link rel="stylesheet" href="../css/main.css">
	<script src="../script/vue.js"></script>
	<script src="../script/main.js"></script>
</head>
<body>
  <div class="app" id="app">
    <img class="music" :class="music_play?'music2':''" @click="music_play=!music_play" src="../image/player.png">
    <div class="bg">
      <img src="../image/bg0.jpg">
    </div>
    <ul>
      <li><img class="module1" src="../image/image1.png"></li> <!-- 果园 -->
      <li><img class="module2" src="../image/image2.png"></li> <!-- 会员 -->
      <li><img class="module3" src="../image/image3.png"></li> <!-- 娱乐 -->
      <li><img class="module4" @click="open_login" src="../image/image4.png"></li> <!-- 签到 -->
    </ul>
  </div>
  <script>
	apiready = function(){
		Vue.prototype.game = new Game("../mp3/bg1.mp3");  // vue3.0里面需要修改这句代码
		new Vue({
			el:"#app",
			data(){
				return {
                    music_play: true,  // 默认播放背景音乐
                }
			},
            watch:{
                music_play(){
                    if(this.music_play){
                        this.game.play_music("../mp3/bg1.mp3");
                    }else{
                        this.game.stop_music();
                    }
                }
            },
            methods:{
                open_login(){
                    this.game.goWin("login","./login.html");
                }
            }
		})
	}
	</script>
</body>
</html>
```

在login.html页面中, 当用户点击`立即注册`,进行新窗口.

```html
<!DOCTYPE html>
<html>
<head>
	<title>登录</title>
	<meta name="viewport" content="width=device-width,minimum-scale=1.0,maximum-scale=1.0,user-scalable=no">
	<meta charset="utf-8">
	<link rel="stylesheet" href="../css/main.css">
	<script src="../script/vue.js"></script>
	<script src="../script/main.js"></script>
</head>
<body>
	<div class="app" id="app">
        <img class="music" :class="music_play?'music2':''" @click="music_play=!music_play" src="../image/player.png">
        <div class="bg">
            <img src="../image/bg0.jpg">
        </div>
        <div class="form">
            <div class="form-title">
                <img src="../image/login.png">
                <img class="back" @click="open_index" src="../image/back.png">
            </div>
            <div class="form-data">
                <div class="form-data-bg">
                    <img src="../image/bg1.png">
                </div>
                <div class="form-item">
                    <label class="text">手机</label>
                    <input type="text" name="mobile" placeholder="请输入手机号">
                </div>
                <div class="form-item">
                    <label class="text">密码</label>
                    <input type="password" name="password" placeholder="请输入密码">
                </div>
                <div class="form-item">
                    <input type="checkbox" class="agree remember" name="agree" checked>
                    <label><span class="agree_text ">记住密码，下次免登录</span></label>
                </div>
                <div class="form-item">
                    <img class="commit" @click="game.play_music('../mp3/btn1.mp3')" src="../image/commit.png">
                </div>
                <div class="form-item">
                    <p class="toreg" @click="open_register">立即注册</p>
                    <p class="tofind">忘记密码</p>
                </div>
            </div>
        </div>
	</div>
	<script>
	apiready = function(){
		Vue.prototype.game = new Game("../mp3/bg1.mp3");
		new Vue({
			el:"#app",
			data(){
				return {
                    music_play:true,
				}
			},
			watch:{
                music_play(){
                    if(this.music_play){
                        this.game.play_music("../mp3/bg1.mp3");
                    }else{
                        this.game.stop_music();
                    }
                }
            },
            methods:{
                open_register(){
                    this.game.goFrame("register","./register.html");
                },
                open_index(){
                    this.game.outWin();
                }
            }
		})
	}
	</script>
</body>
</html>
```



`html/register.html`,代码:

```html
<!DOCTYPE html>
<html>
<head>
	<title>注册</title>
	<meta name="viewport" content="width=device-width,minimum-scale=1.0,maximum-scale=1.0,user-scalable=no">
	<meta charset="utf-8">
	<link rel="stylesheet" href="../css/main.css">
	<script src="../script/vue.js"></script>
	<script src="../script/main.js"></script>
</head>
<body>
	<div class="app" id="app">
        <img class="music" :class="music_play?'music2':''" @click="music_play=!music_play" src="../image/player.png">
        <div class="bg">
            <img src="../image/bg0.jpg">
        </div>
		<div class="form">
			<div class="form-title">
				<img src="../image/register.png">
				<img class="back" @click="open_login" src="../image/back.png">
			</div>
			<div class="form-data">
				<div class="form-data-bg">
					<img src="../image/bg1.png">
				</div>
				<div class="form-item">
					<label class="text">手机</label>
					<input type="text" name="mobile" placeholder="请输入手机号">
				</div>
				<div class="form-item">
					<label class="text">验证码</label>
					<input type="text" class="code" name="code" placeholder="请输入验证码">
					<img class="refresh" src="../image/refresh.png">
				</div>
				<div class="form-item">
					<label class="text">密码</label>
					<input type="password" name="password" placeholder="请输入密码">
				</div>
				<div class="form-item">
					<label class="text">确认密码</label>
					<input type="password" name="password2" placeholder="请再次输入密码">
				</div>
				<div class="form-item">
					<input type="checkbox" class="agree" name="agree" checked>
					<label><span class="agree_text">同意磨方《用户协议》和《隐私协议》</span></label>
				</div>
				<div class="form-item">
					<img class="commit" @click="game.play_music('../mp3/btn1.mp3')" src="../image/commit.png"/>
				</div>
			</div>
		</div>
	</div>
	<script>
    apiready = function(){
        Vue.prototype.game = new Game("../mp3/bg1.mp3");
        new Vue({
            el:"#app",
            data(){
                return {
                    music_play:true,
                }
            },
            watch:{
                music_play(){
                    if(this.music_play){
                        this.game.play_music("../mp3/bg1.mp3");
                    }else{
                        this.game.stop_music();
                    }
                }
            },
            methods:{
                open_login(){
                    this.game.outFrame();
                }
            }
		})
	}
	</script>
</body>
</html>
```

提交版本

```bash
git add .
git commit -m "fix: add function of app's window and frame page"
git push
```



### 3. framegroup  帧页面组

**以下代码，我们当前项目用不上。所以不需要跟着在html页面中敲**

#### 新建帧页面组

```javascript
api.openFrameGroup({
    name: 'group1',   // 组名
    rect: {           // 帧页面组的显示矩形范围
        // 方式1:
        x:,             //左上角x坐标，数字类型
        y:,             //左上角y坐标，数字类型
        w:,             //宽度，若传'auto'，页面从x位置开始自动充满父页面宽度，数字或固定值'auto'
        h:,             //高度，若传'auto'，页面从y位置开始自动充满父页面高度，数字或固定值'auto'
        
        // 方式2: 
        marginLeft:,    //相对父页面左外边距的距离，数字类型
    	marginTop:,     //相对父页面上外边距的距离，数字类型
    	marginBottom:,  //相对父页面下外边距的距离，数字类型
    	marginRight:    //相对父页面右外边距的距离，数字类型
    },
    frames: [{
        name:'',         //frame名字，字符串类型，不能为空字符串
        url:'',          // 页面地址
		useWKWebView:true,
        historyGestureEnabled:false,  //（可选项）是否可以通过手势来进行历史记录前进后退。
        pageParam:{},    // 页面参数
        bounces:true,    // 是否能下拉拖动
    }, {
        name:'',         //frame名字，字符串类型，不能为空字符串
        url:'',          // 页面地址
		useWKWebView:true,
        historyGestureEnabled:false,  //（可选项）是否可以通过手势来进行历史记录前进后退。
        pageParam:{},    // 页面参数
        bounces:true,    // 是否能下拉拖动
    },{
        ...
      
    },...
    ]
}, (ret, err) => {
    // 当前帧页面发生页面显示变化时,当前帧的索引.
    var index = ret.index;
});
```

#### 关闭帧页面组

```javascript
api.closeFrameGroup({
    name: 'group1' // 组名
});
```

#### 切换当前帧页面组显示的帧页面

```javascript
api.setFrameGroupIndex({
    name: 'group1', // 组名
    index: 2        // 索引,从0开始
});
```

