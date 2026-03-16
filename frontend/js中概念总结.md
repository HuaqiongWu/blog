---
title: JS中的概念总结
date: 2018-12-23
---

1、call、apply、bind的区别

  * call apply 和bind都是用来改变this的指向的
  * call 和 bind都是顺序传参数，apply是传数组
  * 利用call和apply会直接执行当前函数，bind不会  
2、reduce是es5中的内容
        
        ```bash
        var arr=“qweqrq"
        var info= arr.split('').reduce((a,b)=>
            (a[b]++ || (a[b]=1),a),{})
        console.log(info)
        ```

3、forEach、 map 、 reduce的区别
  * forEach 方法是将数组中的每一个值取出做一些程序员想让他们做的事情
  * map 方法 是将数组中的每一个值放入一个方法中做一些程序员想让他们做的事情后返回一个新的数组
  * reduce 方法 将数组中的每一个值与前面的被返回相加的总和(初试值为数组的第一个值或者initialValue



4、async await 以及promise  
promise解决的是回调场景中的状态处理问题，async/await解决的是回调嵌套的问题。  
（1）async是声明在回调环境函数  
（2）await是运行在等待回调结果过程中  
（3）promise是封装了回调操作的原子任务

5、require import & export & moudle.exports  
（1）node中用的是commonJs规范，export和require  
（2）为了在客户端使用，因此有了amd和cmd规范，二者都是异步加载，唯一不同的是依赖的模块执行的时机不同，amd是一边加载一边执行，cmd是加载完后才执行。  
（3）es6中的规范是export和import，但由于commonJs深入人心，因此还是有很多人使用commonJs规范。  
（4）require可以出现在代码中，import只能在代码的最顶层

6、JS异步的实现方式  
（1）回调 => 容易出现多重嵌套的问题，代码可读性差  
（2）事件监听（on和fire）  
（3）发布和订阅  
（4）promise对象  
（5）es6中的新加的generate
    
    
    ```bash
    Function *gen(x) {
      var y = yield x + 2;
      return y;
    }
    ```

generator函数函数名和function之间加*，所有的执行片段用yield分割。函数的返回值是一个iterator，每次调用next就返回当前当前yield片段的值。  
适合于返回值有多个状态的情况。  
（6）es7中的await和async

7、json和jsonp的区别：json是一种数据格式、jsonp是一种跨域的数据格式传输协议

8、浏览器同源策略限制不允许跨域。  
非跨域是指： 相同的域名、相同的协议、相同的端口。实现跨域的方式：  
（1）所有含有src属性的标签都不受同源策略限制，比如img，script，iframe，因此可以使用JSONP的方式，即客户端传递一个callback，服务端用callback包裹json数据，返回js片段，这样客户端就可以随意使用json数据，实现跨域了。比如统计点击率时经常用img标签。（jsonp的形式只支持get请求，不支持post请求。）  
（2）基于jquery的跨域，方式见下（9）  
(3) 利用document.domain进行跨域，document.domain可以进行赋值，但只可以设置为当前的域名或者基础域名。  
因此对于同一个基础域名的相同协议相同端口的跨域请求，可以通过设置document.domain来实现跨域。  
（4）window.postMessage()  
（5）cors：跨域资源共享是w3c的一个规范，这个主要在服务器的实现，前端发送请求没有区别。这和jsonp的区别是jsonp只支持get，cors可以支持post。

9、用jquery实现跨域的方式  
（1）直接将dataType设置为jsonp  
/_当前网址是localhost:3000_ /  
js代码
    
    
    ```bash
    $.ajax({
    type:"get",
    url:"http://localhost:3000/showAll",/*url写异域的请求地址*/
    dataType:"jsonp",/*加上datatype*/
    Jsonp: ‘cb', // 传递给请求处理程序或页面的，用以获得jsonp回调函数名的参数名(默认为:callback)
    jsonpCallback:”success",/*设置一个回调函数，名字随便取，和下面的函数里的名字相同就行*/
    success:function(data){
      console.log(data)
    }
    });
    /*而在异域服务器上，*/
    app.js
    app.get('/showAll',students.showAll);/*这和不跨域的写法相同*/
    /*在异域服务器的showAll函数里，*/
    var db = require("./database");
    exports.showAll = function(req,res){
    /**设置响应头允许ajax跨域访问**/
    res.setHeader("Access-Control-Allow-Origin","*");
    /*星号表示所有的异域请求都可以接受，*/
    res.setHeader("Access-Control-Allow-Methods","GET,POST");
    var con = db.getCon();
    con.query("select * from t_students",function(error,rows){
    if(error){
    console.log("数据库出错："+error);
    }else{
    /*注意这里，返回的就是jsonP的回调函数名+数据了*/
    res.send("cb("+JSON.stringify(r)+")");
    }
    });
    }
    ```

（2）直接调用$.getJson进行跨域请求数据，其内部原理是判断是否跨域、如果非跨域，直接用ajax方式发送请求，否则用script标签的方式
    
    
    ```bash
    jQuery.getJSON(url,data,success(data,status,function(data){
       Console.log(data)
    }))
    ```

// url是可以非同源的Url，data为一起传递到服务器的数据。  
10、xss和csrf的区别  
（1）xss通过输入框或者textare等，往页面上注入一段js，csrf是伪装成当前登录用户发送请求，甚至更改当前用户的信息。  
（2）csrf的一种实现方式是xss，csrf注重结果。  
（3）xss避免的一种方式是将html标签进行过滤或者对内容进行encode，另外cookie信息不要放用户名和密码，必要时需要加md5加密，csrf的方式是前后端交互加令牌。  
11、cookie & session  
（1）cookie不设置过期时间则保存在内存中，否则保存在硬盘中，下次打开会话，还能读取。cookie可以设置path和domain，且具有不可跨域性。  
（2）cookie是保存在浏览器上，session是保存在服务器中，因此cookie不是很安全，别人可以分析并且伪装。  
（3）cookie的value最大为4M，有些浏览器有限制每个站最多设置20个cookie。  
（4）session会占用服务器资源。  
12： ajax防止重复发送请求  
（1）点击后disable掉button  
（2）发送此请求时，abort掉上一个请求  
13：xhr（ajax）和fetch的区别  
（1）ajax为xhr，有abort状态(因为ajax是一个xmlhttpRequest，有个表示状态的readystatus，abort会将此值置为0，readystatus小于3，接收到的responseText未空)，fetch是基于promise实现的，只有resolve和reject状态，因而fetch没有abort状态。  
（2）fetch发送请求是不带cookie的，只有通过fetch（url, {credentials: ‘inlucde’ }）来加上  
（3）服务器返回400以及500状态是，fetch并不能reject，只有网络请求失败时会。  
14、ajax的readystatux状态  
（1） 0: 未初始化或者被abort到初始状态  
（2）1：已经初始化正在send请求  
（3）2： send已经完成  
（4）3： 已经解析了部分返回内容，未解析完  
（5）4： 相应内容已经解析完成  
15、函数防抖和节流  
（1） 防抖： 一段时间内，没有执行该函数，则一定时间后执行该函数，如果再次触发了事件，则从0开始计数，到达一定时间后再执行事件。
    
    
    ```bash
    function debounce(fn, wait) {
        var timer = null;
        return function() {
           var context = this;
          var args = arguments;
          if( timer != null) clearTimeout(timer)
          timer = setTimeout(() => {
             fn.apply(context, args)
          }, wait)
        }
    }
    function handle() {
       Console.log(‘handle')
    }
    window.addEventListener(’scroll’, debounce(handle, 1000))
    ```

（2）节流：持续触发事件，保证一段时间内，调用一次当前函数。( 且能保证一段时间内至少执行一次)
    
    
    ```bash
    function throttle(fn, wait) {
        var prev = Date.now();
        return function() {
            var now = Date.now();
            var context = this;
            var args = arguments;
            If (prev - now > delay) {
              fn(context, args);
              Prev = Date.now();
            }
        }
    }
    function handle() {
       console.log('handle')
    }
    window.addEventListener('scroll', throttle(handle, 1000))
    ```

16、vue和react的对比  
（1）vue和react都提倡组件化开发，都是数据驱动，都是通过props向组件传递参数，都有virtual dom  
（2）数据绑定： vue是双向数据绑定、react是单向  
（3）大规模协作的项目用react方便，小的项目用vue方便  
（2）开发风格：react用的是jsx，vue推荐使用的是 js css都在一个文件  
17、 gulp 和 webpack的区别  
18、原型和原型链  
（1）每个对象都有 **proto** 属性， 但只有函数对象才有 prototype属性  
（2）对象的__proto__属性指向他的构造函数 person1.**proto** = Person.prototype  
Person.**proto** = Function.prototype  
Person.prototype.**proto** = Object.prototype  
19、img的加载 - 懒加载  
（1）用img的src进行加载或者用css中background加载  
（2）重要的信息比如logo适合用img标签，如果图片特别大，建议用css中的background，这样不会影响整体框架在加载  
（3）动态的设置img的src信息即可  
20、函数闭包  
（1）闭包的例子：oldLog还保持旧值，而getLogNumber用新的值的原因可以从类定义，每个函数方法保存是单独的一份这点来区分。
    
    
    ```bash
    var gLogNumber, gIncreaseNumber, gSetNumber;
    function setupSomeGlobals() {
      // 局部变量num最后会保存在闭包中
      var num = 42;
      // 将一些对于函数的引用存储为全局变量
      gLogNumber = function() { console.log(num); }
      gIncreaseNumber = function() { num++; }
      gSetNumber = function(x) { num = x; }
    }
    setupSomeGlobals();
    gIncreaseNumber();
    gLogNumber(); // 43
    gSetNumber(5);
    gLogNumber(); // 5
    var oldLog = gLogNumber;
    setupSomeGlobals();
    gLogNumber(); // 42
    oldLog() // 5
    ```

（2）下例中result是闭包变量，i最后值为3，list[3]是undefined
    
    
    ```bash
    function buildList(list) {
        var result = [];
        for (var i = 0; i < list.length; i++) {
            var item = 'item' + i;
            result.push( function() {console.log(item + ' ' + list[i])} );
        }
        return result;
    }
    function testList() {
        var fnlist = buildList([1,2,3]);
        // 使用j是为了防止搞混---可以使用i
        for (var j = 0; j < fnlist.length; j++) {
            fnlist[j]();
        }
    }
     testList() //输出 "item2 undefined" 3 次
    ```

(3) 总结

  * 函数内使用了关键字function就创建了一个闭包
  * 闭包相当于一个副本，当函数退出时，所有的局部变量保存在其中
  * 闭包函数每次被调用，都会被创建一份新的局部变量存储。  
21、JS中的this  
（1）在普通函数中，非严格模式下，this指向window，否则指向undefined；  
（2）作为构造函数中的this指向实例对象  
（3）call和apply可以改变this中的指向  
（4）箭头函数在执行前已经定死了this的指向  
22、常见的几类前端安全性问题：


  1. 老生常谈的XSS（cross site script） 处理方式： 对输入(和URL参数)进行过滤，对输出进行编码。对cookie中的关键信息设置httponly属性，防止被js读取。
  2. 警惕iframe带来的风险：防止自己的网页被iframe（1）直接判断当前网页的host是否和window.top的host是否相同(2)后台配置：header(‘X-Frame-Options:Deny’); iframe别人的网页时： 需要iframe设置sandbox属性，禁止iframe脚本、ajax请求、form表单、限制origin等
  3. 别被点击劫持了（clickjacking）利用iframe先响应click事件，防御策略是后台配置：header(‘X-Frame-Options:Deny’);
  4. 错误的内容推断 (比如：上传的图片是用js片段文件)
  5. 防火防盗防猪队友：不安全的第三方依赖包（第三方插件本身存在漏洞）
  6. 用了HTTPS也可能掉坑里，比如用户直接输入域名，未指定为https方式，会被拦截者获取并模拟服务器与前端交互，处理方式是强制前端用https进行通信
  7. 本地存储数据泄露：本地存储不放敏感信息，只放一些公共信息
  8. 缺失静态资源完整性校验：从cdn中获取的js是被拦截处理过的，导致系统不能正常运行。处理方式有：对引用的script或者css加签名，浏览器拿到资源后，首先进行内容hash，值相同才会进行下一步处理，否则不执行。（ Subresource Integrity | 内容安全策略中添加所有的 Content-Security-Policy: require-sri-for script;）  
23：pwa 离线缓存  
（1）相当于在浏览器和服务器之间加了一层service worker， 用来控制离线缓存，可以将一些不经常更改的静态文件放到缓存中，提升用户体验。并且还可以生成一个原生的操作图标。但这个强制使用https，且浏览器兼容差。  
24:Post请求数据的格式：  
（1）默认：application/x-www-form-urlencoded  
（2）上传图片： multipart/form-data  
（3）application/json;charset=utf-8  
（4）xml  
25: 硬件加速：将渲染的过程交给GPU进行处理，提高渲染性能。  
26: let const var区别  
var支持预定义 以及重复定义 其他二者不可  
let有了块级作用域  
const变量不可以改 不可以不赋值


