---
layout: post
title: jquery瀑布流布局(masonry.js)
categories: 前端
tags: jquery
---

这样参差不齐的多栏布局，叫“方砖石布局”，和“瀑布流布局”，很多网站都有使用这样的布局，比如：[淘宝的哇哦](http://wow.taobao.com/)、[花瓣网](http://huaban.com/)、[蘑菇街](http://www.mogujie.com/)

![09161615-21dbb8e2d53c426fa3e7a5b2368f95c8.jpg](https://dn-biezhi.qbox.me/2015/10/3330929200.jpg)

<!-- more -->

这里，我们仅使用瀑布流插件实现当页内容的排列。 

那这插件到底有什么用呢？请看看下面的图片：右图是左图使用了插件之后的效果。[具体例子](http://masonry.desandro.com/)

![09164127-17a7a3da83764cbb8ea23d29f1738234.jpg](https://dn-biezhi.qbox.me/2015/10/2317610519.jpg)

引用插件实例：

```css
<style type="text/css">
img { border:none; }
.wrapper { width:1000px; margin:0 auto; }
.wrapper h3{color:#3366cc;font-size:16px;height:35px;line-height:1.9;text-align:center;border-bottom:1px solid #E5E5E5;margin:0 0 10px 0;}
#con1_1 { position:relative; }
#con1_1 .product_list { position:absolute; left:0px; top:0px; padding:10px; background:#eee; margin:5px;}
.product_list img { width:200px;}
.product_list p { padding:5px 0px; font-size:12px; text-align:center; color:#333;  white-space:normal; width:200px;}
</style>
```

```html
<div class="wrapper">
  <h3>瀑布流实例</h3>
  <div id="con1_1">
      <div class="product_list"> <a href="#"><img src="images/img6.jpg"></a><p>【乖怪蜜桃.搭搭配】~貂绒嫁到~今年好流行的材质呢~跟风购入~颜色质感都好喜欢de 说呢~重点还很保暖</p></div>
    <div class="product_list"> <a href="#"><img src="images/img4.jpg"></a><p>夏天花裤子是一定要有的哦！</p> </div>
    <div class="product_list"> <a href="#"><img src="images/img1.jpg"></a><p>薄荷绿的西裤子 怎么穿怎么好看</p></div>
    <div class="product_list"> <a href="#"><img src="images/img2.jpg"></a><p>超长款的雪纺开衫，很仙很气场呢，敞开穿比较大气，扣起来系个腰带，很淑女哈</p></div>
    <div class="product_list"> <a href="#"><img src="images/img3.jpg"></a><p>二零一二，十月。去年夏季最喜欢的雏菊小衬衫+牛仔长裙 还有最爱的复古手工包</p></div>
    <div class="product_list"> <a href="#"><img src="images/img5.jpg"></a><p>还有4年就奔3了，现在就拼命地装嫩吧，不然真到30大几岁还这副模样真真是恶心透了 啊哈哈~~ 装嫩必备蛋糕裙！</p></div>
        <div class="product_list"> <a href="#"><img src="images/img8.jpg"></a><p>依旧牛仔上衣和室内照。不要以为主角是衣服，其实是鞋子。</p></div>
    <div class="product_list"> <a href="#"><img src="images/img7.jpg"></a><p>很简单的大学生而已</p></div>
  </div>
</div>
```

```javascript
<script type="text/javascript" src="js/jquery.js"></script>
<script type="text/javascript" src="js/jquery.masonry.min.js"></script>
<script type="text/javascript">
$(document).ready(function(){
    var $container = $('#con1_1');    
    $container.imagesLoaded(function(){
        $container.masonry({
            itemSelector: '.product_list',
            columnWidth: 5 //每两列之间的间隙为5像素
        });
    });
    
});
</script>
```

**如果不想使用插件，也可以把上面的js部分换为：**

```javascript
<script type="text/javascript" src="js/jquery.js"></script>
<script type="text/javascript">
/*
原理：1.把所有的li的高度值放到数组里面
     2.第一行的top都为0
     3.计算高度值最小的值是哪个li
     4.把接下来的li放到那个li的下面
*/
var margin = 10;//设置间距
var li=$(".product_list");//区块名称
var    li_W = li[0].offsetWidth+margin;//取区块的实际宽度
function liuxiaofan(){
    var h=[];//记录区块高度的数组
    var n = 960/li_W|0;
    for(var i = 0;i < li.length;i++) {
        li_H = li[i].offsetHeight;//获取每个li的高度
        if(i < n) {//n是一行最多的li，所以小于n就是第一行了
            max_H =Math.max.apply(null,h);
            h[i]=li_H;//把每个li放到数组里面
            li.eq(i).css("top",0);//第一行的Li的top值为0
            li.eq(i).css("left",i * li_W);//第i个li的左坐标就是i*li的宽度
            }
        else{
            min_H =Math.min.apply(null,h) ;//取得数组中的最小值，区块中高度值最小的那个
            minKey = getarraykey(h, min_H);//最小的值对应的指针
            h[minKey] += li_H+margin ;//加上新高度后更新高度值
            li.eq(i).css("top",min_H+margin);//先得到高度最小的Li，然后把接下来的li放到它的下面
            li.eq(i).css("left",minKey * li_W);    //第i个li的左坐标就是i*li的宽度
        }
      //  $("p").eq(i).text("高度："+li_H);//把区块高度值写入对应的区块H2标题里面
    }
    max =Math.max.apply(null,h) ;
    $("#con1_1").css("height",max);
}
/* 使用for in运算返回数组中某一值的对应项数(比如算出最小的高度值是数组里面的第几个) */
function getarraykey(s, v) {for(k in s) {if(s[k] == v) {return k;}}}
/*这里一定要用onload，因为图片不加载完就不知道高度值*/
window.onload = function() {liuxiaofan();};
window.onresize = function() {liuxiaofan();};

//鼠标在上样式
$(function(){
$(".product_list").hover(function(){
$(this).css("background-color","#ddd");
},function() {
$(this).css("background-color","#eee");
});
});
</script>
```

[点击这里下载实例](http://download.csdn.net/detail/roro5119/5238509)
