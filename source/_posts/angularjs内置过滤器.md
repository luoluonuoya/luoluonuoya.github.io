---
title: angularjs内置过滤器
date: 2016-07-17 11:45:42
categories: [AngularJs]
tags: [js,AngularJs]
---
#### angularjs内置过滤器
> 
**currency（货币处理）：**
{{num | currency:'￥'}}
**date（日期格式化）：**
{{date | date:'yyyy-MM-dd hh:mm:ss EEEE'}}
**filter（匹配子串）：**
{{ childrenArray | filter : 'a' }} //匹配属性值中含有a的
{{ childrenArray | filter : 4 }} //匹配属性值中含有4的
{{ childrenArray | filter : {name : 'i'} }} //参数是对象，匹配name属性中含有i的
{{childrenArray | filter : func }} //参数是函数，指定返回age>4的
$scope.func = function(e){return e.age>4;}
**格式化json对象一般用来调试，看看输出的json串**
json
**limitTo（限制数组长度或字符串长度）：**{{ childrenArray | limitTo : 2 }} //将会显示数组中的前两项
**lowercase（小写）**
**uppercase（大写）**
**number（格式化数字）**
 {{num | number:2}}//保留2位小数
**orderBy（排序）**
&lt;div&gt;{{ childrenArray | orderBy : 'age' }}&lt;/div&gt; //按age属性值进行排序，若是-age，则倒序
&lt;div&gt;{{ childrenArray | orderBy : orderFunc }}&lt;/div&gt; //按照函数的返回值进行排序
&lt;div&gt;{{ childrenArray | orderBy : ['age','name'] }}&lt;/div&gt; //如果age相同，按照name进行排序