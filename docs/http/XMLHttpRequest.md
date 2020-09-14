# 认识 XMLHttpRequest

XMLHttpRequest 是用于网络请求的一组 API，咱们先看[标准](https://xhr.spec.whatwg.org/)。

```java
[Exposed=(Window,DedicatedWorker,SharedWorker)]
interface XMLHttpRequestEventTarget : EventTarget {
  // event handlers
  attribute EventHandler onloadstart;
  attribute EventHandler onprogress;
  attribute EventHandler onabort;
  attribute EventHandler onerror;
  attribute EventHandler onload;
  attribute EventHandler ontimeout;
  attribute EventHandler onloadend;
};

[Exposed=(Window,DedicatedWorker,SharedWorker)]
interface XMLHttpRequestUpload : XMLHttpRequestEventTarget {
};

enum XMLHttpRequestResponseType {
  "",
  "arraybuffer",
  "blob",
  "document",
  "json",
  "text"
};

[Exposed=(Window,DedicatedWorker,SharedWorker)]
interface XMLHttpRequest : XMLHttpRequestEventTarget {
  constructor();

  // event handler
  attribute EventHandler onreadystatechange;

  // states
  const unsigned short UNSENT = 0;
  const unsigned short OPENED = 1;
  const unsigned short HEADERS_RECEIVED = 2;
  const unsigned short LOADING = 3;
  const unsigned short DONE = 4;
  readonly attribute unsigned short readyState;

  // request
  undefined open(ByteString method, USVString url);
  undefined open(ByteString method, USVString url, boolean async, optional USVString? username = null, optional USVString? password = null);
  undefined setRequestHeader(ByteString name, ByteString value);
  attribute unsigned long timeout;
  attribute boolean withCredentials;
  [SameObject] readonly attribute XMLHttpRequestUpload upload;
  undefined send(optional (Document or XMLHttpRequestBodyInit)? body = null);
  undefined abort();

  // response
  readonly attribute USVString responseURL;
  readonly attribute unsigned short status;
  readonly attribute ByteString statusText;
  ByteString? getResponseHeader(ByteString name);
  ByteString getAllResponseHeaders();
  undefined overrideMimeType(DOMString mime);
  attribute XMLHttpRequestResponseType responseType;
  readonly attribute any response;
  readonly attribute USVString responseText;
  [Exposed=Window] readonly attribute Document? responseXML;
};
```

从上面的内容我们就可以知道 XHRHttpRequest 的知识点了。

首先，我们先要分清 Ajax 和 XMLHttpRequest 的关系。我们可以先看定义：

> Asynchronous JavaScript and XML, while not a technology in itself, is a term coined in 2005 by Jesse James Garrett, that describes a "new" approach to using a number of existing technologies together, including HTML or XHTML, CSS, JavaScript, DOM, XML, XSLT, and most importantly the XMLHttpRequest object.

> The XMLHttpRequest object is an API for fetching resources. The name XMLHttpRequest is historical and has no bearing on its functionality.

从定义我们可以得知，Ajax 是一种技术方案，而不是技术，它将很多技术融合在一起配合使用。而 XMLHttpRequest 是浏览器提供的一组 API，用来和服务端通过 HTTP 协议进行交互。我们使用 XMLHttpRequest 来发送一个 Ajax 请求。

## 简单例子

```javascript
function sendAjax() {
  // 构造表单数据
  var formData = new FormData();
  formData.append("username", "johndoe");
  formData.append("id", 123456);

  // 创建xhr对象
  var xhr = new XMLHttpRequest();
  // 设置xhr请求的超时时间
  xhr.timeout = 3000;
  // 设置响应返回的数据格式
  xhr.responseType = "text";
  // 创建一个 post 请求，采用异步
  xhr.open("POST", "/server", true);
  // 注册相关事件回调处理函数
  xhr.onload = function (e) {
    if (this.status == 200 || this.status == 304) {
      alert(this.responseText);
    }
  };
  xhr.ontimeout = function () {};
  xhr.onerror = function () {};
  xhr.upload.onprogress = function () {};

  //发送数据
  xhr.send(formData);
}
```

上面是一个使用 XMLHttpRequest 发送 Ajax 请求的例子。

## 建立连接

```java
undefined open(ByteString method, USVString url);
undefined open(ByteString method, USVString url, boolean async, optional USVString? username = null, optional USVString? password = null);
```

## 设置 request header

在设置 Request 的时候，可以设置 request header。

```java
undefined setRequestHeader(ByteString name, ByteString value);
```

注意：

- `setRequestHeader()` 必须在 `open()` 之后， `send()` 之前调用
- 可以调用多次，但是会以 append 的方式追加

  ```javascript
  const client = new XMLHttpRequest();
  client.open("GET", "http://localhost:3000");

  client.setRequestHeader("X-Test", "one");
  client.setRequestHeader("X-Test", "two");
  // {'x-test': 'one, two'}

  client.send();
  ```

## 发送请求

```java
undefined send(optional (Document or XMLHttpRequestBodyInit)? body = null);
```

## 获取 response header

```java
ByteString? getResponseHeader(ByteString name); // 获取指定 header 字段
ByteString getAllResponseHeaders(); // 获取所有 header 字段
```

- 使用 `getAllResponseHeaders()` 看到的所有 response header 与实际在控制台 Network 中看到的 response header 不一样，原因是标准规定客户端无法获取 response 中的 Set-Cookie、Set-Cookie2 这 2 个字段，无论是同域还是跨域请求。
- 使用 `getResponseHeader()` 获取某个 header 的值时，浏览器抛错 Refused to get unsafe header "XXX"，原因是 W3C 的 cors 标准对于跨域请求也做了限制，规定对于跨域请求，客户端允许获取的 response header 字段只限于“simple response header”和“Access-Control-Expose-Headers”。
  - "simple response header"包括的 header 字段有：Cache-Control,Content-Language,Content-Type,Expires,Last-Modified,Pragma;
  - "Access-Control-Expose-Headers"：首先得注意是"Access-Control-Expose-Headers"进行跨域请求时响应头部中的一个字段，对于同域请求，响应头部是没有这个字段的。这个字段中列举的 header 字段就是服务器允许暴露给客户端访问的字段。

## 指定 response 的数据类型

```java
undefined overrideMimeType(DOMString mime);
attribute XMLHttpRequestResponseType responseType;
```

我们着重讲一下 responseType。

| 值            | response 数据类型 | 说明                    |
| ------------- | ----------------- | ----------------------- |
| ""            | String 字符串     | 默认值                  |
| "text"        | String 字符串     | 默认值                  |
| "document"    | Document 对象     | 希望返回 XML 数据时使用 |
| "json"        | JavaScript 对象   |                         |
| "blob"        | Blob 对象         |                         |
| "arrayBuffer" | ArrayBuffer 对象  |                         |

以下是获取一张图片的代码示例：

```javascript
const client = new XMLHttpRequest();
client.open("GET", "/path/to/image.png");
client.responseType = "blob";
client.onload = function () {
  if (this.status === 200) {
    const blob = this.response;
  }
};
client.send();
```

## 获取 response 数据

```java
readonly attribute any response;
readonly attribute USVString responseText;
[Exposed=Window] readonly attribute Document? responseXML;
```
