![image](/img/img1.gif?raw=true)

#### 前言：这是一段很简单的代码，很多细节并没有去做最优处理，目的是为了让大家大致了解VUE双向绑定的过程，然后再细读源码会有不同的收获。

#### 文章分三个步骤就完成双向绑定的代码，每个部分又细分小步骤去引导你的思路。第一遍看不懂没关系，重复学习没有弄懂的步骤，最后肯定能够掌握的。

### 知识准备

[Object.defineProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

[发布-订阅模式](https://blog.csdn.net/qq_34707744/article/details/79170052)

![image](https://note.youdao.com/yws/public/resource/69eb94b76a10da5388ddcc0ce3590fd4/xmlnote/7451CB37AA384590AE88F557A5EFE151/2839)

代码实现步骤：
1. 根据上图实现一个整体的架构（包括MVVM类、或者VUE类、Watcher类），这里用到一个订阅发布者模式。

1. 把模型的数据绑定到视图（实现MVVM中的M到V）

1. 由文本事件触发更新模型中的数据，同时也更新相对应的视图（实现V-M）

先建一个html文档，代码如下

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>手写一个VUE的双向绑定</title>
    <script>
        //SCRIPT 1
        //这里实现双向绑定

    </script>
</head>
<body>
   <div id="app">
       <h1>响应式</h1>
       <div>
           <div v-text="myText"></div>
           <input type="text" v-model="myText">
       </div>
   </div> 
   <script>
        //SCRIPT 2
       const app = new Vue({
           el: "#app",
           data: {
               myText: "我是响应的"
           }
       })
   </script>
</body>
</html>
```
我们最终的目的是通过==SCRIPT 1==代码，能够使指令v-modal 、v-text实现和VUE一样的数据双向绑定。

## 一、构建一个整体的架构

1.  把==SCRIPT 1==代码拎出来

```
<script>
    //SCRIPT 1
    //这里实现双向绑定
    
    //发布者
    class Vue {}
    //订阅者
    class Watcher{}
</script>
```
2. ==SCRIPT 2== 中实例化Vue时有参数传入，所以class Vue要加入构造函数constructor。

```
<script>
    //发布者
    class Vue {
        constructor(options) {
            this.options = options;
            this.$data = options.data;//获取数据
            this.$el = document.querySelector(options.el);//获取id=app的对象
    
        }
    }
    //订阅者
    class Watcher{}

</script>
```

3.给class Vue增加Observer、Compile两个方法，并初始化

```
<script>
   //发布者
        class Vue {
            constructor(options) {
                this.options = options;
                this.$data = options.data; //获取数据
                this.$el = document.querySelector(options.el); //获取id=app的对象
                //调用
                this.Observer(this.$data); //劫持数据需要传递数据给她
                this.Compile(this.$el); //解析指令的需要DOM对象

            }
            //劫持数据
            Observer(data) {}
            //解析指令
            Compile(el) {}
        }
    //订阅者
    class Watcher{}
</script>
```
## 二、把模型数据绑定到视图
1. 在Compile中解析指令。通过传入的DOM循环他的子元素，找到v-modal、v-text两个指令。

```
<script>
    //发布者
    class Vue {
        constructor(options) {
            this.options = options;
            this.$data = options.data; //获取数据
            this.$el = document.querySelector(options.el); //获取id=app的对象
            //调用
            this.Observer(this.$data); //劫持数据需要传递数据给她
            this.Compile(this.$el); //解析指令的需要DOM对象

        }
    //劫持数据
    Observer(data) {

    }
    //解析指令
    Compile(el) {
        let nodes = el.children; // 获取APP下面的子元素
        for(let i = 0;i < nodes.length;i++) { //循环2次 h1 div
            let node = nodes[i]; //当前元素

            if(node.hasAttribute("v-text")) {//查找v-text

            }

            
            if(node.hasAttribute("v-model")) {}
        }
    }
}
    //订阅者
    class Watcher{}
    
</script>
```

2. 利用递归继续查找子元素的子元素中是否有指令

```
<script>
    //SCRIPT 1
    //这里实现双向绑定
    //发布者
    class Vue {
        constructor(options) {
            this.options = options;
            this.$data = options.data; //获取数据
            this.$el = document.querySelector(options.el); //获取id=app的对象
            //调用
            this.Observer(this.$data); //劫持数据需要传递数据给她
            this.Compile(this.$el); //解析指令的需要DOM对象

        }
        //劫持数据
        Observer(data) {

        }
        //解析指令
        Compile(el) {
            let nodes = el.children; // 获取APP下面的子元素
            for(let i = 0;i < nodes.length;i++) { //循环2次 h1 div
                let node = nodes[i]; //当前元素
                
                if(node.children.length) {//如果当前元素含有子元素，就递归
                    this.Compile(node);
                }
                if(node.hasAttribute("v-text")) {//查找v-text

                }

                
                if(node.hasAttribute("v-model")) {}
            }
        }
    }
    //订阅者
    class Watcher{}

</script>
```

3. 找到指令后想成为订阅者。先在发布者里（Vue 的constructor）准备一个容器（_directives），保存订阅者.在Oberver里初始化_directives

```
<script>
    //SCRIPT 1
    //这里实现双向绑定
    //发布者
    class Vue {
        constructor(options) {
            this.options = options;
            this.$data = options.data; //获取数据
            this.$el = document.querySelector(options.el); //获取id=app的对象
            
            this._directives = {};//存放订阅者
            //{myText:[订阅者1 ,订阅者2],myTBox:[订阅者1]}
            
            //调用
            this.Observer(this.$data); //劫持数据需要传递数据给她
            this.Compile(this.$el); //解析指令的需要DOM对象
    
        }
        //劫持数据
        Observer(data) {
            //   更新视图 是局部更新
            for(let key in data) {
                    this._directives[key] = [];
            }
        }
        //解析指令
        Compile(el) {
            let nodes = el.children; // 获取APP下面的子元素
            for(let i = 0;i < nodes.length;i++) { //循环2次 h1 div
                let node = nodes[i]; //当前元素
    
                if(node.children.length) {//如果当前元素含有子元素，就递归
                    this.Compile(node);
                }
                if(node.hasAttribute("v-text")) {//查找v-text
    
                }
    
                
                if(node.hasAttribute("v-model")) {}
            }
        }
    }
    //订阅者
    class Watcher{}


</script>
```

4. 在Compile中把指令对应的元素存放到_directives中

```
<script>
        //SCRIPT 1
        //这里实现双向绑定
        //发布者
        class Vue {
            constructor(options) {
                this.options = options;
                this.$data = options.data; //获取数据
                this.$el = document.querySelector(options.el); //获取id=app的对象
                this._directives = {};//存放订阅者
                //{myText:[订阅者1 ,订阅者2],myTBox:[订阅者1]}

                //调用
                this.Observer(this.$data); //劫持数据需要传递数据给她
                this.Compile(this.$el); //解析指令的需要DOM对象

            }
            //劫持数据
            Observer(data) {
                //   更新视图 是局部更新
                for(let key in data) {
                    this._directives[key] = [];
                }

            }
            //解析指令
            Compile(el) {
                let nodes = el.children; // 获取APP下面的子元素
                for(let i = 0;i < nodes.length;i++) { //循环2次 h1 div
                    let node = nodes[i]; //当前元素
                    if(node.children.length) {//如果当前元素含有子元素，就递归
                        this.Compile(node);
                    }
                    
                    if(node.hasAttribute("v-text")) { //查找v-text
                        //获取属性里面的值，对号进入相对应的订阅者数组
                        let attVal = node.getAttribute('v-text'); //获取元素
                        this._directives[attVal].push(new Watcher());//放订阅者的实例对象

                    }

                    
                    if(node.hasAttribute("v-model")) {}
                }
            }
        }
        //订阅者
        class Watcher{}

</script>
```

5. 订阅者（Watcher）里实现update更新操作。把传入的数据显示到html上

```
<script>
    //SCRIPT 1
    //这里实现双向绑定
    //发布者
    class Vue {...
       
    //订阅者
    class Watcher{
        constructor() {

        }
        update() {
            div对象[innerHTML] = vue对象.data["myText"];
            input对象[value] = vue对象.data["myText"];
        }
    }

</script>

```

6.根据5 中得到Watcher 需要的参数：node(div 对象)、attrValue属性(myText)、this(vue对象)、div属性（innerHTML）

```
  this._directives[attVal].push(new Watcher(node,attVal,this,"innerHTML"));//放订阅者的实例对象
```


```
<script>
    //SCRIPT 1
    //这里实现双向绑定
    //发布者
    class Vue {...
       
    //订阅者
    class Watcher{
        constructor(el,vm,mySelf,attr) {
                this.el = el;
                this.vm = vm;
                this.mySelf = mySelf;
                this.attr = attr;
                this.update()//初始化数据
        }
        update() {
            //div对象[innerHTML] = vue对象.data["myText"];
            //input对象[value] = vue对象.data["myText"];
            this.el[this.attr] = this.mySelf.$data[this.vm];
        }
    }

</script>
```
这个时候可以把手上的代码运行一下，

```
 <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>手写一个VUE的双向绑定</title>
    <script>
        //SCRIPT 1
        //这里实现双向绑定
        //发布者
        class Vue {
            constructor(options) {
                this.options = options;
                this.$data = options.data; //获取数据
                this.$el = document.querySelector(options.el); //获取id=app的对象
                this._directives = {};//存放订阅者
                //{myText:[订阅者1 ,订阅者2],myTBox:[订阅者1]}

                //调用
                this.Observer(this.$data); //劫持数据需要传递数据给她
                this.Compile(this.$el); //解析指令的需要DOM对象

            }
            //劫持数据
            Observer(data) {
                //   更新视图 是局部更新
                for(let key in data) {
                    this._directives[key] = [];
                }

            }
            //解析指令
            Compile(el) {
                let nodes = el.children; // 获取APP下面的子元素
                for(let i = 0;i < nodes.length;i++) { //循环2次 h1 div
                    let node = nodes[i]; //当前元素

                    if(node.children.length) {//如果当前元素含有子元素，就递归
                        this.Compile(node);
                    }
                    if(node.hasAttribute("v-text")) { //查找v-text
                        //获取属性里面的值，对号进入相对应的订阅者数组
                        let attVal = node.getAttribute('v-text'); //获取元素
                        this._directives[attVal].push(new Watcher(node,attVal,this,"innerHTML"));//放订阅者的实例对象

                    }

                    
                    if(node.hasAttribute("v-model")) {}
                }
            }
        }
        //订阅者
        class Watcher{
            constructor(el,vm,mySelf,attr) {
                this.el = el;
                this.vm = vm;
                this.mySelf = mySelf;
                this.attr = attr;
                this.update()//初始化数据

            }
            update() {
                //div对象[innerHTML] = vue对象.data["myText"];
                //input对象[value] = vue对象.data["myText"];
                this.el[this.attr] = this.mySelf.$data[this.vm];
            }
        }

    </script>
</head>
<body>
   <div id="app">
       <h1>响应式</h1>
       <div>
           <div v-text="myText"></div>
           <input type="text" v-model="myText">
       </div>
   </div> 
   <script>
       const app = new Vue({
           el: "#app",
           data: {
               myText: "我是响应的"
           }
       })
    </script>
</body>
</html>
```

运行结果如下

![image](https://note.youdao.com/yws/public/resource/69eb94b76a10da5388ddcc0ce3590fd4/xmlnote/FDAD018F23CE483C99C7E684834DD6FE/3008)

我们已经将data.myText的值输出到了页面上

7.补齐Compile中对v-modal的解析后，页面上输入框也会显示相应的值

```
 if(node.hasAttribute("v-model")) {
    let attVal = node.getAttribute('v-model'); //获取元素
    this._directives[attVal].push(new Watcher(node,attVal,this,"value"));//放订阅者的实例对象
 }
```
## 三、更新视图同步到模型  --再更新视图
1. 在文本框输入后能触发更新模型.所以在文本框输入时添加一个监听事件，利用闭包返回一个函数

```
<script>
    //SCRIPT 1
    //这里实现双向绑定
    //发布者
    class Vue {
        ...
        //解析指令
        Compile(el) {
            let nodes = el.children; // 获取APP下面的子元素
            for(let i = 0;i < nodes.length;i++) { //循环2次 h1 div
                let node = nodes[i]; //当前元素

                ...
                
                if(node.hasAttribute("v-model")) {
                    let attVal = node.getAttribute('v-model'); //获取元素
                    this._directives[attVal].push(new Watcher(node,attVal,this,"value"));//放订阅者的实例对象
                    node.addEventListener("input", (function(){
                        return function() {
                            console.log("我变了")
                        }
                    })());
                }
            }
        }
    }
       
    //订阅者
    class Watcher{...

</script>
```
此时运行代码，可以发现我们input里的值发生改变时，控制台就会打印“我变了”.

2. 同步到模型

```
<script>
    //SCRIPT 1
    //这里实现双向绑定
    //发布者
    class Vue {
        ...
        //解析指令
        Compile(el) {
            let nodes = el.children; // 获取APP下面的子元素
            for(let i = 0;i < nodes.length;i++) { //循环2次 h1 div
                let node = nodes[i]; //当前元素

                ...
                
                if(node.hasAttribute("v-model")) {
                    let attVal = node.getAttribute('v-model'); //获取元素
                    this._directives[attVal].push(new Watcher(node,attVal,this,"value"));//放订阅者的实例对象
                    let _this = this;
                    node.addEventListener("input", (function(){
                            return function() {
                                //更新视图同步到模型
                                _this.$data[attVal] = node.value
                            }
                    })());
                }
            }
        }
    }
       
    //订阅者
    class Watcher{...

</script>
```
3. 在Oberver里监听(Object.defineProperty)数据模型(data)，一旦模型的新值和订阅者的旧值不想等，新值赋给订阅者。
```
<script>
    //SCRIPT 1
    //这里实现双向绑定
    //发布者
    class Vue {
        constructor(options) {
            this.options = options;
            this.$data = options.data; //获取数据 data是数据模型
            this.$el = document.querySelector(options.el); //获取id=app的对象
            this._directives = {};//存放订阅者
            //{myText:[订阅者1 ,订阅者2],myTBox:[订阅者1]}

            //调用
            this.Observer(this.$data); //劫持数据需要传递数据给她
            this.Compile(this.$el); //解析指令的需要DOM对象

        }
        //劫持数据
        Observer(data) {
            //   更新视图 是局部更新
            for(let key in data) {
                this._directives[key] = [];

                let Val = data[key];//当前的值
                let _this = this._directives[key];

                Object.defineProperty(this.$data,key,{
                    get: function (){
                        return Val;
                    },
                    set: function(newVal){
                        if(newVal !== Val){
                            Val = newVal;
                        }
                    }
                });
            }

        }
        //解析指令
        Compile(el) {
          ...
        }
    }
       
    //订阅者
    class Watcher{...

</script>
```
4. 利用Watcher的实例对象的update方法实现更新视图
```
<script>
    //SCRIPT 1
    //这里实现双向绑定
    //发布者
    class Vue {
        constructor(options) {
            this.options = options;
            this.$data = options.data; //获取数据 data是数据模型
            this.$el = document.querySelector(options.el); //获取id=app的对象
            this._directives = {};//存放订阅者
            //{myText:[订阅者1 ,订阅者2],myTBox:[订阅者1]}

            //调用
            this.Observer(this.$data); //劫持数据需要传递数据给她
            this.Compile(this.$el); //解析指令的需要DOM对象

        }
        //劫持数据
        Observer(data) {
            //   更新视图 是局部更新
            for(let key in data) {
                this._directives[key] = [];

                let Val = data[key];//当前的值
                let _this = this._directives[key];

                Object.defineProperty(this.$data,key,{
                    get: function (){
                        return Val;
                    },
                    set: function(newVal){
                        if(newVal !== Val){
                            Val = newVal;
                            _this.forEach(watcher => {//遍历订阅者的实例
                                    watcher.update();//更新视图
                            });
                        }
                    }
                });
            }

        }
        //解析指令
        Compile(el) {
          ...
        }
    }
       
    //订阅者
    class Watcher{...

</script>
```
此时我们已经完成了所有的代码。

将代码运行到浏览器后就能看到效果了
[代码地址](https://note.youdao.com/)

![image](/img/img2.gif?raw=true)

参考资料来自
[CSDN](https://blog.csdn.net/qq_34707744/article/details/79170052)
[网易云课堂](https://study.163.com/course/courseLearn.htm?courseId=1209072812#/learn/live?lessonId=1278614266&courseId=1209072812)