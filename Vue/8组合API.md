# 组合API

Vue3.0在原来2.0版本基础上进行升级的时候，进行语法上的兼容。

我们在vue2.0中编写的所有的操作api接口在3.0版本中是完全可以使用的。

但是2.0版本的操作api代码，一般写在data，methods，watch等等选项中的，我们称之为“选项API（Option API）”，

在3.0版本中vue官方在不删除选项API的基础上，又新增了新的写法的操作API接口，叫“组合API（Group API）”。

组件API是另一种编写vue代码的写法。

![image-20210701102058031](8%E7%BB%84%E5%90%88API.assets/image-20210701102058031.png)

## 选项API

所谓的选项API，是Vue2.0组件中常用的数据声明和操作数据的方法，按不同的选项把代码进行分开存放。

data，methods，created，mounted，computed，watch....等等这些地方就是选项。

Option.vue，代码：

```vue
<template>
  <p>选项API写法</p>
</template>

<script>
export default {
  name: "Options",
  data(){
    return {
      students:[],
      courses:[],
    }
  },
  methods:{
    get_student(){},
    get_course(){},
  },
  created(){
    this.get_student();
    this.get_course();
  },
  mounted(){

  },
  watch:{

  },
  computed:{

  }
}
</script>

<style scoped>

</style>
```

## 组合API

`src/views/Group.vue`，代码：

```vue
<template>
  <p>组合API的基本使用</p>
  <p>显示对象格式的响应数据</p>
  <p>{{user}}</p>
  <p @click="change_user_info">点击修改{{user.name}}的数据</p>
  <input type="text" v-model="user.age">
  <p>显示普通数据</p>
  <p>{{data}}</p>
  <input type="text" v-model="data.age">
  <hr>
  <p>显示非对象的基本类型数据</p>
  <p @click="msg_add">{{msg}}</p>
  <input type="text" v-model="msg">
  <p>{{new_msg}}</p>
</template>

<script>
import { reactive,ref, onMounted, watch, computed } from 'vue'
export default {
  name: "Group",
  // data(){
  //    return {
  //      num: 10,
  //    }
  // },
  // setup 会自动执行，并且会在befroreCreate钩子方法执行之前就自动执行了
  setup(){
    console.log(this); // undefined，setup执行的时候，连this都没有定义，所以，我们不可能在setup里面调用之前的选项API方法了。
    console.log("setup自动执行了");

    // 显示一个变量(对象)到页面中
    let data = {name:"xiaoming",age:17,sex:true,}
    // 一旦把数据转换成响应式数据，则vm对象就会自动的维护和保持数据的全局一致性, 也就是HTML页面中修改了数据，vm对象中的setup里面的数据也被发生改动
    let user = reactive(data); // // 把数据通过reactive函数进行处理的目的就是为了转换数据为响应式数据

    let change_user_info = ()=>{
      user.age = 30;
    }

    // 显示一个变量(基本类型：布尔值，数值，字符串)到页面中
    let message = 100
    let msg = ref(message);  // ref的内部本质就是右边代码的简写 let msg = reactive({value: message});
    // 如果希望数据提供给视图模板HTML中使用return通过对象进行返回

    console.log(msg.value);  // setup代码中如果要调用经过ref处理过的数据，需要value属性来操作
    msg.value=2000;          // 修改也是通过value属性来修改原来的数据

    // 操作数据的方法
    let msg_add = ()=>{
        msg.value++;
    }

    // 钩子方法的调用[组合API中不再提供created和beforeCreated，原来这2个钩子方法中代码直接写在setup中即可]
    onMounted(() => {
      console.log('Component is mounted!')
    });

    // 监听数据
    // watch("响应数据的变量名", (new_value, old_value)=>{
    //
    // });
    watch(msg, ()=>{
      if(msg.value == "python"){
        console.log("message的值不能是python了");
        msg.value = 2000;
      }
    });

    // 计算属性
    let new_msg = computed(()=>{
      return msg.value * 4;
    });

    return {
      user, // user:user的简写
      data, // 非响应数据
      msg,
      msg_add, // 对外提供操作数据的方法
      change_user_info,
      new_msg,
    }
  },
  // beforeCreate() {
  //   console.log("钩子beforeCreate执行了");
  // }
}
</script>

<style scoped>

</style>
```



组合API的代码：

```vue
<template>
  <h1>组合API</h1>
  <p>显示基本数据类型的信息</p>
  <p>{{age}}</p>
  <button @click="ajax_get_age(18)">点击获取年龄</button>
  <button @click="change_age">年龄自增</button>
  <p>{{new_age}}</p>
  <p>显示复杂数据类型的信息</p>
  <p>{{state}}</p>
  <p>{{state.user_info}}</p>
  <button @click="get_user_info">点击获取用户信息</button>
  <button @click="state.user_info.age++">年龄自增</button>
  <p>调用外部封装的数据和操作数据的方法</p>
  <p>{{student}}</p>
  <p>{{student.info}}</p>
  <button @click="student.get_info">点击获取学生数据</button>
  <button @click="student.change_info">自增数据</button>
</template>

<script>
import {ref,reactive,watch,computed,onMounted} from "vue";
import student from "../api/student";
export default {
  name: "Group",
  beforeCreate() {
    console.log("beforeCreate执行了")
  },
  created(){
    console.log("created执行了")
  },
  // setup 会自动执行，并且会在befroreCreate钩子方法执行之前就自动执行了
  setup(){
    // console.log("setup自动执行了");
    // console.log(`this=${this}`) // undefined，setup执行的时候，连this都没有定义，所以，我们不可能在setup里面像之前那样调用的选项API的方法了，需当然，如果还要使用原来的，props，emit，watch之类的选项，在组合API有新的写法。

    // 申明一个基本数据类型的变量[数值、布尔值、字符串、]
    let age = ref(0) // ref的底层运行的就是reactive({value: 0})

    // 操作基本类型的数据的方法
    let ajax_get_age = (num)=>{  // 模拟ajax从服务端请求得到数据
      age.value = num;
    }
    let change_age = ()=>{
      age.value++
    }

    // 侦听属性. 监听属性
    // 监听数据的变化,
    // 参数1，必须是经过ref或者reactive包装过的数据[响应式数据]
    //       如果监听的是reactive包装过的数据,必须写成回调函数格式
    // 参数2，数据发生改变时，自动执行的回调函数，该回调函数默认提供2个参数，分别是监听数据的修改后和修改前的值，
    watch(
        age, //ref包装过的数据
        ()=>{
          if(age.value > 30){
            console.log("别点了！再点就老了！！！")
          }
        }
    )

    // 计算属性
    let new_age = computed(()=>{
      return age.value * 4;
    })

    // 钩子方法
    onMounted(()=>{
      console.log('Component is mounted!')
    })

    // 声明一个复杂数据类型的变量[对象、数组、]
    let state = reactive({
      user_info: {},
    })
    console.log(age,state)
    let get_user_info = ()=>{
      state.user_info = {name:"xiaoming",age:13}
    }

    watch(
        ()=>state.user_info.age,
        ()=>{
          console.log("监听函数执行了", state.user_info.age)
        }
    )

    // 如果要把setup 中 声明的变量和操作变量的方法提供给template中进行调用，则必须写在return的后面。
    return {age, ajax_get_age, change_age, state, get_user_info,student,new_age}
  }
}
</script>

<style scoped>

</style>
```

把数据以及操作数据的方法全部写在一个单独的文件中，需要这个数据了直接导入即可。`src/api/student.js`，代码：

```javascript
import {reactive} from "vue";
export default reactive({
    info:{

    },
    get_info(){
        this.info = {
            name: "xiaoming",
            age: 19
        }
    },
    change_info(){
        this.info.age++
    }
})
```





基于script的setup属性，对组合API进行简写。

```vue
<template>
  <h1>组合API</h1>
  <p>显示基本数据类型的信息</p>
  <p>{{age}}</p>
  <button @click="ajax_get_age(18)">点击获取年龄</button>
  <button @click="change_age">年龄自增</button>
  <p>{{new_age}}</p>
  <p>显示复杂数据类型的信息</p>
  <p>{{state}}</p>
  <p>{{state.user_info}}</p>
  <button @click="get_user_info">点击获取用户信息</button>
  <button @click="state.user_info.age++">年龄自增</button>
  <p>调用外部封装的数据和操作数据的方法</p>
  <p>{{student}}</p>
  <p>{{student.info}}</p>
  <button @click="student.get_info">点击获取学生数据</button>
  <button @click="student.change_info">自增数据</button>
</template>

<script setup>
import {ref,reactive,watch,computed,onMounted} from "vue";
import student from "../api/student";
  // console.log("setup自动执行了");
  // console.log(`this=${this}`) // undefined，setup执行的时候，连this都没有定义，所以，我们不可能在setup里面像之前那样调用的选项API的方法了，需当然，如果还要使用原来的，props，emit，watch之类的选项，在组合API有新的写法。
  // 申明一个基本数据类型的变量[数值、布尔值、字符串、]
  let age = ref(0) // ref的底层运行的就是reactive({value: 0})

  // 操作基本类型的数据的方法
  let ajax_get_age = (num)=>{  // 模拟ajax从服务端请求得到数据
    age.value = num;
  }
  let change_age = ()=>{
    age.value++
  }

  // 侦听属性. 监听属性
  // 监听数据的变化,
  // 参数1，必须是经过ref或者reactive包装过的数据[响应式数据]
  //       如果监听的是reactive包装过的数据,必须写成回调函数格式
  // 参数2，数据发生改变时，自动执行的回调函数，该回调函数默认提供2个参数，分别是监听数据的修改后和修改前的值，
  watch(
      age, //ref包装过的数据
      ()=>{
        if(age.value > 30){
          console.log("别点了！再点就老了！！！")
        }
      }
  )

  // 计算属性
  let new_age = computed(()=>{
    return age.value * 4;
  })

  // 钩子方法
  onMounted(()=>{
    console.log('Component is mounted!')
  })

  // 声明一个复杂数据类型的变量[对象、数组、]
  let state = reactive({
    user_info: {},
  })
  console.log(age,state)
  let get_user_info = ()=>{
    state.user_info = {name:"xiaoming",age:13}
  }

  watch(
      ()=>state.user_info.age,
      ()=>{
        console.log("监听函数执行了", state.user_info.age)
      }
  )
</script>

<style scoped>

</style>
```


