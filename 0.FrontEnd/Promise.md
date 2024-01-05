## Promise的定义和使用

理解 : Promise是JS中异步编程的解决方案;语法层次上Promise为一个构造函数;

功能实现上Promise对象用来封装一个异步操作可以获取其成功/失败的结果.

异步编程有:

- 文件操作
- 数据库操作
- AJAX
- 定时器

优点 : 回调的方式更加灵活   启动异步任务 => 返回promise对象 => 给promise对象绑定回调函数; 

支持链式调用解决回调地狱的问题

## Promise封装--FS文件读取

```js
/**
 * 封装一个函数 minReadFile 读取文件内容
 * 参数 : path
 * 返回Promise对象
 */

function minReadFile(path){
    //返回promise对象
    return new Promise((resolve,reject) => {
        require('fs').readFile(path,(err,data) => {
            if(err) reject(err);
            resolve(data);
        })
    })
}
minReadFile('../Resource/1.learning.md').then(value =>{
    //输出文件内容
    console.log(value.toString());
}).catch(err=>{
    //输出错误内容
    console.log(err);
})
```

## util.promisify方法

```js
/**
 *   util.promisify方法
 *   可以自动封装promise对象,将原来回调函数风格 转换成 promise风格的函数
 * 	 遵循常见的错误优先的回调风格的函数 例如文件的读取:fs.readFile(url,(err,data) => {})
 */
//引入 util 模块
const util = require('util');
//引入 fs 模板
const  fs  = require('fs');
//返回一个新的函数  
//读取文件
let  mineReadFile = util.promisify(fs.readFile);

mineReadFile('../Resource/1.learning.md').then(value=>{
    console.log(value.toString());
})
//写入文件
let mineWriteFile = util.promisify(fs.writeFile);

mineWriteFile('../Resource/2.writeing.md',"这是文件的写入");
```

**理解 : util.promisify()方法,返回一个promise对象相当于封装好的FS文件读取.只能异步的读取和写入文件.**

## Promise封装--AJAX请求

 ```js
     /**
          * 封装一个函数presentAjax 发送 Get AJAX请求
          * 参数 url
          * 返回结果 promise对象
          */
         function presentAjax (url){
             return new  Promise ((resolve,reject) => {
                 const xhr = new XMLHttpRequest();
                 xhr.open('GET',url);
                 xhr.send();
                 xhr.onreadystatechange = function (){
                     if(xhr.readyState === 4){
                         if(xhr.status >=200 && xhr.status<300){
                             resolve(xhr.response);
                         }else{
                             reject(xhr.status);
                         }
                     }
                 }
             })
         }
         presentAjax('https://api.apiopen.top/searchPoetry?name=古风二首二')
         .then((value) => {
             console.log(value.toString());
         })
 		.catch(err=>{
             console.log(err.toString());
         })
 ```

## Promise的状态

简介 : promise状态属性为,实例对象的一个内置属性 [PromiseState],三种状态.

- pending  未决定的
- resolved / fullfilled 成功的
- rejected 失败的

特点 : 只能从pending --> resolved/rejected,不可以由 resolved <===> rejected.

## Promise的对象值

简介 : promise对象值,为实例对象一个属性 [PromiseResult];

保存着异步任务对象的 成功/失败 的结果,

- resolve 
- reject

## Promise的工作流程图

![image-20220326173029206](https://notebook-img.oss-cn-shanghai.aliyuncs.com/promise-img/image-20220326173029206.png)

## Promise相关的API

#### 1.Promise构造函数

简介 : Promise构造函数:Promise(executor){}

1. executor 函数 : 执行器 (resolve,reject)=>{}
2. resolve  函数 : 内部定义成功时,我们调用的函数
3. reject   函数 : 内部定义失败时,调用的函数

```js
  // 注 : executor函数会在promise内部立即执行同步调用,异步操作在执行器中执行.
let p = new Promise((resolve,reject) => {
    		//构造函数 同步调用
            console.log(111);
        })
      console.log(222);
  // 先输出 111 ,后输出 222 
```

#### 2.Promise的then()方法

简介 : then()方法来指定得到成功value的成功回调和用于得到失败的reason的失败回调返回一个新的promise对象;属于实例对象的方法;

- onResolved 函数 : 成功的回调函数 value  =>{}
- onRejected 函数 : 失败的回调函数 reason =>{}

```js
   let p = new Promise((resolve,reject) => {
            // resolve("成功");
            reject("失败");
        })
      		//promise的then()方法
      p.then(value=>{
          console.log(value); //输出 成功的值
      },reason=>{
          console.log(reason);//输出 失败的值
      })
```

#### 3.Promise的catch()方法

简介 : catch()方法,获得的是失败的回调函数;属于实例对象的方法;

```js
   let p = new Promise((resolve,reject) => {
            // resolve("成功");
            reject("失败");
        })
      //promise的then()方法
      p.then(value=>{
          console.log(value); //输出 成功的值
      }).catch(reason=>{
          console.log(reason);//catch() 方法 失败的回调
      })
```

#### 4.Promise的resolve()方法

简介 : resolve()方法是属于Promise函数对象的方法,结果返回一个成功或者失败的promise对象

```js
      //1. 传递的参数不为 promise类型 ,返回的状态为成功,结果为传入的参数.
        let p = Promise.resolve(123);
        console.log(p);// PromiseState:fullfilled PromiseValue:123
        //2. 传递的参数为 promsie 类型,返回的状态根据参数的类型来决定,结果为promise对象;
        let p1 = Promise.resolve(
            new Promise((resolve,reject)=>{
                resolve("success");
            })
        )
        console.log(p1);// PromiseState:fullfilled PromiseValue:success
        
        let p2 = Promise.resolve(
            new Promise((resolve,reject)=>{
                reject("error");
            })
        )
        p2.catch(reason=>{console.log(reason);})
        console.log(p2);// PromiseState:rejected PromiseValue:error
```

#### 5.Promise的reject()方法

简介 : reject()方法属于Promise函数对象的方法,结果返回一个失败的promise对象;

```js
     //1. 传入的参数不为promise对象 返回的状态为失败,结果为传入的参数.
        let p = Promise.reject(123);
        console.log(p);//PromiseState:rejected PromiseResult:123
        //2. 传入的参数为Promise对象 返回的状态也是失败,结果为promise对象 
        let p1 = Promise.reject(
            new Promise((resolve,reject)=>{
                resolve("success");
                // reject("err")
            })
        )
        console.log(p1);// PromiseState:rejected PromiseResult:promise
```

**理解 : Promise.reject()返回的状态全部为失败的,返回的结果为promise对象.**

#### 6.Promise的all()方法

简介 : all()方法属于Promise函数对象的方法,参数为promise数组,结果返回一个新的成功/失败的promise对象;

```js
		let p1 =  Promise.resolve("success1");
		let p2 =  Promise.resolve("success1");
        let  promises = [p1,p2];

        let  p = Promise.all(promises);
       console.log(p);// 1.p1 p2  都是成功的  ---> p的状态即为成功 返回的数据为 数组 
 

       let p3 =  Promise.resolve("success1");
       let p4 = Promise.reject('success2');
       
       let  promisess = [p3,p4];
       let  p0 = Promise.all(promisess);
       console.log(p0);//2.p3 p4  出现全部为失败 或者 一个/多个为失败  ---> p的状态即为失败 仅仅返回失败的数据
```

#### 7.Promise的race()方法

简介 : race()方法属于Promise函数对象的方法,参数为Promise数组,返回一个新的成功/失败的promise对象;

```js
     //创建数组 Promise数组
        let p1 = Promise.resolve("success1");
        let p2 = Promise.reject("success2");

        let p  = Promise.race([p1,p2]);

        console.log(p); //状态根据由第一个完成promise的结果对象进行决定
```

## Promise自定义封装前关键问题

#### 1.如何修改promise对象的状态

```js
    //修改promise状态的三种方法;   
	let p = new Promise((resolve,reject) => {
                //1.resolve()函数
                // resolve("success");  //pending --> fulfilled/resolve
                
                //2.reject()函数
                // reject("fail");   //pending --> rejected

                //3.抛出错误
                throw new error;   //pending --> rejected
            })
```

#### 2.一个promise指定多个成功/失败回调函数是否都会调用

简介 : 通过then()方法指定多个回调都会执行,当promise状态改变后回调函数都会调用.

```js
    let p = Promise.resolve("Success");

        //状态改变 指定回调函数 会重新执行
        //指定回调函数 -- 1
        p.then(value=>{
            console.log(value);
        }) 
        //指定回调函数 -- 2
        p.then(value=>{
            alert(value);
        })
```

**理解 : 指定回调可以理解为执行then()方法,给promise绑定成功或者失败的回调函数,当promise状态改变后浏览器就会执行成功或者失败的方法.**

#### 3.改变状态与指定回调的顺序

```js
    //两种情况都有可能出现
        //1.在执行器中直接调用  resolve()/reject() 是 同步任务 先执行状态改变,后执行回调
        let p = new Promise((resolve,reject) => {
        //2.在执行器中异步任务  执行resolve()/reject()   先执行回调,后改变状态.
            setTimeout(() => {
                resolve("Success");
            },1000);
        })
        
        p.then(value=>{
            console.log(value);
        },reason=>{
            console.log(reason);
        })
```

**注 : 无论是先改变状态还是先执行回调,都是最后一步执行then()的回调来获取数据.**

#### 4.then()方法的返回结果

![image-20220327111421442](https://notebook-img.oss-cn-shanghai.aliyuncs.com/promise-img/202203271114516.png)

```js
   let promise =  new Promise((resolve,reject)=>{
            //请求成功
            resolve("成功的发送数据");
        })
        //promise调用then方法 then方法的返回结果是Promise对象(可以实现链式调用--->定义:重复使用一段初始化代码,使得代码简洁),
   		//对象的状态由回调函数的执行结果决定
        //1.如果回调函数中的返回的结果是 非 Promise 类型的属性,状态为成功(resolve/fulfilled),返回值为对象的成功值
        //2.返回结果为 Promise类型的属性 ,状态为成功返回 (resolve,fulfilled)返回值为成功值   状态为失败返回(rejected)返回失败的值
   		//3.抛出错位  返回类型为 (rejected)  返回失败值
        let  result =  promise.then((value)=>{
            console.log(value);
            //1.返回非Promise类型的属性
            // return 123;   
            // 2.返回Promise类型 
            // return new Promise((resolve,reject)=>{
            //     reject();
            // })
            //3.抛出错误
            throw new Error("出错了");
        },(error)=>{    
            console.log(error);
        })
        console.log(result);
```

#### 5.then()方法的链式操作

```js
      //链式调用实现读取多个文件
//1.配置环境
const fs  =  require('fs');
//通过promis读取文件
const promise = new Promise((resolve,reject)=>{
    fs.readFile("./Resources/1.Learning.md",(err,data)=>{
          //失败
          if(err) reject(err);
          //成功
        resolve(data.toString());
    })
})

//then方法的链式调用
promise.then((result) => {
    // console.log(result);
    return  new Promise((resolve,reject)=>{
        fs.readFile("./Resources/2.Reading.md",(err,data1)=>{
           resolve([result,data1]);
        })
    })
}).then((value)=>{
    return  new Promise((resolve,reject)=>{
        fs.readFile("./Resources/3.Writing.md",(err,data2)=>{
          value.push(data2);
            //value是一个数组
          resolve(value);
        })
    })
}).then((value)=>{
    console.log(value.join('\r\n'));
});
```

#### 6.异常穿透

简介 : promise中catch()方法的使用,可以在最后输出错误,相当于then()方法失败的回调.

```js
   const p = new Promise((resolve,reject)=>{
            setTimeout(()=>{
                reject("出错了")
            },1000);
           
        })
        p.catch(error=>{
            console.log(error);
        })
```

#### 7.中断promise链

```js
  let  p = new Promise((resolve,reject) => {
            resolve("Success");
        })

        p.then(value=>{
            console.log(value);
            //中断promise链,返回一个状态未定义的promise对象
            return new Promise((resolve,reject) => {})
        }).then(value=>{
            console.log(234);
        }).catch(reason=>{
            console.warn(reason);
        })
```

## Promise自定义封装

```js
//创建 Promise 的构造函数
function Promise(executor) {
    //配置Promise对象的属性
    this.PromiseState = "pending";
    this.PromiseResult = null;
    //创建一个属性来保存 then()方法的两个函数
    this.callback = [];

    //这里的this指向为 promise的实例对象
    const self = this;
    //执行器的参数为函数  创建这两个函数 
    function resolve(data) {
        //函数resolve的this指向--->window

        //进行判断状态只能修改一次 状态只要改变就不可以执行下面代码了
        if (self.PromiseState !== "pending") return;
        //1.修改Promise对象的状态
        self.PromiseState = "fulfilled";
        //2.修改Promise结果
        self.PromiseResult = data;

        //异步任务下 执行器中 执行then()方法的这两个函数
        //目前相当于 在 执行器下的resolve()方法下 去能成功的调用到 then()方法下的Onresolve函数
        //由于将两个方法保存到了 Promise对象的属性下,所以可以调用这两个方法

        setTimeout(() => {
            self.callback.forEach(item => {

                item.Onresolve(self.PromiseResult);
            })
        });
    }
    function reject(data) {

        //进行判断状态只能修改一次 状态只要改变就不可以执行下面代码了
        if (self.PromiseState !== "pending") return;
        //1.修改Promise对象的状态
        self.PromiseState = "rejected";
        //2.修改Promise结果
        self.PromiseResult = data;
        setTimeout(() => {
            self.callback.forEach(item => {

                item.Onreject(self.PromiseResult);
            })
        });

    }

    // try catch 可以处理 throw抛出的异常 所以使用 try catch 将执行器进行包裹起来;
    try {
        //执行器 为 同步函数
        executor(resolve, reject);
    } catch (error) {
        //抛出错误 说明状态为失败,直接调用 reject()函数,参数为失败的原因
        reject(error);
    }

}

//创建then()方法  then()方法接收两个参数 
Promise.prototype.then = function (Onresolve, Onreject) {
    //判读回调函数参数  帮助实现catch()方法的穿透性
    if (typeof Onreject !== 'function') {
        Onreject = reason => {
            throw reason;
        }
    }
    if (typeof Onresolve !== "function") {
        Onresolve = value => value;
    }
    const self = this;
    return new Promise((resolve, reject) => {
        //封装函数 callback()
        function callback(type) {
            try {
                //获取回调函数的执行结果
                let result = type(self.PromiseResult);
                //判断执行结果
                if (result instanceof Promise) {
                    //1.返回的结果为 Promise对象
                    //判断Promise对象的状态
                    result.then(v => {
                        resolve(v);
                    }, r => {
                        reject(r);
                    })
                } else {
                    // 2.返回的结果 为 非Promise对象  结果的对象状态为成功
                    resolve(result);
                }
            } catch (e) {
                //3.throw 传出错误
                reject(e);
            }
        }
        //同步任务 返回then()结果 返回值为 Promise类型的对象
        //同步任务下 在 then()方法中执行两个函数
        if (this.PromiseState === 'fulfilled') {
            setTimeout(() => {
                callback(Onresolve);
            } );

        }
        if (this.PromiseState === "rejected") {
            setTimeout(() => {
                callback(Onreject);
            });

        }

        //异步任务下 应该在 执行器中 执行then()方法的这两个函数
        //同步任务 返回then()结果 返回值为 Promise类型的对象
        //判断 pending 的状态

        if (this.PromiseState === "pending") {
            //保存回调函数 因为状态不确定,没办法直接执行then()方法下的函数
            //将then()方法下的两个函数保存到Promise对象的属性上
            this.callback.push(
                {
                    Onresolve: function () {
                        callback(Onresolve);
                    },
                    Onreject: function () {
                        callback(Onreject);
                    }
                }
            )
        }
    })
}

//创建catch()方法  
Promise.prototype.catch = function (Onreject) {
    return this.then(undefined, Onreject);
}

//创建resolve()方法
Promise.resolve = function (value) {
    //返回Promise对象
    return new Promise((resolve, reject) => {
        //判断resolve()返回的结果 为Promise对象
        if (value instanceof Promise) {
            value.then((v) => {
                resolve(v);
            }, (e) => {
                reject(e);
            })
        } else { //不为Promise对象
            resolve(value);
        }
    })

}

//创建reject()方法
Promise.reject = function (error) {
    return new Promise((resolve, reject) => {
        reject(error);
    })
}

//自定义封装 all()方法
Promise.all = function (promises) {
    return new Promise((resolve, reject) => {
        let count = 0;
        let arr = [];
        //传入的是Promise 实例对象的 数组
        for (let i = 0; i < promises.length; i++) {
            promises[i].then(v => {
                count++;
                arr[i] = v;
                if (count === promises.length) {
                    resolve(arr);
                }
            }, r => {
                reject(r);
            })
        }
    })
}

//自定义封装 race()方法
Promise.race = function (promises) {
    return new Promise((resolve, reject) => {
        for (let i = 0; i < promises.length; i++) {
            promises[i].then((value) => {
                resolve(value);
            }, (error) => {
                reject(error);
            })
        }
    })
}
```

#### 1.自定义封装resolve()方法

```js
//创建resolve()方法
Promise.resolve = function (value){
    //返回Promise对象
    return new Promise((resolve,reject)=>{
         //判断resolve()返回的结果
         if(value instanceof Promise){
                value.then((v) => {
                    resolve(v);
                },(e) => {
                    reject(e);
                })
            }else{
                resolve(value);
            }
    })
}
```

#### 2.自定义封装reject()方法

```js
//创建reject()方法
Promise.reject = function(error){
    return new Promise((resolve,reject) => {
           reject(error);  
    })
}
```

#### 3.自定义封装all()方法

```js
//自定义封装 all()方法
Promise.all = function(promises){
        return new Promise((resolve,reject) => {
            let count = 0;
            let arr= [];
            //传入的是Promise 实例对象的 数组
            for(let i=0;i<promises.length;i++){
                promises[i].then(v=>{
                    count++;
                    arr[i] = v;
                    //当所有的 promise对象都是成功的时候,才实现返回
                   if(count === promises.length){
                       resolve(arr);
                   }
                },r=>{
                    reject(r);
                })
            }
        })
}
```

#### 4.自定义封装race()方法

```js
//自定义封装 race()方法  根据第一个的状态进行返回.
Promise.race =function(promises){
    return new Promise((resolve,reject) => {
        for(let i=0; i<promises.length;i++){
            promises[i].then((value) => {
                resolve(value);
            },(error)=>{
                reject(error);
            })
        }
    })
}
```

## async 和 await 

#### 1.async函数

简介 : 函数的返回值为 Promise 对象,Promise 对象的结果 由async函数的执行的返回值决定;

注  :  相同于then()方法的返回结果

```js
  async function main(){
            //1.返回结果为 非 Promise 类型 状态为成功
            return 123;
            //2.返回结果为 Promise 类型 根据resolve()/reject() 来决定
            return new Promise((resolve,reject) => {
                resolve("Success");
            })
            //3.抛出错误 状态为 失败
              throw "error";
        }
        let  result = main();
        console.log(result);
```

#### 2.await表达式

1. await 必须写在 async函数(单向依赖)
2. await 右侧的表达式一般为promise对象
3. await 返回promise成功的值
4. await 的promise失败了,就会抛出异常,需要通过try...catch捕获处理

```js
     //创建promise对象
        let p = new  Promise((resolve,reject) => {
            resolve("用户数据")
            // reject("失败了")
        })
        // await 要放在 async函数里
        async function fn(){
            //失败的情况 需要try...catch()来处理
            try{
                //调用成功的结果
            let result =  await p;
            console.log(result);
            }catch(e){
                console.log(e);
            }
        }
        fn();
```

理解 : async函数返回的是Promise对象,类似于promise的then()方法;需要try...catch(){}来进行监视处理,而且async函数通过await关键字来接收到promise传递的数据,调用async函数实现promise数据的展现.

#### 3.async函数 搭配AJAX发送请求

```js
       function customAjax (path){
            return new  Promise ((resolve,reject) => {
                const xhr = new XMLHttpRequest();
                xhr.open('GET',path);
                xhr.send();
                xhr.onreadystatechange = function (){
                    if(xhr.readyState === 4){
                        if(xhr.status >=200 && xhr.status<300){
                            resolve(xhr.response);
                        }else{
                            reject(xhr.status);
                        }
                    }
                }
            })
        }

        let customajax = customAjax('https://api.apiopen.top/searchPoetry?name=古风二首二');
        async function main(){
            try{
                let message = await customajax //Promise 类型的数据
                console.log(message);
            }catch(e){
                console.log(e);
            }
        }
        main();
```

