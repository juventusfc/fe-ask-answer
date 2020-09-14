# 认识 Fetch

Fetch 是另一组用于网络请求的 API，是 XMLHttpRequest 的一种替代方案。
Fetch 我们先看[标准](https://fetch.spec.whatwg.org/)。标准中的 [Fetch API](https://fetch.spec.whatwg.org/#fetch-api)这一章，详细讲了 Fetch 的用法。Fetch 中主要定义了 `Header`、`Body`、`Request` 以及 `Response`这几个类。

```javascript
fetch("http://example.com/movies.json")
  .then((response) => response.json())
  .then((data) => console.log(data));
```

Fetch 基于 Promise，相比传统的 XMLHttpRequest 对象，Fetch API 拥有更简洁的编码体验。具体 API 我们就不谈了，可以参考 mdn 或标准。我们来聊聊 Fetch 和 XMLHttpRequest 的区别。

## 兼容性

从兼容性来说，XMLHttpRequest 由于出现得较早，已经被广大浏览器广泛接受。Fetch 的兼容性不好，要引入 polyfill。

## Cookie

有些 Fetch 实现不会带上 Cookie,而 XMLHttpRequest 是会带上 Cookie 的。

## 错误处理

当请求是 404 或 500 之类的错误信息时，Fetch 是 resolved 的。需要使用 `response.ok` 来判断，而 XMLHttpRequest 需要通过 `onreadystatechange` 来判断。

## 超时处理

Fetch 是没有自带超时处理的，而 XMLHttpRequest 可以设置 timeout 以及 ontimeout。当然，我们是可以模拟的：

```javascript
function fetchTimeout(url, init, timeout = 3000) {
  return new Promise((resolve, reject) => {
      fetch(url, init)
      .then(resolve)
      .catch(reject);
      setTimeout(reject, timeout);
  }
}
```

## 取消处理

Fetch 中的取消处理兼容性很差，方式如下：

```javascript
const controller = new AbortController();

fetch("http://domain/service", {
  method: "GET",
  signal: controller.signal,
})
  .then((response) => response.json())
  .then((json) => console.log(json))
  .catch((error) => console.error("Error:", error));

controller.abort();
```

在 XMLHttpRequest 就容易了，只要调用 `xhr.abort()` 就行。
