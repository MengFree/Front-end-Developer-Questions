# Front-end-Developer-Questions
### 1. 深度克隆数据
```javascript
function clone(obj) {
    var val;
    var type=Object.prototype.toString.call(obj).slice(8,-1)
    switch (type) {
        case "String":
            val = obj + '';
            break;
        case "Number":
            val = obj - 0;
            break;
        case "Object":
            val = {};
            for (var key in obj) {
                if (obj.hasOwnProperty(key)) {
                    var p = obj[key];
                    val[key] = clone(p);
                }
            }
            break;
        case "Array":
            val = [];
            for (var i = 0; i < obj.length; i++) {
                var tmp = obj[i];
                val.push(clone(tmp));
            }
            break;
        default:
            val = obj
            break;
    }
    return val;
};
// 判断是数组
function isArray(obj) {
    // Object.prototype.toString.call(obj).slice(8,-1)==='Array'
    return Object.prototype.toString.call(obj) === '[object Array]';
};
```
深度拷贝和浅度拷贝区别<br>
    简单来说，浅复制只复制一层对象的属性，而深复制则递归复制了所有层级。因为浅复制只会将对象的各个属性进行依次复制，并不会进行递归复制，而 JavaScript 存储对象都是存地址的，所以浅复制会导致 obj1.arr 和 obj2.arr 指向同一块内存地址，导致修改源对象造成拷贝对象也会改变<br>
    而深复制则不同，它不仅将原对象的各个属性逐个复制出去，而且将原对象各个属性所包含的对象也依次采用深复制的方法递归复制到新对象上。这就不会存在上面 obj 和 obj2 的 arr 属性指向同一个对象的问题。

    javascript中存储对象都是存地址的，所以浅拷贝是都指向同一块内存区块，而深拷贝则是另外开辟了一块区域。
```javascript
var obj1={
    a:1,
    arr:[1,2,3] 
}
var obj2=clone(obj1)

// 浅拷贝
obj1.arr[0]=5;//造成 boj2.arr[0];
//深度拷贝则不会

```
jQuery.extend第一个参数可以是布尔值，用来设置是否深度拷贝的:
```javascript
jQuery.extend(true, { a : { a : "a" } }, { a : { b : "b" } } );
jQuery.extend( { a : { a : "a" } }, { a : { b : "b" } } );
```
`es6`中有`Object.assign()`这个方法
```javascript
// Cloning an object
var obj = { a: 1 };
var copy = Object.assign({}, obj);
console.log(copy); // { a: 1 }
```
```javascript
// Merging objects
var o1 = { a: 1 };
var o2 = { b: 2 };
var o3 = { c: 3 };

var obj = Object.assign(o1, o2, o3);
console.log(obj); // { a: 1, b: 2, c: 3 }
console.log(o1);  // { a: 1, b: 2, c: 3 }, target object itself is changed.
```
`Object.assign()`只是一级属性复制，比浅拷贝多深拷贝了一层而已。用的时候，还是要注意这个问题的。
```javascript
const defaultOpt = {
    title: {
        text: 'hello world',
        subtext: 'It\'s my world.'
    }
};

const opt = Object.assign({}, defaultOpt, {
    title: {
        subtext: 'Yes, your world.'
    }
});

console.log(opt);
// opt={
//     title: {
//         subtext: 'Yes, your world.'
//     }
// }
```

----------------------------
### 2. 请用Promise改写以下程序
有以下两个方法（callback 参数均为 ‘err,res’）<br>
`model.fetchUsers(skip, limit, callback);`<br>
`service.log(message, callback);`<br>

把以上两个方法用Promise改写，然后改下以下程序，解决回调嵌套问题。
```javascript
model.fetchUsers(0, 100, function(err, users) {
    if (err) {
        console.error('fetchUsers error:', err);
    } else {
        service.log('fetchUsers', function(err) {
            if (err) {
                console.error('log error:', err);
            } else {
                console.log('done');
            }
        });
    }
})
```
>改写
```javascript
var fetchUsers = function(skip, limit) {
    return new Promise((reslove, reject) => {
        model.fetchUsers(skip, limit, function(err, res) {
            if (err) return reject(err);
            reslove(res)
        })
    });
}

var log = function(message) {
    return new Promise((reslove, reject) => {
        service.log(message, function(err, res) {
            if (err) return reject(err);
            reslove(res)
        })
    });
}
fetchUsers().then(res => {
    return log('fetchUsers');
}, err => {
    console.log('fetchUsers error:', err)
}).then(res => {
    return log('done');
}, err => {
    console.log('log error:', err)
})
```
----------------------------
### 3. 数组去重
>方法一：
```javascript
Array.prototype.unit = function() {
    let me = this;
    let tmp = [],
        lice = [];
    var k = 0;
    for (var i = 0, len = me.length; i < len; i++) {
        let m = me[i];
        if (tmp.indexOf(m) == -1) {
            tmp.push(m)
        } else {
            // lice.push(me.splice(i, 1)[0]);
            lice = lice.concat(me.splice(i, 1));
            i--;
            len--;
        }
    }
    return lice;
}
```
>方法二：(无法区分字符串`string`与数字`number`)
```javascript
Array.prototype.$unit = function() {
    let me = this;
    let tmp = {},lice=[];
    var k = 0;
    for (var i = 0, len = me.length; i < len; i++) {
        let m = me[i];
        if (tmp[m] === undefined) {
            tmp[m] = true;//以val做键值不能区分string、number 例如  1 和 '1'
        } else {
            // lice.push(me.splice(i, 1)[0]);
            lice = lice.concat(me.splice(i, 1));
            i--;
            len--;
        }
    }
    return lice;
}
```
----------------------------
### 4. jQuery事件绑定方法bind、 live、delegate和on的区别
*  `bind()` <br>
    会绑定事件类型和处理函数到DOM element上
* `live()` <br>
    绑定相应的事件到你所选择的元素的根元素上，即是document元素上
* `delegate()` <br>
    .delegate()有点像.live(),不同于.live()的地方在于，它不会把所有的event全部绑定到document,而是由你决定把它放在哪儿。而和.live()相同的地方在于都是用event delegation.
* `on()` <br>
    其实.bind(), .live(), .delegate()都是通过.on()来实现的，.unbind(), .die(), .undelegate(),也是一样的都是通过.off()来实现的
>总结:<br>
- 用`.bind()`的代价是非常大的，它会把相同的一个事件处理程序hook到所有匹配的DOM元素上；<br>
- 不要再用`.live()`了，它已经不再被推荐了，而且还有许多问题；
- `.delegate()`会提供很好的方法来提高效率，同时我们可以添加一事件处理方法到动态添加的元素上；<br>
- 我们可以用`.on()`来代替上述的3种方法；
----------------------------
### 5. 你所了解到的Web攻击技术

* （1）XSS（Cross-Site Scripting，跨站脚本攻击）：指通过存在安全漏洞的Web网站注册用户的浏览器内运行非法的HTML标签或者JavaScript进行的一种攻击。 
* （2）SQL注入攻击 
* （3）CSRF（Cross-Site Request Forgeries，跨站点请求伪造）：指攻击者通过设置好的陷阱，强制对已完成的认证用户进行非预期的个人信息或设定信息等某些状态更新。
----------------------------
### 6. 清除浮动有哪些方式？比较好的方式是哪一种？
    （1）父级div定义height。
    （2）结尾处加空div标签clear:both。
    （3）父级div定义伪类:after和zoom。(较常用)
    （4）父级div定义overflow:hidden。
    （5）父级div定义overflow:auto。
    （6）父级div也浮动，需要定义宽度。
    （7）父级div定义display:table。
    （8）结尾处加br标签clear:both。
----------------------------
### 7. DOM怎样添加、移除、移动、复制、创建和查找节点
    // 创建新节点
    createDocumentFragment() //创建一个DOM片段
    createElement() //创建一个具体的元素
    createTextNode() //创建一个文本节点
    // 添加、移除、替换、插入
    appendChild()
    removeChild()
    replaceChild()
    insertBefore() //在已有的子节点前插入一个新的子节点
    // 查找
    getElementsByTagName() //通过标签名称
    getElementsByName() //通过元素的Name属性的值(IE容错能力较强，会得到一个数组，其中包括id等于name值的)
    getElementById() //通过元素Id，唯一性
----------------------------
### 8. call() 和 apply() 的区别和作用
    apply()函数有两个参数：第一个参数是上下文，第二个参数是参数组成的数组。如果上下文是null，则使用全局对象代替。
    如：function.apply(this,[1,2,3]);
    call()的第一个参数是上下文，后续是实例传入的参数序列。
    如：function.call(this,1,2,3);
----------------------------
### 9.性能优化的方法
    （1） 减少http请求次数：CSS Sprites, JS、CSS源码压缩、图片大小控制合适；网页Gzip，CDN托管，data缓存 ，图片服务器。
    （2） 前端模板 JS+数据，减少由于HTML标签导致的带宽浪费，前端用变量保存AJAX请求结果，每次操作本地变量，不用请求，减少请求次数
    （3） 用innerHTML代替DOM操作，减少DOM操作次数，优化javascript性能。
    （4） 当需要设置的样式很多时设置className而不是直接操作style。
    （5） 少用全局变量、缓存DOM节点查找的结果。减少IO读取操作。
    （6） 避免使用CSS Expression（css表达式)又称Dynamic properties(动态属性)。
    （7） 图片预加载，将样式表放在顶部，将脚本放在底部 加上时间戳。
------------
### 10. 已知有字符串foo="get-element-by-id",写一个function将其转化成驼峰表示法"getElementById"
```javascript
function combo(msg){
    var arr = msg.split("-");
    var len = arr.length;    //将arr.length存储在一个局部变量可以提高for循环效率
    for(var i=1;i<len;i++){
        arr[i]=arr[i].charAt(0).toUpperCase()+arr[i].substr(1,arr[i].length-1);
    }
    msg=arr.join("");
    return msg;
}
```
--------------
### 11.原生JS的`window.onload`与Jquery的`$(document).ready``(function(){})`有什么不同
`window.onload()`方法是必须等到页面内包括图片的所有元素加载完毕后才能执行。<br>
`$(document).ready()`是DOM结构绘制完毕后就执行，不必等到加载完毕。 <br>
执行顺序->`$(document).ready()`->`window.onload()`　
>用原生JS实现Jq的ready方法
```javascript
function ready(fn){
    if(document.addEventListener) { //标准浏览器
        document.addEventListener('DOMContentLoaded', function() {
            //注销事件, 避免反复触发22222
            document.removeEventListener('DOMContentLoaded',arguments.callee, false);
            fn(); //执行函数
        }, false);
    }else if(document.attachEvent) { //IE
        document.attachEvent('onreadystatechange', function() {
            if(document.readyState == 'complete') {
                document.detachEvent('onreadystatechange', arguments.callee);
                fn();        //函数执行
            }
        });
    }
};
```
---------------
### 12. JS如何实现面向对象和继承机制
>构造器继承
```javascript
function Super(a){
    this.a = a;
    this.func = function(){};
}
function Sub(a){
    Super.call(this,a);
}
var obj = new Sub();
```
>原型继承
```javascript
function Super(a){
    this.a = a;
    this.func = function(){};
}
function Sub(){
}
Sub.prototype = new Super();
Sub.prototype.constructor = Sub;
var obj = new Sub();
```
>混合继承
```javascript
function Super(a){
    this.a = a;
    this.func = function(){};
}
function Sub(a){
    Super.call(this,a);
}
Sub.prototype = new Super();
Sub.prototype.constructor = Sub;
var obj = new Sub();
```
>寄生继承
```javascript
function Super(a){
    this.a = a;
    this.func = function(){};
}
Super.prototype.talk = function(){console.log('talk');}
function Sub(a){
    Super.call(this,a);
}
Sub.prototype = Object.create(Super.prototype);
Sub.prototype.constructor = Sub;
var obj = new Sub();
```
---------------
## 13.移动设备忽略将页面中的数字识别为电话号码的方法
```html
<meta name="format-detection" content="telephone=no"> 
<meta name="format-detection" content="email=no">  
<meta name="format-detection" content="adress=no"> 
<!--telephone=禁止数字拨号 email=不识别邮箱，点击之后不自动发送 adress=跳转至地图-->  
<meta name="format-detection" content="telephone=no,email=no,adress=no">  
```
---------------
### 14. 事件委托
    利用事件冒泡的原理，使自己的所触发的事件，让它的父元素代替执行！
    这里用父级ul做事件处理，当li被点击时，由于冒泡原理，事件就会冒泡到ul上，因为ul上有点击事件，所以事件就会触发
    Event对象提供了一个属性叫target，可以返回事件的目标节点，我们成为事件源，也就是说，target就可以表示为当前的事件操作的dom，但是不是真正操作dom，当然，这个是有兼容性的，标准浏览器用ev.target，IE浏览器用event.srcElement，此时只是获取了当前节点的位置，并不知道是什么节点名称，这里我们用nodeName来获取具体是什么标签名，这个返回的是一个大写的
```javascript
window.onload = function(){
    var oUl = document.getElementById("ul1");
    oUl.onclick = function(ev){
        var ev = ev || window.event;
        var target = ev.target || ev.srcElement;
        if(target.nodeName.toLowerCase() == 'li'){
            alert(123);
            alert(target.innerHTML);
        }
    }
}
```
---------------
### 15. 原生js封装一个添加监听事件
```javascript
functionaddEvent(obj,event,fn){
        //现代：addEventLister 不用加on
        //ie：touchEvent 需要加on
        if (window.addEventListener){
                obj.addEventListener(event,fn);
        }else{
                obj.attachEvent("on"+event, fn);
        }
}
```
---------------
### 16. JSONP 的工作原理
利用`<script>`标签没有跨域限制的“漏洞”来达到与第三方通讯的目的。当需要通讯时，通过js创建一个`<script>`元素，地址指向第三方的API网址，形如：<br>
`<script src="http://www.example.net/api?param1=1&param2=2"></script>`<br>并提供一个回调函数来接收数据（函数名可约定，或通过地址参数传递）。
第三方产生的响应为json数据的包装（故称之为jsonp，即json padding），形如：<br>`callback({"name":"hax","gender":"Male"})`<br>这样浏览器会调用callback函数，并传递解析后json对象作为参数。本站脚本可在callback函数里处理所传入的数据。
### 17. 如何阻止事件冒泡和默认事件

    阻止冒泡:
    现代浏览器:e.stopPropagation
    低版本浏览器:e.cancelBubble=true
    阻止默认事件:
    现代浏览器:e.preventDefult()
    低版本浏览器:return false
---------------
### 18. GET/POST的区别
>GET请求:
>>请求的数据会附加在URL之后，以?分割URL和传输数据，多个参数用&连接。URL的编码格式采用的是ASCII编码，而不是uniclde，即是说所有的非ASCII字符都要编码之后再传输。

>POST请求：
>>POST请求会把请求的数据放置在HTTP请求包的包体中。

>关于传输数据的大小
>>在HTTP规范中，没有对URL的长度和传输的数据大小进行限制。但是在实际开发过程中，对于GET，特定的浏览器和服务器对URL的长度有限制。因此，在使用GET请求时，传输数据会受到URL长度的限制。对于POST，由于不是URL传值，理论上是不会受限制的，但是实际上各个服务器会规定对POST提交数据大小进行限制，Apache、IIS都有各自的配置。
---------------

### 19. Jquery命名冲突解决的方案
$只是jquery的一个别名而已，假如我们需要使用 jquery 之外的另一 js 库，我们可以通过调用 $.noConflict() 向该库返回控制权。
```javascript
// 方法一
jQuery.noConflict();                  //将变量$的控制权让渡给prototype.js
jQuery(function(){                    //使用jQuery
    jQuery("p").click(function(){
        alert( jQuery(this).text() );
    });
});
// 方法二
var $j = jQuery.noConflict();        //自定义一个比较短快捷方式
$j(function(){                       //使用jQuery
    $j("p").click(function(){
        alert( $j(this).text() );
    });
});
// 方法三
jQuery.noConflict();                //将变量$的控制权让渡给prototype.js
jQuery(function($){                 //使用jQuery
    $("p").click(function(){        //继续使用 $ 方法
        alert( $(this).text() );
    });
});
// 方法四
jQuery.noConflict();                //将变量$的控制权让渡给prototype.js
(function($){                       //定义匿名函数并设置形参为$
    $(function(){                   //匿名函数内部的$均为jQuery
        $("p").click(function(){    //继续使用 $ 方法
            alert($(this).text());
        });
    });
})(jQuery);                         //执行匿名函数且传递实参jQuery
```
----

### 20. methods和computed和watch的联系和区别
*methods和computed*<br>
```js
var vm = new Vue({
        el: '#example',
        data: {
        message: 'Hello'
    },
    /*--------computed--------*/
    computed: {
        reversedMessage: function () {
            return this.message.split('').reverse().join('')
        }
    },
    /*-------methods---------*/
    methods: {
        reversedMessage: function () {
            return this.message.split('').reverse().join('')
        }
    }
})

```
    computed计算属性是基于它们的依赖进行缓存的。计算属性computed只有在它的相关依赖发生改变时才会重新求值。这就意味着只要message 还没有发生改变，多次访问 reversedMessage 计算属性会立即返回之前的计算结果，而不必再次执行函数。而对于method ，只要发生重新渲染，method 调用总会执行该函数。
        总之：数据量大，需要缓存的时候用computed；每次确实需要重新加载，不需要缓存时用methods
*computed和watch*<br>
```js
var vm = new Vue({
    el: '#demo',
    data: {
        firstName: 'Foo',
        lastName: 'Bar',
        fullName: 'Foo Bar'
    },
    /*-------watch---------*/
    watch: {
        firstName: function (val) {
            this.fullName = val + ' ' + this.lastName
        },
        lastName: function (val) {
            this.fullName = this.firstName + ' ' + val
        }
    },
    /*-------computed---------*/
    computed: {
        fullName: function () {
            return this.firstName + ' ' + this.lastName
        }
    }
})
```
    计算属性(computed)只有在它的相关依赖发生改变时才会重新求值。这就意味着只要 lastName和firstName都没有发生改变，多次访问 fullName计算属性会立即返回之前的计算结果，而不必再次执行函数。
    而观察watch是观察一个特定的值，当该值变化时执行特定的函数。例如分页组件中，我们可以检测页码执行获取数据的函数。

>computed在数据未发生变化时，优先读取缓存。computed 计算属性只有在相关的数据发生变化时才会改变要计算的属性，当相关数据没有变化是，它会读取缓存。而不必像 motheds方法 和 watch 方法是的每次都去执行函数。
-----

### 21. Vue组件间通信
* 父组件向子组件通信

        方法一：props 
            使用props，父组件可以使用props向子组件传递数据。
        方法二：使用$children 
            使用$children可以在父组件中访问子组件。
* 子组件向父组件通信
    
        方法一:使用vue事件
            父组件向子组件传递事件方法，子组件通过$emit触发事件，回调给父组件。
        方法二： 通过修改父组件传递的props来修改父组件数据
            这种方法只能在父组件传递一个引用变量时可以使用，字面变量无法达到相应效果。因为饮用变量最终无论是父组件中的数据还是子组件得到的props中的数据都是指向同一块内存地址，所以修改了子组件中props的数据即修改了父组件的数据。
            但是并不推荐这么做，并不建议直接修改props的值，如果数据是用于显示修改的，在实际开发中我经常会将其放入data中，在需要回传给父组件的时候再用事件回传数据。这样做保持了组件独立以及解耦，不会因为使用同一份数据而导致数据流异常混乱，只通过特定的接口传递数据来达到修改数据的目的，而内部数据状态由专门的data负责管理。
        方法三：使用$parent
            使用$parent可以访问父组件的数据。
* props双向绑定

        通过 sync 双向绑定，属性变化会同步到所有组件，这也是最简单的实现方式，缺点是属性会比较多
* 非父子组件、兄弟组件之间的数据传递

        `$on`方法用来监听一个事件。
        `$emit`用来触发一个事件。

* 复杂的单页应用数据管理.

        当应用足够复杂情况下，请使用vuex进行数据管理。
-----
### 22. vue联动嵌套组件的实现
组件在它的模板内可以递归地调用自己，只有当它有 name 选项时才可以。 在官网这句话就是关键定义组件是一定要有name属性。
```html
<template>
  <li>
    <div @click='toggle'>
      <i v-if='isFolder' class="fa " :class="[open?'fa-folder-open':'fa-folder']"></i>
      <!--isFolder判断是否存在子级改变图标-->
      <i v-if='!isFolder' class="fa fa-file-text"></i> {{model.data.menuName}}
    </div>
    <ul v-show="open" v-if='isFolder'>
      <items v-for='cel in model.childTreeNode' :model='cel'></items>
    </ul>
  </li>
</template>
<script type="text/javascript">
  export default {
    name: 'items',
    props: ['model'],
    components: {},
    data() {
      return {
        open: false,
        isFolder: true
      }
    },
    computed: {
      isFolder: function() {
        return this.model.childTreeNode && this.model.childTreeNode.length
      }
    },
    methods: {
      toggle: function() {
        if(this.isFolder) {
          this.open = !this.open
        }
      },
    }
  }
</script>
```
