# Ajax工作原理

Asynchronous JavaScript and XML

## 原理

通过 `XmlHttpRequest` 对象来向服务器发异步请求，从服务器获得数据，然后用javascript来操作DOM而更新页面。

## 创建Ajax过程

### 创建 XMLHttpRequest 对象

```
var xmlhttp = new XMLHttpRequest();                    //IE7及以上
var xmlhttp = new ActiveXObject('Microsoft.XMLHTTP'); //IE5 和 IE6
```

### 打开连接

```
xmlhttp.open( method, url, async );
```

* method：请求的类型，GET 或 POST
* url：文件在服务器上的位置
* async：true（异步）或 false（同步）
	- 同步：指发出数据后，等接收到响应以后再发送下一个数据包的通讯方式。
	- 异步：指发出数据后，不用等待接收到响应，接着发送下一个数据包的通讯方式。

### 向服务器发送请求

```
 xmlhttp.send(string);
```

* GET请求

```
xmlhttp.send();
```

* POST请求

```
xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");    //向请求添加HTTP头
  xmlhttp.send("fname=Bill&lname=Gates");
```

> 与 POST 相比，GET 更简单也更快，并且在大部分情况下都能用。
然而，在以下情况中，请使用 POST 请求：
>
* 无法使用缓存文件（更新服务器上的文件或数据库）；
* 向服务器发送大量数据（POST 没有数据量限制）；
* 发送包含未知字符的用户输入时，POST 比 GET 更稳定也更可靠。

### 接收服务器响应

```
xmlhttp.onreadystatechange = function( ){
  if (xmlhttp.readyState==4 && xmlhttp.status>=200 && xmlhttp.status<300 || xmlhttp.status==304){
      document.getElementById("myDiv").innerHTML=xmlhttp.responseText;    //responseText    获得字符串形式的响应数据
  }
}
```

* onreadystatechange：每当readyState 属性改变时，就会调用该函数。
* readyState：存有XMLHttpRequest 的状态信息，从 0 到 4 发生变化。
	- 0: 请求未初始化
	- 1: 服务器连接已建立
	- 2: 请求已接收
	- 3: 请求处理中
	- 4: 请求已完成，且响应已就绪
* status：http状态码

## 封装的ajax函数

*GET方法:*

```
function ajax(url,success,error){
  if(window.XMLHttpRequest){
      var oAjax = new XMLHttpRequest();
  }else{
      var oAjax = new ActiveXObject('Microsoft.XMLHTTP');
  }
  oAjax.open('GET',url,true);
  oAjax.send();
  oAjax.onreadystatechange = function(){
      if(oAjax.readyState==4){
          if(oAjax.status>=200&&oAjax.status<300||oAjax.status==304){
              success&&success(oAjax.responseText);    //成功的回调函数
          }else{
              error&&error(oAjax.status);              //失败的回调函
          }
      }
  };
}
```

*POST方法：*

```
function ajax(url,success,error){
  if(window.XMLHttpRequest){
      var oAjax = new XMLHttpRequest();
  }else{
      var oAjax = new ActiveXObject('Microsoft.XMLHTTP');
  }
  oAjax.open('POST ',url,true);
  oAjax.setRequestHeader('Content-Type','application/x-www-form-urlencoded');
  oAjax.send('fname=Bill&lname=Gates');
  oAjax.onreadystatechange = function(){
      if(oAjax.readyState==4){
          if(oAjax.status>=200&&oAjax.status<300||oAjax.status==304){
              success&&success(oAjax.responseText);    //成功的回调函数
          }else{
              error&&error(oAjax.status);              //失败的回调函
          }
      }
  };
}
```