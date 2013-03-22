---
layout: post
styles: [syntax]
title: 鼠标特效
---

#禁用鼠标右键
<h2>实例描述</h2>
为了保证网站内容不被非法拷贝，可以通过判断鼠标按键来禁止用户操作。实现代码如下：{% highlight javascript linenos %}
<script language="javascript">
 function click()
 {
     if(event.button == 2)
     {
     	alert('本网站禁用右键');
     }
 }
 document.onmousedown=click;
</script>
