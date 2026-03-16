---
title: 整理的css知识
date: 2018-12-14
---

css标准地址 <https://www.w3.org/TR/CSS2/visuren.html#inline-formatting>  
1、block、inline、inline-block  
每个页面元素都有一个display属性，每个元素的display属性都有一个默认值，比如div的display属性为block，span的display属性为inline，inline元素不会自动换行且没有宽和高、block元素有宽和高度，且会自动换行。常见的元素分类如下：  
（1）block元素：body form textarea h1 - h6 html table button p ul ol div  
（2）inline元素：title span a em b strong I map 等  
（3）inline-block元素：img input td select textarea label  
区别方式为：是否可以设置宽、高、margin、padding值、是否会换行。  
对inline元素这只padding-top padding-bottom margin-bottom margin-top不会对周边元素产生影响，会加大自身的范围。  
2、嵌套规则：  
（1）块状元素可以包含内联元素或者块元素，内连元素不可以包含块元素只包含内连元素
    
    
    ```bash
    <a href="#"><span></span></a>  — 正确
    <div><h1></h1><p></p></div> — 正确
    <span><div></div></span> — 正确
    ```

（2）h1- h6、p、 dt这几个块元素只能包含内联元素或者可变元素
    
    
    ```bash
    <p><ol><li></li></ol></p> -- wrong
    <p><div></div></p>  — wrong
    ```

（3）特殊的li标签内可以出现div标签  
（4）块级元素可以与块级元素并列、内联元素可以和内联元素并列
    
    
    ```bash
    <div><h2></h2><p></p></div>  — right
    <div><a href="#"></a><span></span></div>  — right
    <div><h2></h2><span></span></div> -- wrong
    ```

3、inline元素改为block元素的方式  
（1）直接将display值设置为block/inline-block  
（2）直接将position设置为absolute或者fix  
4、inline-block元素出现缝隙的解决方式  
（1）两个inline-block元素之间不能出现换行、空格等  
（2）设置margin-right值为负值  
（3）将父元素的font-size、letter-spacing、word-spacing值  
5、block元素内的inline-block元素出现底部空白或者两个inline-block元素无法对齐的情况  
（1）使用vertical-align：top  
（2）例子： <https://segmentfault.com/a/1190000010934928>  
6、em: 相对于父级元素font-size的比例。  
rem：相对于根元素html的font-size比例。  
ex：所用字体中x的高度，通常取em的一半。  
7、css的三种定位机制：普通流、浮动和绝对定位。position有relative、absolute、fix、static  
relative：相对于元素应该出现的位置的相对位置。  
absolute：绝对定位的盒子是相对于离他最近的一个已定位的盒子进行定位的，可能是relative，可能是absolute。默认是body。  
浮动和绝对定位都将元素剥离了文档流。  
8、background-origin: 取值可以为content-box、padding-box、border-box，规定背景图放置的位置。  
9、box盒子模型  
（1）ie设置的height和 width = margin + padding +content-width  
（2）标准的盒子模型的 width = content-width  
（3）css3中加了一个属性叫box-sizing 来区分上面的情况（content-box \ border-box \ padding-box）  
10、formatting context: 它是页面中的一块渲染区域、并且有自己的一套渲染规则、他决定了元素如何定位以及与其他元素如何相互作用。常见的类别有：  
（1）BFC (block formatting context)

  * BFC是一个独立的区域，不受外部元素影响也不影响外部元素
  * 只有块级元素参加
  * box会在垂直方向上一个接一个的放置
  * 属于同一个BFC的相邻的两个box的margin会重叠
  * 计算BFC高度时，浮动元素也参与计算
  * 每个元素的margin box的左边与包含块的border box的左边相接处，存在浮动也是如此  
会生成BFC的方式：
  * 根元素比如body
  * float不为none
  * position为absolute或者fixed
  * display为inline-block、table-cell、table-caption、flex、inline-flex
  * overflow为hidden、scroll、auto  
例子： <https://www.jianshu.com/p/66632298e355>  
（2）IFC (inline formatting context)
  * 只会在一个块级元素中只包含内连级别元素时才会生成。
  * 行内级元素（inline-level element）的display为inline、inline-block、inline-table
  * 行内级元素生成行内盒（inline-level box），参与行内格式化上下文
  * 行框的宽度由其内部包含的块以及浮动元素所决定
  * 如果几个行内框无法放入一个行框内，他们可能分配在两个或者多个垂直的行框内
  * 同一个行内框如果不能放入一个行框内，也会分配到多个垂直的行框
  * 行框的高度可以容纳所包含的框，对齐标准为vertical-align（此处会引入下面的12问题）  
（3）GFC  
（4）FFC  
11、line-height: 设置行间的距离，可以设置的方式  
（1）number：以当前的字体属性值来设置行高(不同的浏览器默认值不同， 介于1 - 1.2之间)  
（2）百分比：以当前字体属性的百分比来设置  
（3）length：固定的值  
12、vertical-align: 默认值为baseline，即按照基线进行对齐，此值是和line-height相关的。  
（1）css中对基线的定义为：inline-block元素的基线是标准流中最后一个行框的基线， 除非这个行框没有行盒子或者本身overflow属性计算值不是visible，这种情况下，基线是该元素margin底边缘。  
（2）inline元素有两个高度：和字体相关的content-area，以及实际区域virtual-area（line-height）


