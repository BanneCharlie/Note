#  HTML5

## 1.HTML实体(转义字符)

```html
<body>
    <!-- 在网页中,出现多个空格会转化成一个空格;

        在HTML中,我们不能直接使用一些特殊的符号,需要实体进行转化
        比如:连续的空格 字母两边的大于和小于号 
        
        使用HTML的实体(转义字符)来实现
            实体语法:
                    &实体的名字;
                        &nbsp; 空格
                        &gt;   大于号
                        &lt;   小于号
                        &copy; 版权所有等   
        可以通过文档查询-->
        <h1>Hello&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;World</h1>

        <p>a &lt; b &gt; c</p>
</body>
```

## 2.meta标签(元标签)

```html
<head>
    <meta charset="UTF-8">
    <!-- 三秒跳转百度 -->
    <meta http-equiv="refresh" content="3;url=https://www.baidu.com">
    <!-- 关键字  -->
    <meta name="keywords" content="width=device-width,initial-scale=1.0">
    <!-- 描述 -->
    <meta name="description" content="对于网页的描述">
    <title>meta 标签</title>
    <!-- meta标签用于设置网页的元数据,元数据用来配合浏览器和搜索引擎的使用
            meta的一些属性:
                        charset 指定网站的字符集(UTF-8)
                        name    指定数据名称(content)
                        content 指定数据的内容
                        http-equiv 重定向网站(content)
                    name中的固定属性:
                        keywords 网站关键字
                        description 网站的描述
                    http-equiv固有属性:
                        refresh content(内容具有跳转的时间和跳转的网页)                   
                    -->
</head>
```

## 3.块元素和行内元素

```html
<!-- 块元素(block element)
		在网页中一般通过块元素来对网页进行布局
	行内元素(inline element)
		行内元素主要用来包裹文字

	块元素可以包裹行内元素,行内元素不可以包裹块元素
	块元素基本上都可以放   注意:p元素除外  特殊的块元素 
	本质为一个文字标签  p可以包含行内元素
	
在浏览器解析网页的时候,会对网页不符的内容进行修改:
	比如:
			标签写在根元素外部
			p元素中嵌套了块元素
			.... ....
注意事项:p元素不可以嵌套块元素
	-->
```

## 4.布局标签(结构布局)

```html
<body>
    <!-- 
        header  网页头部
        main    网页主体(一个网页只有一个主体)
        nav     网页的导航栏
        section 一个独立的区块
        aside   表示侧边栏  
        article 表示文章

        div     没有语义,就是表示一个区块,最主要的布局元素
        span    行内元素  一般用于在网页中选中文字
	元素的命名一般最好为英文命名   
     -->
     <header></header>
     <nav></nav>
     <section>
         <!-- 左边栏 -->
         <aside></aside>
         <!-- 右边栏 -->
         <aside></aside>
     </section>
     <footer></footer>
     
</body>
```

### 4.1列表标签

```html
<body>
    <!-- 列表标签有三种:
		有序列表
		无序列表
		定义列表
列表其实就是相当于 一个大的div 包括许多小的div
-->
    <!-- 无序列表 -->
    <ul>
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
    <!-- 有序列表 -->
    <ol>
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ol>
    <!-- 定义列表 -->
    <dl>
        <dt>定义</dt>
        <dd>啊对对对</dd>
        <dd>啊对对对</dd>
    </dl>
</body>
```

### 4.2超链接标签

```html
<body>
    <!-- 超链接可以让我们跳转到其他页面,
            或者是当前页面的其他位置

        使用 a 标签来定义超链接
            属性:
                href:指定跳转的路径
                可以是一个外部网站
                也可以是一个内部网站
                target(目标):指定打开超链接的位置
                _self 默认值 在当前页面打开
                _blank 在一个新的页面打开
        注意:超链接也是一个行内元素,在a标签中可以嵌套除了它自身外的任何元素 

        跳转到当前页面的其他位置
            将超链接的href设置为# 页面不会发生跳转会跳转到当前页面的顶部
            跳转到页面的指定位置,只需要将href 属性设置为 #目标元素的id属性值
            id属性(唯一不重复)
                每个标签都可以添加一个id 属性
                id为唯一的标识符,同页面不能出现重复
            -->
            <!-- 跳转到外部网站 -->
            <a href="https://www.baidu.com" target="_blank">外部网站跳转</a>
            <br> <br>
            <!-- 跳转到内部网站  ./表示当前目录(也可以省略)   ../表示上一级目录-->
            <a href="./05语义化标签(列表标签).html">内部网站跳转</a>
            <br>  <br>
            <!-- 跳转到当前页面的其他位置 -->
		<a href="#bottom">到达底部</a>
            <div></div>
            <a id="bottom" href="#">回到顶部</a>
</body>
```

### 4.3图片标签

```html
<body>
    <!-- 图片标签
            相当于从当前页面引入一个外部标签
            使用img标签来引入外部元素 img是一个自结束标签 
            img属于替换元素(块和行内元素之间,具有两种元素的特点)
                属性:
                    src 指定的是外部图片的位置
                    alt 图片的描述(当图片加载不出来的时候会出现的内容)
                    width 宽度
                    height 高度
                改变宽度或者高度 图片会等比例缩放
            
        注:一般Pc端不建议修改图片的大小,一般直接裁剪
    图片的格式:
            jpg
                支持的颜色比较丰富,不支持透明效果,不支持动图
                一般用于照片
            gif 
                支持的颜色比较少,支持简单透明效果,支持动图
            png 
                支持的颜色比较丰富,支持复杂透明效果,不支持动图
            webp(谷歌公司推出 兼容性比较差)
                兼容性不好 其余都很好
            base64
                将图片进行base64 编码,将图片转化成字符,通过字符引入图片 (字符太多 我省略了)
                优点:base64编码几乎不需要加载 一般作为网页的背景图
    -->

        <img width="400px" src="../Image/微信图片_20220102180816.png" alt="初音未来">
</body>
```

### 4.4内联框架(不常用)

```html
<body>
    <!-- 内联框架
            当前页面引入一个其他的页面
            src 引入的路径
            frameborder 指定内联框架的边框 0 代表不具备边框 -->
        <iframe src="./05语义化标签(列表标签).html" frameborder="0"></iframe>
</body>
```

### 4.5音视频

```html
<body>
    <!-- audio 标签向页面引入音频文件
        video  标签向页面引入视频文件(preload 未播放视频的页面)
        属性相同
            属性:
                autoplay 自动播放
                loop      循环播放
                controls  用户控制
        两种引入方式:
            1.正常引入
            2.source(保持文件的兼容性问题)
             -->
        <audio src="../Audio/祖娅纳惜 - 孤勇者.mp3"  controls loop>

        </audio>

        <audio controls loop >
            <source src="../Audio/祖娅纳惜 - 孤勇者.mp3">
            <source src="../Audio/祖娅纳惜 - 孤勇者.ogg">
        </audio>
        <!-- 音频相同  -->
</body>
```

### 4.6表格

```html
 <title>表格</title>
    <style>
        table{
            width: 1200px;
        }
    </style>
</head>
<body>
    <!-- table 表示一个表格  tr代表行  td 代表列
         border 表示边框的粗细
         width 表示盒子的宽度   
         合并两行 rowspan   合并两列 colspan 
		表格相对而言只有在特殊规定的情况下使用,一般不使用
    -->
    <table border="1">
        <tr>
            <td>1</td>
            <td>2</td>
            <td>3</td>
            <td rowspan="2">4</td>
        </tr>
        <tr>
            <td>1</td>
            <td>2</td>
            <td>3</td>
        </tr>
        <tr>
            <td>1</td>
            <td>2</td>
            <td colspan="2">3</td>
        </tr>
    </table>
```

### 4.7表单

```html
   <title>表单</title>
</head>
<body>
    <!-- from的属性 
        action 表单要提交的服务器地址
        文本框 
            text     指定一个name属性
        密码框
            password 需要指定name
        单选按钮
            radio   必须设置相同的name属性
            必须指定value属性 将信息传给服务器
            checked默认被选中
        多选框
            checkbox  name属性相同 设置 value属性 
        下拉列表
            select   option

        三个按钮 submit(提交按钮) button(普通按钮配合JS) reset(重置按钮)
         关闭自动补全 autocomplete 属性值 on / off
         禁用表单 disabled 
         自动获取焦点 autofocus
    -->
    <form action="#">
        所有的文本框必须设置name属性,value值将传递给服务器.
        文本框<input type="text" name="username">
        <br> <br>
        密码 <input type="password" name="password">
        <br> <br>
        性别 <input type="radio" name="sex" value="man">男 <input type="radio" name="sex" value="woman"> 女 
        <br> <br>
        下拉列表
        <select name="下拉" id="">
            <option value="one">1</option>
            <option value="two">2</option>
            <option value="three">3</option>
            <option value="four">3</option>
            <option value="five">4</option>
        </select>
        <input type="submit" value="提交">
    </form>
</body>
```

# CSS3

## 1.CSS的基本介绍

```html
<head>
    <meta charset="UTF-8">
    <title>CSS的简介和语法</title>
    <style>
        /*内部样式*/
        p{
            color: red;
        }
        /* CSS中的注释, 注释中的内容会自动被浏览器忽略
           CSS的基本语法:
                选择器 声明块
                选择器:通过选择器可以选中页面中指定的元素
                
                声明块:通过声明块来指定要为元素设置的样式
                    声明块由一个一个的声明组成
                    声明是一个名值对结构  一个样式名对一个样式值,名和值之间以:连接,以;结尾.
            */
    </style>
</head>
<body>
    <!-- CSS样式具有 三种 样式
        1.层叠样式表
        2.内部样式表
        3.外部样式表(常用)
        -->
    <p>你好呀 CSS样式</p>
</body>
```

## 2.CSS选择器

### 2.1 CSS的常用选择器

```html
<head>
    <meta charset="UTF-8">
    <title>常用选择器</title>
    <style>
        /* 四种常用的选择器:
            1.元素选择器
            作用:选择指定的所有元素
            语法: 元素名{}
            2.id选择器
            作用:根据元素的id属性值选择一个元素
            语法:#id属性值{}
            3.类选择器
            作用:根据元素的class属性值选中一组元素
            语法:.class属性值{}
            4.通配选择器
            作用:选中html标签的所有属性
            语法:*{} 
        */
        p{
            color: red;
        }
        #id_selector{
            color: pink;
        }
        .class_selector{
            color: green;
        }
    </style>

</head>
<body>
    <p>你好</p>
    <p id="id_selector">你好</p>
    <p class="class_selector">你好</p>
    <p class="class_selector">你好</p>
</body>
```

### 2.2复合选择器

```html
<head>
    <meta charset="UTF-8">
    <title>复合选择器</title>
    <style>
        /* 复合选择器存在两种:
                    1.交集选择器
                    2.并集选择器
         */
         /* 交集选择器
                作用:选中同时符合多个条件的元素
                语法:选择器1选择器2选择器n{}
                注:交集选择器如果存在元素选择器,必须优先元素选择器
         */
          div.class_selector{
                color: red;
          }
          span.class_selector{
                font-size: 20px;
          }
          /* 并集选择器
                作用:选中多个选择器对应的元素
                语法:选择器1,选择器2,选择器3,选择器n{} */
        .box1,.box2,.box3{
            color: green;
        }
    </style>
</head>
<body>

    <div class="class_selector">我是div</div>
    <span  class="class_selector" >我是span</span>
    <p class="box1">1</p>
    <p class="box2">2</p>
    <p class="box3">3</p>
</body>
```

### 2.3关系选择器

```html
<head>
    <meta charset="UTF-8">
    <title>关系选择器</title>
    <style>
        /* 关系选择器:
                    1.子元素选择器
                    2.后代选择器
                    3.兄弟选择器 
        */
        /* 子元素选择器
            作用：选中指定父元素的选择器
            语法：父元素>子元素 
        */
        /* 后代选择器
            作用：选中指定元素的指定后代
            语法：祖先 后代  
        */
        /* 选择下一个兄弟
            语法:前一个 + 后一个
            选择下面所有兄弟
            语法:前一个 ~ 后一个
        
        关系选择器中最常用的是 后代选择器
         */
        
        
        div>span{
            color: red;
        }
        div p{
            color: green;
        }
        div + span{
            color: blue;
        }
        div ~ span{
            color: pink;
        }
    </style>
</head>
<body>
    <div>
        我是父元素
        <div>我是子元素
                <p>我是子元素也是后代元素</p>
                <p>我是p元素的兄弟元素</p>
        </div>
        <span>我是span的兄弟元素</span>
        <span>兄弟二号</span>
        <span>兄弟三号</span>
    </div>
</body>
```

### 2.4属性选择器

```html
    <title>属性选择器</title>
    <style>
        /*  [属性值] 选择含有指定属性的元素\
            [属性名 = 属性值] 选择含有指定属性和属性值得元素
            [属性名 ^= 属性值] 选择属性值以指定值开头的元素 
            [属性名 $= 属性值] 选择属性值以指定值结尾的元素 
            [属性名 *= 属性值] 选择属性中含有某值的元素 */ 
            [title]{
                color: red;
            }
            [title = one]{
                color: pink;
            }
            [title ^= one]{
                color: green;
            }
            [title$=one]{
                color: blue;
            }
            [title*=one]{
                color: orange;
            }
    </style>
</head>
<body>
    <ul>
        <li title="one">你好</li>
        <li title="onetwo">我好</li>
        <li title="threeone">他好</li>
        <li>她好</li>
        <li>它好</li>
    </ul>
</body>
```

### 2.5伪类选择器

```html
   <title>伪类选择器</title>
    <style>
        /* 伪类选择器: 
            将所有同级的元素 同一等级的 分配在一起进行排序 选择 
                    1.:first-child  第一个  所有被选中的子类中的第一个
                    2.:last-child  最后一个
                    3.:nth-child() 随意选择
                    nth-child(odd)选择奇数序号的  nth-child(even)选择偶数序号的
       		将所有相同元素进行排列
                    :first-of-type  
                    :last-of-type
                    :nth-of-type()*/
            /* ul>li:first-child{
                color: pink;
            } */
            /* ul li:nth-child(even){
                color: red;
            } */
            ul li:nth-of-type(2){
                color: red;
            }
    </style>
</head>
<body>
    <ul>
        <span>你好呀</span>
        <li>one</li>
        <li>two</li>
        <li>three</li>
        <li>four</li>
        <li>five</li>
    </ul>
</body>
```

### 2.6选择器的权重

```html
   <style>
        /* 样式冲突：
                发生样式冲突时,就可以根据选择器的权重决定
            选择器的权重:
                    内联样式        1,0,0,0
                    id选择器        0,1,0,0
                    类选择器        0,0,1,0
                    元素选择器      0,0,0,1
                    通配选择器      0,0,0,0
                    继承选择器      没有优先级
            比较优先级的时候,需要将所有的选择器的优先级进行相加计算,最后优先级最高的最先显示
            选择器累加不会超过他的上一级选择器的权重,优先级相同的选择器,优先使用靠下的选择器.
         */
         *{
             font-size: 20px;
         }
         div{
             font-size: 18px;
         }
         .box{
             font-size: 15px;
         }
         #box{
             font-size: 5px;
         }
    </style>
</head>
<body>
    <div id="box" class="box" style="font-size: 1px;">这是一个div盒子</div>
</body>
```



## 3.超链接伪类(表示一个状态)

```html
    <title>超链接的伪类</title>
    <style>
        /* 超链接的伪类
                :link 用来表示没有访问过的连接
                :visited 表示访问过的链接
        		常用的状态
                :hover 表示鼠标移入的状态
                :active 表示鼠标点击 
        */
        a:link{
            color: white;
        }
        a:visited{
            color: black;
        }
        a:hover{
            color: red;
        }
        a:active{
            color: darkblue;
        }
    </style>
</head>
<body>
    <a href="https://www.baidu.com">百度</a>

    <a href="http://www.bilibili.com">B站</a>
</body>
```

## 4伪元素(表示特殊的位置)

```html
    <title>伪元素</title>
    <style>
        /*伪元素,表示页面中一些特殊的并不真实存在的元素(特殊位置)
            伪元素的开头 使用:: 开头
        
            ::first-letter 第一个字母
            ::first-line 第一行
            ::selection  选中的内容
        	在开始或者元素结束的时候,添加内容.
            ::before 元素的开始
            ::after 元素的结束
                    before 和 after 必须结合content属性来使用
            */
        p::first-letter{
            font-size: 25px;
        }
        p::first-line{
            background-color: black;
        }
        p::selection{
            color: red;
        }
        
        p::before{
            content:'开始';
            color:pink;
        }
        p::after{
            content: '结束';
            color: pink;
        }
    </style>
</head>
<body>
    <p>
文本分析
分析电视剧情或者流行曲歌词，研究这些媒体如何塑造角色、演员或歌手的形象，以及这些作品所隐藏的某些对人对事的看法
报章的标题的用字、字体、大小、版面放置、占用的空间等
分析广告的用色、配乐、选角、桥段
内容分析
把杂志内的广告分类，或数算一本杂志内有多少个纤体广告
分析某一电视剧中所特定一类人物，如大学生、律师、领综援人士、有色人种的遭遇
    </p>
</body>
```

## 5.像素(PX) 和 颜色(RGB)

```html
<style>
            /* 
            长度单位:
                像素 (px)
                百分比 
                    根据父元素的大小改变而改变
                em
                    em相当于元素的字体大小
                    1em = 1 font-size 
                    em会根据自定义font-size的改变而改变
                rem
                    rem相对于根元素(html)的字体大小来计算 
            */
             /* 颜色单位
                RGB值:
                    通过三种不同的颜色进行组合,
                    red  green blue 
                    每种颜色的范围为 (0-255) (0-100%)
                RGBA值:
                    比RGB多一个透明度 1表示不完全透明  0 表示完全透明 
                十六进制的RGB":
                    语法:#红色绿色蓝色  范围(00 -ff)
                    如果颜色两两重位可以重写*/
        .box{
            width: 200px;
            height: 200px;
            border: 1px solid;
            background-color: rgb(150, 230, 123);
        }   
        .box .box2{
            width: 10rem;
            height: 10rem;
            border: 1px solid;
            font-size: 18px;
            background-color: rgba(123, 34, 34, 0.5);
        } 
        .box2 div{
            width: 100px;
            height: 100px;
            border: 1px solid;
            background-color: #bfa;
        }
       
    </style>
</head>
<body>
    <div class="box">
        我是一个实验盒子
        <div class="box2">
            <div> </div>
        </div>
    </div>
</body>
```

## 6.盒子模型

### 6.1块级元素模型

```html
 <title>盒子模型</title>
    <style>
        /* 盒子模型 (box model)
            CSS将页面的所有元素都设置为一个矩形盒子
            每个盒子的组成部分为:
                                内容区(content)
                                 边框(border)
                                内边距(padding)
                                外边距(margin)
        */
        /* 
        内容区(content) 元素中的所有子元素和文本内容都在内容区
            内容区的大小被 width 和 height 设置
        边框(border) 边框属于盒子的边缘 
            边框的大小会影响盒子的大小  设置边框的三个属性
            border-style 四个属性:solid dashed dotted double 默认为null
            border-color 默认为black
            border-width 默认为3px
        内边距(padding) 内容区距离边框的距离
            内边距的设置会影响到盒子的大小
            背景颜色会延伸到内边距上(一般内容区为子元素)   
        一个盒子的可见框大小,由内容区 内边框 和 边框 一起决定的
        外边距(margin) 外边距不会影响盒子的大小,但会影响盒子的位置
            margin-top 上边距,盒子向下移动  (负值向上移动)
            margin-right 右边距,默认不会产生影响
            margin-left 左边距,盒子向右移动 (负值向左移动)
            margin-bottom 下边距,下边元素回向下移动
         */
         .box{
             width:200px;
             height: 200px;
             border: 10px red dashed;
             padding: 100px;
             background-color: #baf;
         }
         .box1{
             width: 100%;
             height: 100%;
             background-color: #abf;
             
         }
    </style>
</head>
<body>
    <div class="box">
        <div class="box1"></div>
    </div>
</body>
```

### 6.2行内元素模型

```html
 <style>
        /* 行内元素盒子模型
                行内元素不支持设置宽高
                行内元素可以设置padding border margin ,垂直方向不会影响页面布局
         */
         /* 
            display 用来设置元素显示的类型
                可选值: inline 设置为行内元素
                        block  设置为块元素
                        inline-block 行内块元素 可以设置宽高也不会独占一行
                        table 设置为一个表格
                        none  元素在页面中不显示(常用)
     		overflow hidden 截取 
            visiblity 用来设置元素的显示和隐藏
                可选值: visible  默认值 显示
                        hidden  隐藏
                区别: visiblity 隐藏之后还是会占位  但是 display 不会进行占位(常用)
         */
         span{
             /* padding: 100px; */
             display: block;//块级元素
             width: 100px;
             height: 100px;
             background-color: orange;
             /* visibility: hidden; */
         }
         div{
             width: 100px;
             height: 100px;
             background-color: red;
         }
    </style>
</head>
<body>
    <span>这是一个span盒子</span>
    <div>这是一个div盒子</div>
</body>
```

### 6.2盒子的尺寸

```html
        <style>
            /* 默认情况下,盒子可见框的大小由内容区 边框 内边距 共同确定
                box-sizing 用来设置盒子尺寸的计算方式 (设置 width 和 height 的作用)
                可选值: content-box 默认值 宽度和高度来设置盒子的内容区
                        border-box 宽度和高度 用来设置整个盒子的可见框大小
                                width和height 指的是内容区 内边距 边框的总大小
            总结:box-sizing :border-box; 搭配padding使用无敌
            使用padding值一般改变内容区的位置(文本 和 子元素) */
            .box{
                width: 100px;
                height: 100px;
                border: 10px solid red;
                padding: 10px;
                box-sizing: border-box;
            }
        </style>
    </title>
</head>
<body>
    <div class="box"></div>
</body>
```

### 6.3盒子的轮廓  阴影 和 圆角 

```html
       <style>
            /* 
                轮廓outline 来设置元素的轮廓线,用法和border一模一样
                轮廓和边框不同在于: 轮廓不会影响到可见框的大小
                
                阴影box-shadow 设置盒子的阴影,阴影也不会影响布局页面
                第一个值 代表水平方向  正值向左移动 负值向右移动
                第二个值 代表垂直方向  正值向下移动 负值向上移动
                第三个值 代表阴影的模糊半径(值越大 阴影的边缘就越模糊)
                第四个值 代表阴影的颜色

                盒子的圆角
                border-radius:用来设置圆角 圆角设置的圆的半径大小

            */
            .box{
                width: 300px;
                height: 200px;
                background: pink;
                /* border: 10px solid red; */
                /* outline: 10px solid red; */
                box-shadow:20px 20px 100px #666;
                border-radius: 20px;
            }
        </style>
 
</head>
<body>
    <div class="box"></div>
    <span>border会影响可见框 但是 outline不会</span>
</body>
```

### 7.字体样式

```html
   <title>字体样式</title>
    <style>

        /*  字体的样式:
        字体的颜色  color
        字体的大小: font-size 
                相关的单位 em相当于当前元素的一个font-size 
                 rem相当于根元素的font-size
        字体的粗细: font-weight 
                属性值:nomal bold 
        字体的样式: font-style
                属性值:nomal italic倾斜
        字体的样式: font-family 
                属性为 : serif 衬现字体
                        sans-serif 非衬线字体
                        monospace 等宽字体
        注:可以指定多个字体,字体优先使用第一个    
        自定义引入字体
        @font-face{
            font-family:"自定义字体名字";
           src:url("地址");
        }  
        
        简写属性 :  font:加粗 倾斜 字体框大小/行高 字体样式;
        */
        @font-face {
            font-family:"myfont" ;
            src: url(../font/思源黑体-繁体/SourceHanSansKR-Medium.ttf);
        }
        /* p{
        color: red;
        font-size: 15px;
        font-weight: bold;
        font-style: italic;
        font-family: 'myfont';
        } */
        p{
            font:italic bold 30px/2 'myfont';
            color: red;
        }

    </style>
</head>
<body>
    <p>
        花有重开日,人物再少年! 
    </p>
</body>
```

### 8.行高

```html
 <title>行高 line height</title>
    <style>
        /* 行高
            行高指的是文字所占有的实际高度
            line-height来设置行高
                属性值: 1.可以设置大小(px em)2.整数会是字体的指定倍数
            设置行间距
               行间距 = 行高 - 字体大小
            字体框 
                字体框就是字体存在的格子,设置的font-size其实是字体框的高度
            行高会在字体框上下均匀分配
            注:可以将 行高 和 盒子高度相同 来 实现居中效果
         */
         div{
             height: 100px;
             border: 1px solid red;
             font-size: 25px;
             line-height: 100px;
         }
    </style>
</head>
<body>
    <div>
        你 好 世 界!
    </div>
</body>
```

### 9.修饰文本

```html
<title>水平对齐 和 垂直对齐</title>
    <style>
        /* 水平对齐的属性
            text-align
                    left(默认值)
                    right
                    center
                    justify(两边对齐) 
            垂直对齐属性
            vertical-align
                    baseline(默认) 基线对齐
                    middle居中
                    top
                    bottom
            垂直属性以基线为标准,进行对齐.

            文本修饰 text-decoraton 
               可选值:none 什么都没有 
                    underline下划线 
                    line-through 删除线
                    overline 上划线
            省略文本操作: 
                white-space 属性值
        
                normal 正常值
                nowrap  不换行
                pre     保留空白
            overflow:hidden;

            text-overflow:ellipsis;
        */
        div{
            border: 1px solid red;
            font: 15px/2 '新宋体';
            text-align: left;
        }
        span{
            border: 1px solid blue;
            font-size: 10px;
            vertical-align: middle;
        }
        p{
            width: 100px;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
        }
    </style>
</head>
<body>
    <div> 你好 <span>呀</span> 世界 </div>

    <p>文本文件是一种计算机文件，
        它是一种典型的顺序文件，
        其文件的逻辑结构又属于流式文件。
        特别的是，文本文件是指以ASCII码方式(也称文本方式)存储的文件，更确切地说，英文、数字等字符存储的是ASCII码，而汉字存储的是机内码。文本文件中除了存储文件有效 </p>
</body>
```

### 10.背景图片样式

```html
 <title>背景图片</title>
    <style>
        /*
        background-image 设置背景图片
            background-image:url("图片地址");
            背景图片的显示会根据元素的大小进行展示
        background-repeat 背景的重复方式
            设置属性:repeat 默认值 重复
                    no-repeat 不重复
                    repeat-x 沿着x轴重复
                    repeat-y 沿着y轴重复
        background-position设置背景图片的位置
            水平方向 和 垂直方向  相当于 margin的left 和 top
            具体方位的设置 
                top bottom left right center
                必须存在两个值
        
        背景图片样式的简写  
            可以设置所有的值并且没有顺序可言
            background-position/background-size
            origin 需要在  clip 前面
        background  颜色 图片地址 重复属性 位置选择   
        
        background-clip背景范围
            可选值:border-box 默认值 背景会出现在边框的下边
                padding-box 背景只会出现在内容区和内边距区域
                content-box 背景只会出现在内容区
        backgroud-origin 背景图片的偏移计算的圆点
                padding-box 默认值 background-position 默认根据内边距开始计算
                content-box  从内容区计算
                border-box  边框区开始计算
        background-size设置背景图片的大小
                第一个表示宽度
                第二个表示高度
                cover 图片比例不变,将元素铺满
                contain 图片比例不变,将元素中完整显示  一般这个比较常用
      background:color url(地址) no-repeat 位置/背景图片的大小(contain);
        */  
        .box{
            width: 800px;
            height: 800px;
            border: 1px solid red;
            /* background-image: url("../Image/京东导航栏图片/地址.png");
            background-repeat: no-repeat;
            background-position: right top; */
            background: url("../Image/京东导航栏图片/地址.png") no-repeat right top;
            background-size: 100px 100px;
        }
    </style>
</head>
<body>
    <div class="box"></div>
</body>
```

### 11.表格样式

```html
 <title>表格样式</title>
    <style>
        /* 
            border-spacing 指定边框之间的距离
        
            border-collapse collapse 设置边框的合并 常用
        	
            默认情况下所有元素在 td 都是居中的 
        */
        table{
            width: 400px;
            border: 1px solid;
            border-collapse: collapse;
            text-align: left;
            /* border-spacing: 10px; */
        }
        td{
            border: 1px solid;
        }
        table tr:nth-child(odd){
            background: #baf;
        }
    </style>
</head>
<body>
    <table>
        <tr>
            <td>1</td>
            <td>2</td>
            <td>3</td>
            <td>4</td>
        </tr>
    </table>
</body>
```



## 浮动

### 1.浮动的基本概念

```html
    <style>
        /* 
            浮动可以让元素向其父元素的左右移动
                float属性来设置浮动
                float: left none(默认值) right
            注:元素设置浮动后,水平等式将不再成立;
                完全从文档流脱离,不占据文档流的位置;
                所以下面文档流的元素会自动向上移动
            特点:1.浮动元素完全脱离文档流,不占据文档流中的位置
                2.子元素不会从父元素中移出去
                3.不会超过它前面的浮动元素   
                4.浮动元素前面是一个没有浮动的元素,则浮动元素无法向上 
             */
        .box1 {
            width: 200px;
            height: 200px;
            background: red;
        }

        .box2 {
            width: 200px;
            height: 200px;
            background: blue;
        }

        .box3 {
            width: 200px;
            height: 200px;
            background: #baf;
        }
    </style>
</head>

<body>
    <div class="box1"></div>
    <div class="box2"></div>
    <div class="box3"></div>
</body>
```

### 2.浮动脱离文档流(脱离文档流特点)

```html
   <style>
        /* 元素设置浮动后会从文档流中脱离
          脱离文档流的特点:
                块元素: 1.块元素不独占一行
                        2.块元素的宽高被默认内容撑开
                行内元素:
                        行内元素脱离文档流变成块元素,特点和块元素一样
                脱离文档流就不区分块元素和行内元素了
        所有文档流元素特定:可以设置宽高 不再独占一行
       文本内容不会出现在脱离文档流的元素下(图片的环绕)
         */
         span{
             float: left;
             width: 100px;
             height: 200px;
             background: red;
         }
         div{
             color: red;
         }
    </style>
</head>
<body>
    <span> 行内元素 脱离文档流 </span>
    <div>块级元素 脱离文档流</div>
</body>
```

 ### 3.盒子塌陷问题(BFC解决/伪类元素::after)

```html
 <style>
        /* 高度塌陷问题: 在浮动布局中,父元素的高度默认被子元素撑开;
                        当子元素浮动后,其完全脱离文档流,子元素从文档流脱离
                        将无法撑起父元素的高度,导致父元素高度缺失
            父元素高度缺失以后,其下的元素会自动上移动,导致页面的混乱

            BFC(Block Formatting Context) 块级元素格式化环境
            为元素开启BFC模式,元素会变成一个独立的布局区域
            开启BFC的特点:
                    1.不会被浮动所覆盖
                    2.子元素和父元素外边距不会重合
                    3.包含浮动的子元素
            开启BFC的方式为: 给父元素添加 overflow:hidden; 清除浮动
         */
         *{
             margin: 0;
             padding: 0;
         }
         .farther{
            overflow: hidden;
            border: 10px solid green;
         }
         .son{
             width: 100px;
             height: 100px;
             border: 1px solid red;
             /* 由于父元素没有 宽度 子元素浮动 父元素的高度就塌陷了*/
             float: left; 
         }
    </style>
</head>
<body>
  <div class="farther">
      <div class="son"></div>
  </div>
</body>
```

### 4.清除浮动对其他元素的影响

```html
  <style>
        /* 由于box1的浮动,导致box3位置上移
            box3受到box1的影响,位置发生了改变
         
        要想让某个元素不受到其他浮动元素的影响,使用清除 浮动属性
             clear : 属性值为   left right  both
        原理:设置清除浮动后,浏览器会自动为元素添加一个外边距,使其位置不受到其他元素影响. 
         */
        div{
            width: 100px;
            height: 100px;
            border: 1px solid red;
            }
        .box1{
            border: 1px solid green;
            float: left;
        }
        .box2{
            width: 1000px;
            height: 400px;
            border: 1px solid blue;
            float: right;
        }
        .box3{
            clear: both;
        }
    </style>
</head>
<body>
    <div class="box1">1</div>
    <div class="box2">2</div>
    <div class="box3">3</div>
</body>
```

### 5.伪类元素after清除浮动

```html
    <title>通过伪元素after解决盒子塌陷问题</title>
    <style>
        .father{
            border: 10px solid red;
            /* 开启BFC 给父级元素添加 overflow:hidden; */
            /* overflow: hidden; */
        }
        .son{
            height: 100px;
            width: 100px;
            background-color: green;
            float: left;
        }
        /* 通过伪元素 after 是一个行内元素 所有需要变成块级元素 清除浮动 */
        .father::after{
            content:'';
            clear: both;
            display: block;
        }
    </style>
</head>
<body>
    <div class="father">
    <div class="son"></div>
    </div>
</body>
```

### 6.高度塌陷和外边距重叠问题的解决

```html
    <title>clearfix类解决高度塌陷和外边距重叠问题</title>
    <style>
        /* 解决高度塌陷 和 外边距重叠问题  通过添加 clearfix类来解决
        添加伪类元素  before(外) 和 after
        */
    .clearfix::before,.clearfix::after{
        content: '';
        display: table;
        clear: both;
    }
    .box1{
            border: 1px solid red;
        }
    .box2{
        float: left;
        height: 100px;
        width: 100px;
        background-color: red;
    }
    .box3{
        height: 50px;
        width: 50px;
        background-color: seagreen;
        margin-top: 50px;
    }
    </style>
</head>
<body>
    <div class="box1 clearfix">
        <div class="box2">
            <div class="box3"></div>
        </div>
    </div>
</body>
```

## 定位

### 1.定位的简介

```html
 <title>定位的简介</title>
    <style>
        /* 定位的简介:
                定位是一种高级的布局手段
                通过定位可以将元素摆放在任意位置 
                通过position来设置定位:
                        static 默认属性
                        relative 相对定位 
                        absolute 绝对定位
                        fixed 固定定位
                        sticky 粘滞定位
                通过偏移量(offset)来改变位置:
                        top  bottom  left right(上下左右)
        */
        div {
            width: 100px;
            height: 100px;
        }
        .one {
            background-color: red;
        }

        .two {
            background-color: seashell;
            margin-left: 100px;
            margin-top: -100px;
        }

        .three {
            background-color: sienna;
            margin-top: 100px;
        }
    </style>
</head>

<body>
    <div class="one">1</div>
    <div class="two">2</div>
    <div class="three">3</div>
</body>
```

### 2.相对定位

```html
<title>相对定位</title>
    <style>
        /* 相对定位的特点:
                1.开启相对定位,不设置偏移量不会发生任何改变
                2.相对定位参照于自身元素在文档流的位置
                3.提升自身元素的层级
                4.不会脱离文档流
                5.不会改变元素的性质
         */
         div {
            width: 100px;
            height: 100px;
        }
        .one {
            background-color: red;
        }

        .two {
            background-color: seashell;
            position: relative;
            left:80px;
            top: -100px;
        }

        .three {
            background-color: sienna;
          }
    </style>
</head>
<body>
    <div class="one">1</div>
    <div class="two">2</div>
    <div class="three">3</div>
</body>
```

### 3.绝对定位

```html
   <title>绝对定位</title>
    <style>
        /* 绝对定位的特点
                1.开启绝对定位没有偏移量元素位置不会发生改变
                2.元素会从文档流脱离(和浮动效果相同)
                3.改变元素的性质,行内元素变成块元素
                4.元素提升一个层级
                5.绝对定位相对于包含块来定位
                绝对定位的包含块:
                        包含块就是距离它最近的开启了相对定位的祖先元素
                        如果所有祖先元素都没有开启相对定位,则根元素就是包含块
         */
         div {
            width: 100px;
            height: 100px;
        }
        .one {
            background-color: red;
        }

        .two {
            background-color: seashell;
        }

        .three {
            background-color: sienna;
         
          }
        .four{
            width: 500px;
            height: 500px;
            background-color: burlywood;
            position: relative;
        }
        .five{
            background-color: cyan;
            position: absolute;
            top: 100px;
            left: 100px;
        }
    </style>
</head>
<body>
     <div class="one">1</div>
     <div class="two">2</div>
     <div class="three">3</div>
     <div class="four">
         <div class="five">5</div>
     </div>
</body>
```

### 4.固定定位

```html
  <title>固定定位</title>
    <style>
        /* 固定定位
                特殊的相对定位,唯一不相同的是固定定位永远参照浏览器的可视窗口进行
                定位,固定定位的元素不会伴随着网页的滚动条滚动
        */
        body{
            height: 2000px;
        }
        div{
            width: 20px;
            height: 150px;
            background: cyan;
            position: fixed;
            top: 50%;
            right:0;
        }
    </style>
</head>
<body>
    <div>
        这是固定定位
    </div>
</body>
```

### 5.粘滞定位

```html
  <style>
        /* 粘滞定位
                类似于相对定位 相对于包含块来定位
                兼容性差 */
    </style>
```

### 6.绝对元素的位置布局

```html
   <title>绝对定位元素的布局</title>
    <style>
        /*  水平布局等式
                margin-left+border-left+padding-left+width+padding-right+border-right+margin-right 
            当我们开启绝对定位的时候,水平方向布局等式就会添加left 和 right 值
                    发生过度约束:如果这里面的所有值不存在auto就会自动调节right值让等式满足
                    存在auto值则调节auto的值
            可设置auto的值:
                        margin width left right
            垂直布局等式
                margin-top+border-top+padding-top+height+padding-bottom+border-bottom+margin-bottom
            开启绝对定位的时候,水平方向布局等式就会添加top 和 bottom 值
            本质内容和水平布局等式相同
                 */
        .father{
            height: 500px;
            width: 500px;
            background-color: cyan;
            position: relative;
        }
        .son{
            position: absolute;
            height: 50px;
            width: 50px;
            background-color: darkgreen;
            margin: auto;
            left: 0;
            top:0;
            bottom: 0;
            right: 0;
        }
    </style>
</head>
<body>
    <div class="father">
        <div class="son"></div>
    </div>
</body>
```

### 7.元素的层级

```html
  <title>元素的层级</title>
    <!-- 
     添加了定位的元素都可以通过z-index来指定元素的层级
     z-index需要整数作为参数,整数越大层级越高 ,元素的层级越高越优先显示
     如果元素的层级相同,优先显示靠下的元素
     
     注:祖先元素的优先级再高也不会盖住后代元素
    -->
    <style>
        div {
            width: 200px;
            height: 200px;
            position: absolute;
        }

        .box1 {
            background-color: red;
            z-index: 1;
        }
        .box2{
            background-color: seagreen;
            top: 100px;
            left: 100px;
        }
        .box3{
            background-color: sienna;
            top: 200px;
            left: 200px;
            z-index: 1;
        }
        .box4{
            background-color: silver;
            z-index: 3;
        }
    </style>
</head>

<body>
    <div class="box1">1</div>
    <div class="box2">2</div>
    <div class="box3">3</div>
    <div class="box4">4</div>
</body>
```

 ## 过渡/动画

### 1.过渡效果

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>过渡的简介</title>
    <style>
        /* 过渡 transition 
            过渡的属性 :
            property 设置过渡的属性值 一般使用all

            duration 设置过渡的时间值

            timig-function  指定过渡的直行方式
            ease 默认值 匀速
            ease-in 加速运动
            ease-out 减速运动
            ease-in-out 先加速,后减速运动
            
            delay  过渡效果的延迟 等待时间

            通过简介属性直接写  两个时间中第一个为持续时间值,第二位延迟时间.
        */ 
        div{
            width: 100px;
            height: 100px;
        }
        .father{
            width: 800px;
            height: 800px;
            background: silver;
        }
        .box1{
            background: red;
            margin-bottom: 100px;
            transition: all 1s ease-in; 
        }
        .father:hover .box1{
            margin-left: 700px;
        }
        .box2{
            background: orange;
            transition: all 1s ease-out;
        }
        .father:hover .box2{
            margin-left: 700px;
        }
    </style>
</head>
<body>
    <div class="father">
        <div class="box1"></div>
        <div class="box2"></div>
    </div>
</body>
</html>
```

### 2.动画效果

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>动画效果</title>
    <style>
        /* 设置动画的属性: 关键帧
               @keyframes 关键帧 名字可以自己命名
               from 开始  to 结束
        创建关键帧 之后才可以使用animation属性
        
        animation  属性值
        name  默认设置的name
        duration 持续的时间
        delay 延迟的时间
        timing-function  执行的方式  steps() 分为几个阶段
        
        iteration-count 执行的次数 infinite(无数次)
        play-state 执行的状态  running(默认值)   paused  
        fill-mode 动画的填充模式 none 默认值 回到原来的位置  frowards 停止在结束的位置  alternate 停止在结束位置
        */
        @keyframes test{
            from{
                background-position:0 0;
            }
            to{
                background-position:-300px 0;
            }
        }
    </style>
</head>
<body>
        <div class="box"></div>
</body> 
</html>
```

### 3旋转(roate) 平移  缩放

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>旋转  平移  缩放 </title>
    <style>
        /* transform 用来设置元素的变形效果 
            不会影响网页的布局  和  其他元素的位置

            属性值 : 平移 translateX() 沿着X轴进行平移   正值为 向左平 移
                        translateY() 沿着Y轴进行平移    正值为 向下平移
                        translateZ() 沿着Z轴进行平移    近大远小  设置了视距
            Z轴平移,调整元素在Z轴的位置  调整元素和人眼的距离,
            默认情况下网页是不支持透明效果   必须设置网页的视距  (perspective)  

            属性值 : 旋转  rotateX()   rotateY()  rotateZ()  设置视距
            
            属性值 : 缩放  scaleX()  scaleY()    scaleZ()    水平 垂直  Z轴方向缩放

            变形的圆点 :  transform-origin  改变圆点的位置  
         */
         .box{
             width: 200px;
             height: 200px;
             background: #bfa;
             margin: 100px auto;
             transform: translateX(100px) translateY(50px) rotateZ(45deg) scale(1.2); 
         }
    </style>
</head>
<body>
    <div class="box"></div>
</body>
</html>
```

## 弹性

### 1.弹性简介

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>弹性盒子</title>
    <style>
        /* flex (弹性盒子 伸缩盒子) 
            Css中的布局方式,它主要代替浮动来进行布局
            flex可以让元素具有弹性,让元素根据页面的大小进行改变而改变
            
            弹性容器:  
                要是让盒子变成弹性容器 需要display 来设置属性
        
        		flex 设置为块级弹性容器
        
                inline-flex 设置为行内弹性容器

            弹性元素:
                弹性容器的直接子元素是弹性元素
                弹性元素可以同时是弹性元素

            flex-direction 指定 容器中的弹性元素的排列方式
            可选值 :  row 默认值  向左排列
                    row-reverse  反向排列 
                    column  自上向下
                    column-reverse  自下向上  

            弹性元素的属性
            flex-grow : 父元素的剩余空间会进行比列排列  增长属性 
            flex-shrink  : 伸缩属性   
        */
        *{
            margin:0;
            padding: 0;
        }
        ul{
            width: 800px;
            border: 1px solid red;
            list-style: none;
            /* 将父盒子直接设置为弹性盒子   */
            display: flex;
            /* overflow: hidden; */
            /* 设置弹性元素的排列方式 */
            flex-direction: row-reverse;
        }
        li{
            /* 添加浮动由于父元素没有添加高度,所以盒子塌陷需要overflow:hidden;来清除浮动*/
           /* float:left; */
           width: 200px;
           height: 100px;
           font: 20px/5 "宋体";
           text-align: center;
        }
        li:nth-child(1){
            background: red;
        }
        li:nth-child(2){
            background: pink;
        }
        li:nth-child(3){
            background: green;
        }
    </style>
</head>
<body>
    <ul>
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
</body>
</html>
```

### 2弹性容器的属性

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>弹性容器的属性</title>
    <style>
        /* 
            flex-direction  弹性元素的排列方式  row  row-reverse  column  column-reverse

            flex-wrap      弹性元素是否换行  wrap自动换行  nowrap 不换行 wrap-reverse反方向换行

            flex-flow  可以将两个属性进行简写   没有顺序的要求

            justify-content  如何分配  主轴上的空白位置   属性值 
                         flex-start沿着主轴(X轴)开始排列 flex-end 沿着主轴结束排列 center中间排列 space-around 分配在两侧 space-between
            
            align-items 辅轴上的空白元素如何对齐
                         flex-start沿着辅轴(Y轴)开始对齐 flex-end 沿着辅轴结束对齐 center 居中对齐
        */
    </style>
</head>
<body>
    
</body>
</html>
```

### 3.弹性元素的属性

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>弹性盒子</title>
    <style>
        /*
            弹性元素的属性
            flex-grow : 父元素的剩余空间会进行比列排列  增长属性 
            flex-shrink  : 伸缩属性   
            
            flex-basis  元素的基础长度 

            简写属性 flex 可以设置 弹性元素的三个样式  1 0 auto 按照增长 伸缩  基础元素 正常  

            order 决定 弹性元素的排列顺序  
        */
        *{
            margin:0;
            padding: 0;
        }
        ul{
            width: 800px;
            border: 1px solid red;
            list-style: none;
            /* 将盒子直接设置为弹性盒子   */
            display: flex;
            /* overflow: hidden; */
            /* 设置弹性元素的排列方式 */
            flex-direction: row-reverse;
        }
        li{
            /* 添加浮动由于父元素没有添加高度,所以盒子塌陷需要overflow:hidden;来清除浮动*/
           /* float:left; */
           width: 200px;
           height: 100px;
           font: 20px/5 "宋体";
           text-align: center;
        }
        li:nth-child(1){
            background: red;
        }
        li:nth-child(2){
            background: pink;
        }
        li:nth-child(3){
            background: green;
        }
    </style>
</head>
<body>
    <ul>
        <li>1</li>
        <li>2</li>
        <li>3</li>
    </ul>
</body>
</html>
```

### 4.总结弹性盒子的用法

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>弹性容器的用法</title>
    <style>
        *{
            margin: 0;
            padding: 0;
        }
        /* 给父元素添加上 display flex 属性 将其改变成弹性容器
           一般而言 对于移动端的布局  ul 不需要设置宽度 高度  子元素自动撑开
        */
        ul{
            list-style: none;
            /* 设置为弹性容器  弹性容器的属性  
                1.进行换行 和 排列方式
                2.如何分配空白位置 常用分配在两侧 space-around
            */
            display: flex;

            flex-flow: nowrap row;

            justify-content: space-around;
        }
        li{
            /* 弹性元素
                三个属性 变成一个  flex  增长 缩短  自动分配
            */
            width: 100px;
            height: 20px;
            border: 1px solid red;
            flex: auto;
        }
    </style>
</head>
<body>
    <ul>
        <li>1</li>
        <li>2</li>
        <li>3</li>
        <li>4</li>
    </ul>
</body>
</html>
```

