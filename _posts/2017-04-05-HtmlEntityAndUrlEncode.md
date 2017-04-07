---
layout: post
title: htmlentities()与urlencode()
date: 2017-04-05
tags: 服务器    
---
之前做过一段时间php，现在工作需要，再拾起来，翻看php文档，扫到htmlentities()与urlencode()两个方法，对使用场景做下总结。

### HttpEntity
在html中有些字符是预留的，（比如，< 和> 会被渲染为标签，&nbsp代表空格等等）。若对这些不进行处理，可能**得不到预期的显示**，或者导致**XSS漏洞**。所以对于在html上呈现的内容，要根据情况使用htmlentities()转码，比如用户通过输入框提交的内容，都需要处理下，切记**不要信任用户输入**。

字符实体的表示方法：
* &entity_name;
* &#entity_number;

比如，需显示小于号，我们必须这样写：
```html
&lt; 或 &#60;
```
**空格```&nbsp;```**
浏览器总是会截短 HTML 页面中的空格。如果在文本中写 10 个空格，在显示该页面之前，浏览器会删除它们中的 9 个。如需在页面中增加空格的数量，就需要使用 ```&nbsp; ```字符实体。
完整的实体符号，可以参看[HTML实体符号参考手册](http://www.w3school.com.cn/tags/html_ref_entities.html)

### Url Encode
这个问题涉及到URL的定义,URL是为了统一的命名网络中的一个资源（URL不是单单为了HTTP协议而定义的，而是网络上的所有的协议都可以使用）。因为一些历史的原因URL设计者使用US-ASCII字符集表示URL。（原因比如ASCII比较简单；所有的系统都支持ASCII）为了满足URL的以上特性，设计者就将转义序列移植了进去，来实现通过ASCII字符集的有限子集对任意字符或数据进行编码。URL转义表示法包含一个百分号，后面跟上两个表示字符ASCII码的十六进制数值，比如%E4。

不同的编程语言对于URL的转义还不太一样，比如：
[java URLEncoder](http://docs.oracle.com/javase/8/docs/api/java/net/URLEncoder.html)
[php urlencode](http://php.net/manual/zh/function.urlencode.php)

具体参见[Wikipedia](https://en.wikipedia.org/wiki/Percent-encoding)

### 所以
* 对于用户通过网页提交的数据，使用html entities进行转意，因为这些数据将来很可能会再次被呈现在网页上，避免xss；
* 对于构造url，使用url encode进行转意，因为这是大家通用的标准;