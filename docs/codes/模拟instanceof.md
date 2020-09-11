# 模拟 instanceof

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

const myInstanceof = (left, right) => {
  let leftValue = Object.getPrototypeOf(left);
  let rightValue = right.prototype;
  while (true) {
    if (leftValue === null) return false;
    if (leftValue === rightValue) return true;
    leftValue = Object.getPrototypeOf(leftValue);
  }
};

let p = new Person("frank", 18);

console.log(p instanceof Person);
console.log(myInstanceof(p, Person));
```
