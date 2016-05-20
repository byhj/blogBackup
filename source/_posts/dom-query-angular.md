---
title: DOM element在Qjuery和Anglular中转换
date: 2016-05-20 15:21:28
tags:
    - JavaScript
    - Angular
    - Qjuery

---

# DOM 在 Qjuery转换
```
var elm = document.createElement("elm");
var jelm = $(elm);//convert to jQuery Element
var htmlElm = jelm[0];//convert to HTML Element
```

# DOM 在 Angular转换
```
 var elm = document.getElementById('elm'); // 获取DOM的Element
 var elm = angular.element('elm');  //将获取的Element转换为jqLite
 ps:因为Angular使用的是jqLite作为element的操作，如果包含Qjeury则会转换成为一个JQuery Object.
```
