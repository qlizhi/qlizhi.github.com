---
layout: post
styles: [syntax]
title: 鼠标特效
---

#禁用鼠标右键
<h2>实例描述</h2>
为了保证网站内容不被非法拷贝，实现代码如下：
{% highlight javascript linenos %}
<script language="javascript">
  function click()
  {
    if(event.button == 2)
    {
     	alert('禁止右键');
    }
  }
  document.onmousedown=click;
</script>
{% endhighlight %}
