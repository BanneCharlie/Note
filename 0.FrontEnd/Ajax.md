# Ajax简介

作用实现页面的局部更新;

浏览器

创建XMLHttpRequest对象,发送HttpRequest http请求

服务器

处理XMLHttpRequest创建响应式并将数据(response数据)返回给浏览器

浏览器

使用JS处理被返回的数据,更新页面内容.

# HTTP协议

第一点 http协议(超文本传输协议) 规定了浏览器和服务器之间的通信规则

---

第二点 请求(request)--响应(response) 模式

1. 请求报文(request)

   - 请求行   (GET/POST)-->请求方式  请求URL  HTTP协议/版本号

   - 请求头    格式为  Key : value  形式  每个键值对都是一个请求头,一次请求可以有多个请求头

   - 空行

   - 请求体    GET(不具有请求体)  / POST(可以具有请求体)

     ----

   注 : 

   - 请求体 POST请求传递参数的放在请求体上,GET请求传递参数通过URL地址栏来传递参数在地址栏上,POST请求也是可以通过url来传递参数;

   - 所有GET请求不具备请求体,而且GET请求可以在浏览器地址栏中看到请求参数;

   - GET请求传递参数受到限制,最多允许255个字符;所以大数据传递使用POST请求,因为POST方法参数是在报文主体中的.

   - GET请求不具备保密性,所以一般需要保密的数据都是通过POST请求传递的.

     ![image-20220402104012695](https://notebook-img.oss-cn-shanghai.aliyuncs.com/git-img/202204021058418.png)

     请求头信息

     ![image-20220402104509368](https://notebook-img.oss-cn-shanghai.aliyuncs.com/git-img/202204021059509.png)

2. 响应报文(response)

   - 响应行  HTTP/版本号(响应版本)  200(响应状态码)  OK(响应字符串)

   - 响应头  格式为  Key : value  形式

   - 空行

   - 响应体  页面显示的数据

     ---

注 : 

1. 状态码的分类 

   - 100~199（信息性状态码）     接收的请求正在处理

   - 200~299（成功状态码）       请求正常处理完毕

   - 300~399（重定向状态码）     被请求的资源已移动到新位置,需要进行附加操作以完成请求

   - 400~499（客户端错误状态码）  服务器无法处理请求

   - 500~599（服务器端错误状态码）服务器处理请求出错

     ![image-20220402104120102](https://notebook-img.oss-cn-shanghai.aliyuncs.com/git-img/202204021059720.png)

     响应头信息

     ![](https://notebook-img.oss-cn-shanghai.aliyuncs.com/git-img/202204021059935.png)

---

第三点 无保存模式

简介 : HTTP协议自身不具备保存之前发送过的请求或者响应的功能.

# Ajax方式发送请求

## 1.Ajax发送GET请求(传递参数)

```js
浏览器  --- > 服务器
//html页面中发送AJAX请求  
 <script>
            //获取元素
            const btn = document.querySelector('button');
            const box = document.querySelector('.box');
            btn.onclick = function(){
                // console.log("test");
                
                //1.创建XMLHttpRequest对象
                const xhr = new XMLHttpRequest();
                
                /*2.初识化  设置请求方法 和  请求对象的url  
                
                可以在url路径里面编写参数,将参数传递给服务器,这是url传参
                  * (GET请求传递参数通过url地址栏 参数直接放在地址栏上)
                  * (POST请求传递参数也可以通过 url 地址栏传递,POST请求还有其他的传递参数的方式)
                */
                xhr.open('GET','http://localhost:8000?name="小李"');
                //3.发送请求
                xhr.send();
                
                //4.返回接收的结果    浏览器接收到服务器的数据
                xhr.onreadystatechange = function (){
                  /*
                  监听状态的改变的事件,当readyState发生了改变,触发该事件,我们可以在这里处理服务端响应的数据
                     readyState 表示状态,是XMLHttpRequest对象的属性
                      * 0 请求访问,还未开始工作[调用完new XMLHttpRequest(),创建一个XMLHttpRequest对象]
                  	  * 1 服务器链接已经建立,完成初始化任务 [xhr.open()调用完毕]
                  	  * 2 发送请求,并且请求已经接收 [xhr.send()调用完毕]
                  	  * 3 正在处理请求,服务端响应了部分数据
                  	  * 4 请求已经完成响应就绪,服务端响应了全部状态
                  所以监听事件触发4次 0-->1-->2-->3-->4
                  */
                        if(xhr.readyState === 4){
                          //处理结果  行 头 空格 体
                  /*
                  status访问状态码 (200 "OK" 403 "Forbidden", 404 "page not found")
                  */          
                   
                            if(xhr.status >=200 && xhr.status<300){
                          //响应行里面包含的数据
                            console.log(xhr.status); //状态码
                            console.log(xhr.statusText);//返回文本状态
                           console.log(xhr.getAllResponseHeaders());//所有的响应头
                           console.log(xhr.response);//响应数据
                             //将服务器里面的内容传递给 浏览器中
                                box.innerHTML = xhr.response;      
                            }
                        }
                }
            }   
        </script>
```

```js
 服务器 --- > 浏览器
//创建的基本服务器     
//1.引入 express 
const express  = require('express');
//2.创建引用对象
const app = express();
//3创建路由规则
app.get('/',((req,res)=>{
        // 必须配置 setHeader() 响应体中的 'Access-Control-Allow-Origin' 属性 实现允许跨域
    	// AJAX的跨域操作 基于 设置响应头 Access-Control-Allow-Origin 
        res.setHeader('Access-Control-Allow-Origin','*');
    //设置响应体,将响应数据传递给前端.
        res.send('New AJAX');
}))
//4.创建监听端口号
app.listen(8000,()=>{
    console.log("服务已经启动,8000端口监听ing");
})
```

## 2.Ajax发送POST请求(传递参数)

```js
    let  xhl =  new XMLHttpRequest();
            let name ="李白";
            let age = 20
            xhl.open('POST','http://localhost:8000/post-argument');

            /*
            post请求的参数一般都放在请求体中(当然也可以通过url传参),所以我只需要编写请求体的内容,就能做到给服务端传参的效果
            Post 方法传递参数 通过 xhr.send()方法传递 具有多种格式:
                * xhl.send('hello server')
        		* xhl.send('username:li&age:20')
        		* xhl.send('username=li&age=20')
        		* xhl.send("姓名 = "+name+"年龄 = "+age); 自定义name和age属性
            */
            xhl.send("姓名 = "+name+"年龄 = "+age);
            xhl.onreadystatechange = function(){
                if(xhl.readyState === 4){
                    if(xhl.status>=200 && xhl.status<300){
                        console.log(xhl.response);
                    }

                }
            }
        </script>
```

```js
 服务器 --- > 浏览器
//创建的基本服务器     
//1.引入 express 
const express  = require('express');
//2.创建引用对象
const app = express();
//3创建路由规则 处理post 请求
app.psot('/post-argument',((req,res)=>{
  
    // 设置响应头  Access-Control-Allow-Origin  允许跨越操作
        res.setHeader('Access-Control-Allow-Origin','*');
    // 设置响应体, 将响应数据传递给前端.
        res.send('New AJAX');
}))
//4.创建监听端口号
app.listen(8000,()=>{
    console.log("服务已经启动,8000端口监听ing");
})
```

# AJAX设置请求头信息

```html
  <script>
            let  xhr = new XMLHttpRequest();
            xhr.open("POST", "http://localhost:8000/setHeader");
            /* 我们一般会把身份校验的信息放在请求头里,传递给服务器,由服务器做提取,做一个身份的校验

            请求头
                * 预定义请求头: HTTP内置的请求头
                * 自定义请求头: 用户自定义的请求头
            * */
            //设置预定义请求头 :
            xhr.setRequestHeader('Context-Type', 'application/x-www-form-urlencoded');
            //设置自定义请求头 :
            xhr.setRequestHeader("name","Jue");
            //设置自定义请求头需要在后端设置响应头res.setHeader('Access-Control-Allow-Headers', '*'),这样才不会对自定义请求头报错

            xhr.send();

            xhr.onreadystatechange = function () {
              if (xhr.readyState==4 && xhr.status >=200 && xhr.status<300){
                  console.log(xhr.response);
              }
            }
        </script>
```

```js
//搭建服务器
const express = require('express');

const app = express();

app.post("/setHeader",(req,res)=>{
    //设置响应头  解决跨域问题
    res.setHeader('Access-Control-Allow-Origin',"*");
    //设置响应头
    res.setHeader('Access-Control-Allow-Headers', '*')
    //返回数据
    res.send("返回数据");
})

app.listen(8000,()=>{
    console.log("服务器已经开启");
})
```

# AJAX服务端响应JSON数据

```html
    <button class="btn1"> 手动处理JSON数据按钮</button>
        <hr>
        <button class="btn2"> 自动处理JSON数据按钮</button>

        <script>
            //1.手动转化成 JSON 数据 --> 对象格式  使用的JSON.api为 JSON.parse(argument)  参数为 JSON数据
            let btn1 = document.querySelector(".btn1");
            btn1.onclick = function() {
                let xhr = new XMLHttpRequest();
                xhr.open("POST","http://localhost:8000/manageJSON");
                xhr.send();
                xhr.onreadystatechange = function(){
                    if(xhr.readyState===4 && xhr.status>=200 && xhr.status<300){
                        // 直接输出是输出的json格式的数据,不是对象格式的
                        console.log(xhr.response); // {"name":"Jue","age":18}
                        console.log(xhr.response.name); // undefined; 因为是json格式的数据,所以无法通过对象的获取方法获取
                        
                        // json数据 -> 对象 (手动转换) 
                        let person = JSON.parse(xhr.response);
                        console.log(person) // {name: 'Jue', age: 18}
                        console.log(person.name) // Jue; 因为已经转成了对象类型的数据,所有可以直接获取其属性的值
                    }
                }
            }
            
            //2.自动转化成 JSON 数据 --> 对象格式
            let btn2 = document.querySelector(".btn2");
            btn2.onclick = function() {
                let xhr = new XMLHttpRequest();
                // 设置响应体的数据的类型为json格式的 可以是实现自动转换
                xhr.responseType = 'json'
                
                xhr.open("POST","http://localhost:8000/manageJSON");
                xhr.send();
                xhr.onreadystatechange = function(){
                    if(xhr.readyState===4 && xhr.status>=200 && xhr.status<300){
                        //自动转换完成后 直接输出是输出的json格式的数据 是对象的格式
                        console.log(xhr.response); //{   "name": "Jue", "age": 20}
                        
                        console.log(xhr.response.name); //Jue
                        
                    }
                }
            }
        </script>
```

```js
/*  在web服务器传递数据

    字符串数据,可以直接响应

    其他的数据(数字 对象 数组 布尔 Null),直接响应是不行的,我们可以将其他数据转成json格式的数据,进行传递
    将其他数据转换成json格式的数据 的 JSON api为JSON.stringify(argument) 参数为json中有效的非基本数据
    json中有效的数据类型 为 字符串 数字 对象（JSON 对象） 数组 布尔 Null
 */
//搭建服务器
const express = require('express');

const app = express();

app.post("/manageJSON",(req,res)=>{
    //设置响应头  解决跨域问题
    res.setHeader('Access-Control-Allow-Origin',"*");
    //创建对象
    let person ={
        name:"Jue",
        age:20
    }
    
    //在web服务器,返回数据的数据一定为  字符串  
    //对象数据 -> json数据  
    let str = JSON.stringify(person);
    res.send(str);
})


app.listen(8000,()=>{
    console.log("服务器已经开启");
})
```

# AJAX中IE缓存机制

```html
<button id="btn1">发送请求1</button>

<script>
    let btn1 = document.querySelector('#btn1');
    btn1.addEventListener('click', function () {
        let xhr = new XMLHttpRequest()
        /*
            因为ie浏览器有缓存机制,当发送相同的get请求,他会把原先缓存的数据拿来,不会去再访问一次
                * 当服务端响应的数据为'10', 浏览器第一次请求'http://127.0.0.1:9999/server',获取到数10,并且将数据'10'做一次缓存
                * 当服务端响应的数据为'20', 浏览器第二次请求'http://127.0.0.1:9999/server',ie浏览器会检测到与上一次请求相同,就从缓存里将原先的数据拿出来
            针对这个ie缓存机制,我们可以每次都发送不同的请求,搭配 url传参,时间戳 实现,这样就能避免ie缓存机制,每次都会重新去请求服务器
                * 当服务端响应的数据为'10', 浏览器第一次请求'http://127.0.0.1:9999/server?t=1645945217543',获取到数10,并且将数据'10'做一次缓存
                * 当服务端响应的数据为'10', 浏览器第二次请求'http://127.0.0.1:9999/server?t=1645945217590',因为与上一次请求的url路径不同,所以会重新发起请求,获取到数据20,并且将数据'20'做一次缓存

            实际工作中,我们都采用了框架,不需要我们去亲自加时间戳,直接输入想要访问的url地址即可
         */

        xhr.open('POST', 'http://127.0.0.1:8000/server?t=' + Date.now())
        xhr.send()
        xhr.onreadystatechange = () => {
            if (xhr.readyState == 4 && xhr.status >= 200 && xhr.status < 300) {
                console.log(Date.now()) // 1645945217543
                console.log(xhr.response)
            }
        }
    });
</script>
```

# Ajax请求超时和网络异常

```html
<!--
	有可能服务器压力比较大,处理起来比较慢,所以会出现请求了很久,但是没有响应的情况,这就需要做出响应的处理

	当用户的网络出现了异常,我们也需要对此做出响应的处理
-->

<button id="btn1">发送请求1</button>

<script>
    let btn1 = document.querySelector('#btn1');
    btn1.addEventListener('click', function () {
    let  xhr =  new XMLHttpRequest();
    xhr.open('POST','http://localhost:8000/fail');
    xhr.send();
    //允许事件延迟在2秒内,两秒之后没有响应,就会自动取消请求.
    xhr.timeout = 2000;
    //超时的回调函数
    xhr.addEventListener('timeout',function () {
            alert("服务器出现延迟");
    })
    //无法访问网络时,会自动调用这个回调函数
    xhr.addEventListener('error',function () {
            alert("网络请求失败");
    })
    xhr.onreadystatechange = function(){
        if(xhr.readyState === 4){
            if(xhr.status>=200 && xhr.status<300){
                console.log(xhr.response);
            }

        }
    }
    });
</script>
```

```js
const express = require('express')
const app = express()

app.all('/server', (req, res) => {
    // 设置响应头,允许跨域
    res.setHeader('Access-Control-Allow-Origin', '*')

    // 模拟延迟响应的效果  延迟3000返发送
    setTimeout(() => {
        res.send('hello world')
    }, 3000)
})

app.listen('8000', () => {
    console.log('server is running')
})
```

# AJAX手动取消请求

![image-20220402195950663](C:/Users/JueCheng/AppData/Roaming/Typora/typora-user-images/image-20220402195950663.png)

```html
<!--
	上面的xhr.timeout=2000设置,是超时了2s后,自动取消请求

	可以通过xhr.abort()手动取消刚才发送的请求

	在浏览器的开发者工具里的network里可以看到请求取消了的信息
-->
<button> 发送请求</button>
<hr>
<button> 取消请求</button>

<script>
        let btn1 = document.querySelectorAll("button");
        let xhr = new XMLHttpRequest();
        //按钮1 正常发送请求
        btn1[0].addEventListener("click",function () {

                xhr.open("POST", "http://localhost:8000/cancel");
                xhr.send();
                xhr.onreadystatechange = function () {
                    if(xhr.readyState == 4 && xhr.status >= 200&&xhr.status<300){
                        console.log(xhr.response);
                    }
                }
        })
        //按钮2  取消请求的发送
        btn1[1].addEventListener("click",function (){
            //xhr的abort() 方法取消请求的发送
                xhr.abort();
        })

</script>

```

```js
const express = require('express')
const app = express()
app.post("/cancel",(req,res)=>{
    //设置响应头  解决跨域问题
    res.setHeader('Access-Control-Allow-Origin',"*");

    res.send("传递的数据");
})
app.listen(8000,()=>{
    console.log("服务器已经开启");
})
```

# AJAX重复发送请求

![image-20220402202322023](https://notebook-img.oss-cn-shanghai.aliyuncs.com/Webpack/202204022023474.png)

```html
<!--
    当重复发送请求的时候,我们可以将上一次的请求取消掉,然后发送新的请求,这样就不会出现请求积压的情况
-->
            <button> 重复请求 </button>

<script>
            let btn = document.querySelector("button");

            //建一个标识变量,用于判断,是否正在发送请求(即节流阀的使用)
            let  isSend = false;
            let xhr = null ;
            btn.addEventListener("click",function(){
                // 如果正在发送请求,就取消上一次请求,然后创建最新的请求
                if(isSend){
                    xhr.abort();
                }
                // 设置isSending=true,表示正在处理请求
                isSend = true;
                xhr = new XMLHttpRequest();
              
                xhr.open("POST", "http://localhost:8000/repeat");
                xhr.send();
                xhr.onreadystatechange = function () {
                    if (xhr.readyState == 4 && xhr.status >= 200 && xhr.status<300){
                        console.log(xhr.response);
                        // 设置isSending=false,表示请求已经处理完毕,可以去处理下一次请求

                    }
                }
            })
</script>
```

```js
const express = require('express')
const app = express()

app.all('/repeat', (req, res) => {
    // 设置响应头,允许跨域
    res.setHeader('Access-Control-Allow-Origin', '*')

    // 模拟延迟响应的效果
    setTimeout(() => {
        res.send('hello world')
    }, 3000)
})

app.listen('8000', () => {
    console.log('server is running')
})
```

# JQuery发送AJAX请求

```html
    <!--引入JQuery库    -->
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.js"></script>
			<button>JQuery发送GET请求</button>
            <hr>
            <button>JQuery发送POST请求</button>
            <hr>
            <button>JQuery发送AJAX</button>
        <script>
            //第一种 GET请求
            $('button').eq(0).click(function(){
                //发送get请求,参数传递为url传参
                $.get("http://localhost:8000/JQuery",{username:"Jue",password:123456},function(data){
                    //{username:" " ,password:} 数据为参数
                    //data是服务器传递回来的响应数据
                    console.log(data); //传递的是JSON数据 没有进行转换 {"name":"JueCheng","age":"20"}
                    //实现转换数据 JSON数据 ---> 对象数据
                    let obj = JSON.parse(data);
                    console.log(obj); //{name:"JueCheng",age:20}
                });

            });
            //第二种 POST请求
            $("button").eq(1).click(function(){
                //发送Post请求,参数传递的方式不为url 传递
                $.post("http://localhost:8000/JQuery",{username:"JueCheng",password:654321},function (data) {
                    //{username:" " ,password:} 数据为参数 但是参数保存在请求体内
                    //data是服务器传递回来的数据
                    console.log(data);// 'json' 实现自动JSON数据的转换  输出的data为对象
                },'json');
            })
            //第三种 AJAX请求--自定义选择请求的发送
            $('button').eq(2).click(function(){
                //通过JQuery发送ajax请求
                 $.ajax({
                    //请求方式
                    type:"post",
                    //请求的地址  GET 请求只能通过地址栏传参
                    url:"http://localhost:8000/JQuery?USERNAME=TEST&password=12345",
                    //请求参数传递
                    //post请求,就是请求体传参也可以地址栏传参
                    //get请求,只能地址栏传参
                    data:{username:"JueCheng",password:12345},
                    //访问成功后的回调函数
                    success:function (data) {
                        console.log(data)//接收响应体的数据,设置了响应结果数据类型为json格式,所有实现了自动转换的形式
                    },
                    //响应体结果的设置  实现JSON数据的自动转换
                    dataType:"JSON",

                    //设置超时时间
                    timeout:2000,
                    //请求失败后的回调函数,比如网络延迟,网络异常
                    error:function () {
                        console.log("错误")
                    },
                    //设置请求头信息
                    headers:{
                        //设置自定义请求头需要在,服务器上配置响应头 res.setHeader("Access-Control-Allow-Headers","*")
                        self_name:"jue",
                        self_age:20
                    }
                })
            })
        </script>
```

```js
const express = require('express')
const app = express()
app.all("/JQuery",(req,res)=>{
    //设置响应头  解决跨域问题
    res.setHeader('Access-Control-Allow-Origin',"*");
    //设置响应头  解决自定义请求头问题
    res.setHeader('Access-Control-Allow-Headers',"*");
    //创建对象
    let person ={name:"JueCheng",age:20};
    //将对象转换成JSON数据
    let str = JSON.stringify(person);
    //传递数据   send(argument) argument参数一定为 字符串
    res.send(str)

})
app.listen(8000,()=>{
    console.log("服务器已经开启");
})
```



# Axios方式AJAX请求

```html
<!--
    Axios是对原生的AJAX进行封装,简化书写的工具包,Axios官网是：https://www.axios-http.cn

    Axios已经为所有支持的请求方法提供了别名
        * get请求: axios.get(url[,config])
        * post请求: axios.post(url[,data[,config])
        * delete请求: axios.delete(url[,config])
        * head请求: axios.head(url[,config])
        * options请求: axios.option(url[,config])
        * put请求: axios.put(url[,data[,config])
        * patch请求: axios.patch(url[,data[,config])
-->
    <!--引入axios, 如果担心报跨域警告,可以加上属性 crossorigin="anonymous" -->
    <script src="https://cdn.bootcdn.net/ajax/libs/axios/0.26.1/axios.js" crossorigin="anonymous"></script>
    <title>Axios发送ajax请求</title>
</head>
<body>
<button>Axios发送 Get请求</button>
<hr>
<button>Axios发送 Post请求</button>
<hr>
<button>Axios发送 Ajax请求</button>

<script>

    let btn = document.querySelectorAll('button');

    btn[0].onclick = function () {
        //axios发送get请求
        axios.get('http://localhost:8000/JQuery', //设置请求的url地址
            {
                //url传递参数
                params: {
                    a: 100, b: 100
                },
                //设置自定义响应头
                headers: {
                    Self_name: "JueCheng",
                    Self_age: 20
                },
                //设置允许延迟时间
                timeout:2000,
            }
        ).then((res) => {  //通过then()方法来处理,响应体返回的结果
            console.log(res);
            //res为一个对象具有的属性为: { data:{....} , status:200 , statusText:"OK", headers:{....} config{....} }
            console.log(res.data);//{name:"JueCheng",age:20} 自动转化成了对象属性
            console.log(res.status); //200  当前状态码
            console.log(res.statusText);//OK  当前状态描述
        }).catch((err) => { //出现网络延迟 或者网络异常处理
            console.log("网络异常或者延迟过久");
        })
    }

    btn[1].onclick = function () {
        //axios发送post请求 设置请求体传参(json数据的格式进行)
        axios.post("http://localhost:8000/JQuery", {name: "Jue", age: 20}, //请求体传参 传递的参数放在请求体里
            {
            //url的实行传参  放在地址栏上
            params: {
                id: "001",
            },
            //设置请求头
            headers: {
                Self_name: "JueCheng",
                Self_name: 20
            }
        }).then((res) => { //then()方法 处理响应体的数据
            console.log(res); //res为对象具有的属性为{ data:{....} , status:200 , statusText:"OK", headers:{....} config{....} }
            console.log(res.data);//{name:"JueHCheng",age:20 }
        });
    }

    btn[2].onclick= function () {
        //axios发送ajax请求
        axios({
            //设置请求方式
            method:"POST",
            //设置请求地址
            url:"http://localhost:8000/JQuery",
            //设置参数 url传参
            params:{
                id:"008"
            },
            //请求体传参数
            data:{
                name:"小白"
            },
            //自定义设置请求头
            headers:{
                name: "jue"
            },
            //设置可接受延迟时间
            timeout:2000,

        }).then((res)=>{//处理响应体的数据
            console.log(res)//返回的是对象  {data:{...},headers:{},status:200,statusText:"OK",config:{...}}
            console.log(res.data);//{name:"JueCheng",age:20}
        }).catch(()=>{     //出现网络延迟 或者网络异常处理
            console.log("出现错误")
        })
    }

</script>
```

```js
const express = require('express')
const app = express()
app.all("/JQuery",(req,res)=>{
    //设置响应头  解决跨域问题
    res.setHeader('Access-Control-Allow-Origin',"*");
    //设置响应头  解决自定义请求头问题
    res.setHeader('Access-Control-Allow-Headers',"*");
    //创建对象
    let person ={name:"JueCheng",age:20};
    //将对象转换成JSON数据
    let str = JSON.stringify(person);
    //传递数据   send(argument) argument参数一定为 字符串
    res.send(str)

})
app.listen(8000,()=>{
    console.log("服务器已经开启");
})
```

# fetch()发送AJAX请求

```html
<button>fetch()函数发送ajax请求</button>

<script>
    document.querySelector("button").onclick = function () {
        fetch("http://localhost:8000/JQuery",{
            //请求的方法
            method: "POST",
            //请求体传递参数
            body:"username=jue",
            //自定义请求头信息
            headers: {
                name: "jue"
            },
        }).then((res)=>{
            /*
             因为是返回的一个Promise对象,所有需要return,再次调用then()来获取值
                       * res.text() 返回浏览器响应数据
                      * res.json() 将JSON数据进行转换再返回
            */
            return res.json();
        }).then((data)=>{
            console.log(data);//{name:"JueCheng",age:20}
        })
    }
</script>
```

```js
const express = require('express')
const app = express()
app.all("/JQuery",(req,res)=>{
    //设置响应头  解决跨域问题
    res.setHeader('Access-Control-Allow-Origin',"*");
    //设置响应头  解决自定义请求头问题
    res.setHeader('Access-Control-Allow-Headers',"*");
    //创建对象
    let person ={name:"JueCheng",age:20};
    //将对象转换成JSON数据
    let str = JSON.stringify(person);
    //传递数据   send(argument) argument参数一定为 字符串
    res.send(str)

})
app.listen(8000,()=>{
    console.log("服务器已经开启");
})
```

# 跨域

### 同源策略

```js
/*
    同源策略(Same-Origin Policy)最早由Netscape公司提出,是浏览器的一种安全策略(AJAX是默认遵循同源策略的)
        * 协议、域名、端口号必须完全相同,这才满足同源策略
        * 比如: 当前目录url为http://182.142.145.11:8080,我们想要发送请求,那么url也必须为http://182.142.145.11:8080
    违背同源策略就是跨域
    
    浏览器访问"http://localhost:9999/home",服务器响应index.html页面到浏览器,然后我们点击index.html页面的button按钮,再像服务器发送请求,获取数据,此时就满足同源策略
    
    我们之前在webstorm开启的服务器上访问了index.html页面,然后向服务器"http://localhost:9999"发送请求,明显这就是跨域了
*/
const express = require('express')
const app = express()

app.get('/home', (req, res) => {
    // 响应一个页面
    res.sendFile(__dirname + '/index.html')
})

app.get('/data', (req, res) => {
    res.send('用户数据...')
})

app.listen('9999', () => {
    console.log('server is running')
})
```

# JSONP解决跨域

### JSONP原理

```html
<!--
    在网页中,有一些标签本身就具有跨域的能力
        * 比如: link, img, iframe, script
        * 比如: 我们通过script跨域引入一个的js文件 <script src="http://localhost:63342/ajax/demo01/app.js"></script>
	这个方法,只能发送get请求,因为这些标签的参数通过url的方式传递,所以jsonp的type类型只能是get 
-->

<!--通过script,向服务器发送一个跨域的请求,是可以实现的-->
<script src="http://localhost:8000/jsonp-server"></script>
```

```js
const express = require('express')
const app = express()

app.get('/jsonp-server', (req, res) => {
    //
    /*
        不能直接通过res.send()响应数据给浏览器,因为script请求的是js表达式
            res.send('hello world') // error
        所以我们可以发送一些js表达式到浏览器,比如: console.log()
     */
    res.send('console.log("hello world")')
})
app.listen('8000', () => {
    console.log('server is running')
})
```

### JSONP的实践

```html
<button>JOSNP的实践</button>
<p></p>
<script>
    document.querySelector('button').onclick = function(){
        //创建一个script标签,设置src为要请求的路径,然后将该script标签插入到页面里
        let script = document.createElement('script');
        script.src = 'http://localhost:8000/jsonp-server';
        document.body.append(script);
    }
    //编写handle()函数,由script跨域请求到的js表达式来调用
    function handle(data){
        //data直接输出的就是对象格式的数据类型,而不是JSON数据类型可以直接使用
        console.log(data);//{name:" ",age:20}
        //插入数据到标签内部
        document.querySelector("p").innerHTML = data.name;
    }

</script>
```

```js
const express = require('express')
const app = express()
app.all("/jsonp-server",(req,res)=>{
    //设置响应头  解决跨域问题
    res.setHeader('Access-Control-Allow-Origin',"*");
    //设置响应头  解决自定义请求头问题
    res.setHeader('Access-Control-Allow-Headers',"*");
    //创建对象
    let person ={name:"JueCheng",age:20};
    //将对象转换成JSON数据
    let str = JSON.stringify(person);
    //传递数据   send(argument) argument参数一定为 字符串
    //响应一个js表达式,调用handle方法,传入参数
    res.send(`handle(${str})`);

})
app.listen(8000,()=>{
    console.log("服务器已经开启");
})
```

### JQuery发送JSONP请求

```html
    <!--引入JQuery库    -->
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.6.0/jquery.js"></script>
</head>
<body>
   <button>JQuery发送jsonp请求</button>
   <p></p>
    <script>
        let btn  = document.querySelector("button");
        btn.onclick = function(){
            //设置url路径时,要注意添加一个参数 "callback=?"
            $.getJSON('http://127.0.0.1:8000/jquery-jsonp-server?callback=?',function (data) {
                //data 直接就是对象格式的数据,而不是JSON格式的,可以直接使用
                console.log(data);//{name:"JueCheng"}
                //插入数据到标签内部
                $('p').html(`${data.name}`);
            })
        }
    </script>

</body>
```

```js

const express = require('express');
const app = express();
app.all("/jquery-jsonp-server",(req,res)=>{
    //设置响应头  解决跨域问题
    res.setHeader('Access-Control-Allow-Origin',"*");
    //创建对象
    let person ={name:"JueCheng",age:20};
    //将对象转换成JSON数据
    let str = JSON.stringify(person);
    //通过req.query获取url参数中设置的callback
    let callback = req.query.callback;
    //传递数据   send(argument) argument参数一定为 字符串
    //返回js表达式,调用函数,callback(str)
    res.send(`${callback}(${str})`);

})
app.listen(8000,()=>{
    console.log("服务器已经开启");
})
```

# CORS解决跨域

```js
/*
    CORS(Cross-Origin Resource Sharing),跨域资源共享,是官方的跨域解决方案
    它的特点是不需要在客户端做任何特殊的操作,完全在服务器中进行处理,支持get和post请求,只需要添加一个响应头,就可以完成跨域的设置
    
        // 设置所有的url地址,都可以向本服务器发送请求
        res.setHeader('Access-Control-Allow-Origin', "*")
        // 设置"http://127.0.0.1:5000"地址,可以向本服务器发送请求
        res.setHeader('Access-Control-Allow-Origin', "http://127.0.0.1:5000")
 */

const express = require('express')
const app = express()
app.all("/JQuery",(req,res)=>{
    //设置响应头  解决跨域问题
    res.setHeader('Access-Control-Allow-Origin',"*");
    //设置响应头  解决自定义请求头问题
    res.setHeader('Access-Control-Allow-Headers',"*");
    //创建对象
    let person ={name:"JueCheng",age:20};
    //将对象转换成JSON数据
    let str = JSON.stringify(person);
    //传递数据   send(argument) argument参数一定为 字符串
    res.send(str)

})
app.listen(8000,()=>{
    console.log("服务器已经开启");
})
```



















