# jQuery

### 简介
- js函数库，包含html元素选取/操作，css操作，html事件，js动画特效，DOM遍历修改，AJAX，和相关插件。
- 对主流浏览器做了兼容

### 语法
- 美元符号代表jQuery
- 选择符用于获取html元素
- 各类函数对获得元素执行操作
```
$(selector).action()
$(selector).action().action()//可以使用链式操作
```
- 文档就绪事件，防止文档加载完成之前运行jQuery代码，在html所有标签(DOM)都加载之后就会执行。
```
$(document).ready(function(){
    //编写jQuery代码
});
//简写
$(function(){ ... });
```

### 选择器
- 以$()开头，选择器编写与css选择器一致
```
$("p")//元素选择器
$("#test")//id选择器
$(".test")//类选择器
...//使用css语法即可
```

### 事件
- 与html的事件大多一致，一般分为触发（无参）和设置回调（参数为一个函数）两种用法。
```
//设置回调
$("p").click(function(){});
//触发点击
$("p").click();
```
- 常见包括ready/click/dblclick/mouseenter/mouseleave/mousedown/mouseup/hover/focus/blur
    - hover可传递2个参数，分别代表mouseenter/mousleave触发时回调。

### 动画效果
- 显隐：hide/show/toggle
    ```
    //speed可取'slow','fast','normal',毫米表示速度
    //callback即完成操作后的回调
    $(selector).hide(speed? ,callback?);
    $(selector).show(speed? ,callback?);
    $(selector).toggle(speed? ,callback?);
    ```
- 淡入淡出：fadeIn/fadeOut/fadeToggle/fadeTo
    ```
    //参数同上
    $(selector).fadeIn(speed?, callback?);
    $(selector).fadeOut(speed?, callback?);
    $(selector).fadeToggle(speed?, callback?);
    //speed和opacity为必需参数，opactiy代表目标透明值
    $(selector).fadeTo(speed, opacity,callback?);
    ```
- 滑动：slideDown/slideUp/slideToggle
    ```
    //参数同上
    $(selector).slideDown(speed?, callback?);
    $(selector).slideUp(speed?, callback?);
    $(selector).slideToggle(speed?, callback?);
    ```
- 动画：animate，多次调用animate会形成队列依次执行
    ```
    //params为必需参数，其中定义动画完成时的css属性，需使用驼峰命名
    //speed, callback同上
    $(selector).animate({params}, speed?, callback?);

    例：
    $("div").animate({
        left:'250px',
        height:'+=150px',//使用相对值
        width:'show'//使用预定义值show/hide/toggle
    });
    $("div").animate( ... )//将在上一个动画完成后执行
    ```
- 停止动画：stop，适用于所有动画效果
    ```
    //stopAll规定是否应该清除动画队列，默认false仅停止当前动画
    //goToEnd规定是否立即完成当前动画，默认false。立即完成会触发当前动画的回调，并显示当前动画的目标属性值
    $(selector).stop(stopAll? ,goToEnd?);
    ```

### HTML操作
- 获取内容和属性：text/html/val/attr
    ```
    //获取内容
    $(selector).text()//获取文本内容
    $(selector).html()//获取内容（包括html标记）
    $(selector).val()//获取表单字段的值
    $(selector).attr('href')//获取对应属性

    //设置内容
    $(selector).text('hello world!')
    $(selector).html('<b>hello world!</b>')
    $(selector).val('testValue')
    $(selector).attr('href', 'www.baidu.com')
    $(selector).attr({
        "href" : "http://www.runoob.com/jquery",
        "title" : "jQuery 教程"
    });//通过对象同时设置多个属性

    //通过函数设置内容
    $(selector).text(function(i,origText){
        //i代表元素列表（当匹配到多个元素）的下标
        //origText代表旧值
        return "旧文本: " + origText + " 新文本: Hello world! (index: " + i + ")"; 
    });
    //html/val/attr也支持通过函数设置内容
    ```
- 添加元素：append/prepend/after/before
    ```
    $(selector).append("追加文本");//在元素内部的结尾添加
    $(selector).prepend("追加文本");//在元素内部的开头添加
    $(selector).after("追加文本");//在元素之后添加
    $(selector).before("追加文本");//在元素之前添加
    
    //参数
    let txt1="<b>I </b>";//支持使用HTML创建元素
    let txt2=$("<i></i>").text("love ");//支持jQuery创建元素
    let txt3=document.createElement("a");//支持DOM创建元素
    $(selector).after(txt1,txt2,txt3);//添加内容支持变长参数
    ```
- 删除元素：remove/empty
    ```
    $(selector).remove();//删除被选元素及子元素
    $(selector).empty();//删除被选元素的子元素

    //参数支持css选择器来过滤
    $('p').remove('.italic')//删除class为.italic的所有p元素
    ```
- css操作：addClass/removeclass/toggleClass/css
    ```
    $(selector).addClass('class1 class2');//添加css类
    $(selector).removeClass('class1');//移除css类
    $(selector).toggleClass('class2');//进行添加/删除css类的切换操作
    
    $(selector).css('background-color');//返回对应css属性
    $(selector).css('background-color', 'green');//设置对应css属性
    $(selector).css({"background-color":"yellow","font-size":"200%"});//通过对象同时设置css属性
    ```
- 尺寸：width/height/innerWidth/innerHeight/outerWidth/outerHeight
    ```
    //无论box-sizeing为何值
    $(selector).width()//返回元素内容宽度（不包含边框，内边距），可传入参数设置值
    $(selector).height()//返回元素内容高度（不包含边框，内边距），可传入参数设置值
    $(selector).innerWidth()//返回包含内边距的元素宽度
    $(selector).innerHeight()//返回包含内边距的元素高度
    $(selector).outerWidth()//返回包含边框，内边距的元素宽度
    $(selector).outerHeight()//返回包含边框，内边距的元素高度
    $(selector).outerWidth(true)//返回包含外边距，边框，内边距的元素宽度
    $(selector).outerHeight(true)//返回包含外边距，边框，内边距的元素高度
    ```
### 遍历
- 祖先：parent/parents/parentsUntil
    ```
    $(selector).parent();//返回直接父元素
    $(selector).parents();//返回所有祖先元素
    $('span').parentsUntil('div')//返回span和div之间的所有祖先元素
    //以上方法都支持可选参数传入css选择器进行过滤
    ```
- 后代：children/find
    ```
    $(selector).children();//返回所有直接子元素，可选参数支持传入css选择器进行过滤
    $(selector).find('*');//返回所有满足条件的后代元素，参数为必需
    ```
- 同胞：siblings/next/nextAll/nextUntil/prev/prevAll/prevUntil
    ```
    $(selector).siblings();//返回所有同胞元素，可选参数支持传入css选择器进行过滤
    $(selector).next();//返回下一个同胞元素
    $(selector).nextAll();//返回跟随的所有同胞元素
    $('h1').nextUntil('h6');//返回h1和h6之间跟随的所有同胞元素
    //prev/prevAll/prevUntil作用与next/nextAll/nextUntil相似，只是方向相反。
    //以上方法都支持可选参数传入css选择器进行过滤
    ```
- 过滤：first/last/eq/filter/not
    ```
    $(selector).first();//返回被选元素中的第一个
    $(selector).last();//返回被选元素中的最后一个
    $(selector).eq(index);//返回被选元素中的第index+1个
    $(selector).filter('.url');//返回被选元素中满足css选择器的元素
    $(selector).not('.url');//返回被选元素中不满足css选择器的元素
    ```

### AJAX
> 即异步javascrip和xml，指在不重载整个网页的情况下，通过后台加载数据并在网页上显示
- load
    ```
    $(selector).load(URL, data?, callback?);//向服务端加载数据，并放入被选元素中
    //URL为加载地址, data为查询键值对，callback为请求完成时触发回调
    //callback包含responseTxt表示成功时结果内容，statusTxt表示状态描述，xhr表示XMLHttpRequest对象
    
    //例：
    $("#div1").load("demo_test.txt",function(responseTxt,statusTxt,xhr){
        if(statusTxt=="success")
            alert("外部内容加载成功!");
        if(statusTxt=="error")
            alert("Error: "+xhr.status+": "+xhr.statusText);
    });
    ```
- get/post：get用于请求数据，post用于提交数据
    ```
    $.get(URL,callback);
    $.get( URL, data?, callback?, dataType)
    $.post(URL,callback);
    $.post( URL, data?, callback?, dataType)
    //URL为加载地址, data为查询键值对，callback为请求完成时触发回调，dataType表示规定服务端响应的数据类型
    //callback包含responseTxt表示成功时结果内容，statusTxt表示状态描述，xhr表示XMLHttpRequest对象
    
    //例：
    $.post("/try/ajax/demo_test_post.php",
    {
        name:"菜鸟教程",
        url:"http://www.runoob.com"
    },
    function(data,status){
        alert("数据: \n" + data + "\n状态: " + status);
    });
    ```

### 其他
- 处理$冲突：noConflict
    ```
    let jq = $.noConflict();//释放对$符号的控制权，便于其他第三方库使用
    jQuery(document).ready();//使用jQuery访问
    jq(document).ready();//使用返回的引用访问
    //通过ready回调的$参数来避免冲突，但是外部依然要使用jQuery
    jQuery(document).ready(function($){
        $("button").click();
    });
    ```
- JSONP
    - JSONP即JSON with Padding，是一种跨域处理方式，因为script标签的src属性不受跨域策略影响。所以可以通过script标签的src属性向服务端发起请求，服务端模拟一段JS代码并返回
    - 通常前端要预定义一个全局方法作为回调，并在请求时携带回调方法名。服务端返回的JS代码会通过调用该回调，并通过参数来传递信息。
    ```
    //纯手写时，需定义全局方法作为回调。然后创建script标签，通过src发起请求。
    //使用jQuery不需要手动创建script标签
    $.getJSON("https://www.test.com?jsoncallback=?", function(data) {
        //通过data获取服务端传递的信息
    });
    ```