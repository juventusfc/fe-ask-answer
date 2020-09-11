# 模拟 call

```javascript
function myCall(context, ...args) {
  if (context) {
    context = new Object(context);
  } else {
    context = window;
  }
  let fn = Symbol();
  context[fn] = this;

  let result = context[fn](...args);
  delete context[fn];
  return result;
}

function Product(name, price) {
  this.name = name;
  this.price = price;
}

Function.prototype.myCall = myCall;

function Food(name, price) {
  Product.myCall(this, name, price);
  this.category = "food";
}

console.log(new Food("cheese", 5).name);
```
