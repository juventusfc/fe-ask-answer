# 模拟 apply

```javascript
function myApply(context, arr) {
  if (context) {
    context = new Object(context);
  } else {
    context = window;
  }
  let fn = Symbol();
  context[fn] = this;

  let result = context[fn](...arr);
  delete context[fn];
  return result;
}

function Product(name, price) {
  this.name = name;
  this.price = price;
}

Function.prototype.myApply = myApply;

function Food(name, price) {
  Product.myApply(this, [name, price]);
  this.category = "food";
}

console.log(new Food("cheese", 5).name);
```
