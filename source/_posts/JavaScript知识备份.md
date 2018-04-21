---
title: JavaScript知识备份
tags: |-

  - 前端知识
permalink: javascriptzhi-shi-bei-fen
id: 17
updated: '2016-09-22 11:40:49'
date: 2016-09-22 11:40:08
---

##JavaScript

获取时间

	var someDate = new Date();
	var dateFormated = someDate.toISOString();
	alert(dateFormated);


json和string之间转换

	JSON.stringify(obj， null, 2)将JSON转为字符串。
	JSON.parse(string)将字符串转为JSON格式；

js前端函数传对象获取内容

	onclick="test(this)"
	function test(obj){
		console.log(obj.innerHtml);
	}


js 字符串常用操作：

	这篇文章总结的很全  http://riny.net/2012/the-summary-of-javascript-string/
