## JavaScript 开发技巧



##### 1. 通过 `call` 调用原型链上的方法

​	由于 JavaScript 没有对原型链上的方法的保护机制，对象可以在自有属性中覆盖原型链上的方法或属性。当一个对象占用其原型链上的某个属性名的属性时，可能会出现如下问题：

```javascript
const foo = {
  bar: 'Here be dragons',
  hasOwnProperty: function () {
    return false;
  },
};

// 始终返回 false
foo.hasOwnProperty('bar');
foo.hasOwnProperty('hasOwnProperty');
```

遇到这种情况，可以通过 call 方法直接使用原型链上真正的方法：

```javascript
// true，原型链上的方法
({}).hasOwnProperty.call(foo, 'bar');
({}).hasOwnProperty.call(foo, 'hasOwnProperty');

// 也可使用 Object 原型上的方法
Object.hasOwnProperty.call(foo, 'bar');
Object.hasOwnProperty.call(foo, 'hasOwnProperty');
```





