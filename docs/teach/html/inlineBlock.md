很早之前我们想让块元素比如`div、li`等在一行显示，通常使用float属性使元素浮动进而在一行展示，但这种做法其实个人是非常不提倡的。

**因为浮动后元素本身会脱离文档流**，导致父容器的高度没有被子元素自动撑开，为此我们不得不单独写一个清除浮动的css来解决这个问题，然后在需要的地方进行类名的添加，比如这样：

```css
.clearfix:after,.clearfix:before{
   content:"";
   display:table;
}
 .clearfix:after{
   clear:both;
  }
 .clearfix{zoom:1;}
```

在那个css3没有普及的时候，又不想用float怎么办，我当时通过将块状元素属性设置成`display:inline-block;`来解决这个问题，而且元素不会脱离文档流，造成一些未知的问题。


### 人生不如意十之八九
但在实际运用过程中，会发现元素之间产生了神秘的空隙，看下例，三个div之间产生了莫名空隙

[示例代码](../exam/inlineBlock.html ':include :type=iframe width=100%')

代码如下：
```html
<style>
  #exam{
    padding: 50px;
    text-align: center;
  }
  .one,.two,.three{
    display:inline-block;
    width: 100px;
    height: 50px;
    border:1px solid teal;
  }
</style>

<div id="exam">
  <div class="one"></div>
  <div class="two"></div>
  <div class="three"></div>
</div>
```

### 但求甚解
当时很纳闷，神秘的空隙到底从何而来，百度了一番，感觉都没说到点子上，于是在stackoverflow(程序员版知乎)上搜了一下关键字终于找到了答案,有个回答是这么说的

```
In this instance, your div elements have been changed from block level elements to inline elements. A typical characteristic of inline elements is that they respect the whitespace in the markup. This explains why a gap of space is generated between the elements

大体意思：
因为内联元素的典型特征是尊重标签中的的空白，这地方的空白我个人认为包含空格和回车。 这解释了div从块元素更改为内联元素后，为什么产生神秘的空隙。因为我们通常写代码时，每行最后都会回车换行。
```

### 方法总比问题多

> 方法1：移除标签中的空白

```html
Exam1:
<div class="one"></div><!--
--><div class="two"></div><!--
--><div class="three"></div>
  
Exam2:
<div class="one"></div
><div class="two"></div
><div class="three"></div>

Exam3:
<div class="one"></div><div class="two"></div><div class="three"></div>

Exam4:
<div class="one">111
</div><div class="two">222
</div><div class="three">333
</div>
```

> 方法2：重置font-size

?> 由于内联元素之间的空白由字体大小决定，所以可以简单地将字体大小重置为0，从而删除元素之间的空间。只需在父元素上设置字体大小`font-size:0;`，然后为子元素重新声明一个新的字体大小即可。

```css
#exam{font-size:0;}
.one, .two, .three{font-size:16px;}
```