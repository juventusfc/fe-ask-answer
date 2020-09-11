# 模拟 bind

```javascript
const obj = {
  x: 42,
  getX: function () {
    return this.x;
  },
};

const unboundGetX = obj.getX;
console.log(unboundGetX());
const boundGetX = unboundGetX.bind(obj);
console.log(boundGetX());

Function.prototype.myBind = function (context, ...args) {
  const fn = this;

  const bindFn = (...newArgs) => {
    return fn.call(
      this instanceof bindFn ? this : context,
      ...args,
      ...newArgs
    );
  };
  bindFn.prototype = Object.create(fn.prototype);
  return bindFn;
};

const boundMyBindGetX = unboundGetX.myBind(obj);
console.log(boundMyBindGetX());
```
