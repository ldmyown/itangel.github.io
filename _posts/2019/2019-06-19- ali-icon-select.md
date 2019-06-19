---
layout: post
title: 阿里图标库全选
category: other
tags: [other]
---



使用阿里图标库时，加入购物车，不能批量添加，

此时使用js脚本可以批量添加



~~~javascript
var icons = document.querySelectorAll('.icon-gouwuche1');
    for (var i = 0; i < icons.length; i++) {
    icons[i].click();
}
~~~

