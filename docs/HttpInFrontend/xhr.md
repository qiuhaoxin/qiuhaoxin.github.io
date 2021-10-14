## 浏览器之网络请求 <!-- {docsify-ignore} -->

### 语法

```javascript
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function () {
  /*
   * readyState有4个取值
   * 0：尚未初始化，未调用open
   * 1：调用了open，但未调用send，open只是做了准备
   * 2：调用了send
   * 3：开始接收数据，但未完成
   * 4：请求完成，响应数据填充xhr的属性status、statusText、responseText、responseXML
   * 每次readyState状态变化时，onreadystatechange方法都会调用
   */
  if (xhr.readyState == 4) {
    if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304) {
      let responseData;
      if (xhr.getResponseHeader('Content-Type') == 'application/xml') {
        responseData = xhr.responseXML;
      } else {
        responseData = xhr.responseText;
      }
    }
  }
};
//请求动作、资源地址、是否异步操作
xhr.open('method', 'url', isAsync);
xhr.timeout = 1000;
xhr.ontimeout = function () {
  //超时回调
};
//设置/变更 请求首部、自定义请求首部
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.send(null); //请求主体，如果为空要传null

//调用完毕记得对xhr进行解引用操作，不建议重用xhr对象，避免有些浏览器因兼容性问题引发内存泄漏
xhr = null;

//取消请求
document.getElement('stopXhr').addEventListener('click', () => {
  if (xhr) {
    xhr.abort();
    xhr = null;
  }
});
```

兼容性：上面的写法兼容到 IE6+ 的浏览器和其他浏览器，如果是低于 IE6 的浏览器要用 MSHttp 库来实现兼容，这里不阐述，可自行查找相关资料。
