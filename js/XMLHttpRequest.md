# XMLHttpRequest
```shell script
var xhr = new XMLHttpRequest();
xhr.open('post', 'http://xxx/api/abc');
xhr.setRequestHeader("api-tag", "account7");
xhr.setRequestHeader("content-type", "application/json");
// 该属性允许跨域cookie传输到服务端，允许跨域设置响应cookie（同一个根域名下
xhr.withCredentials = true
xhr.send('{}');
```