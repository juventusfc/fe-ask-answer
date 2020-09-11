# 模拟 new

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
}

let createNew = (con, ...args) => {
  const obj = {};
  Object.setPrototypeOf(obj, con.prototype);
  const result = con.call(obj, ...args);

  return result instanceof Object ? result : obj;
};

console.log(JSON.stringify(new Person("frank", 18)));

console.log(JSON.stringify(createNew(Person, "frank", 18)));
```
