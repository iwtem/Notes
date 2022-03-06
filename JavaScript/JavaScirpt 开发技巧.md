## JavaScript 开发技巧



#### 1. 通过 `call` 调用原型链上的方法

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

#### 2. 判断一个变量是否为Function

以下代码摘自 Jquery 源码。

在一些浏览器中，`typeof document.createElement( "object" ) === "function"`，比如 IE 浏览器，到那时我们不需要任何 DOM 的 Node 作为一个 `function`， 所以需要排除此类情况。

另外还有一些获取到的 HTML collections，其 `typeof document.getElementsByTagName("div") === "function"`，此类情况也需要排除。

```javascript
const isFunction = function isFunction(obj) {
  return (
    typeof obj === "function" &&
    typeof obj.nodeType !== "number" &&
    typeof obj.item !== "function"
  );
};
```

#### 3. 正确的获取一个变量的类型

```javascript
function toType(obj) {
  // null 或 undefined，直接转换为字符串返回
  if (obj == null) {
    return obj + "";
  }

  return typeof obj === "object" || typeof obj === "function"
    ? Object.prototype.toString.call(obj).slice(8, -1).toLowerCase()
    : typeof obj;
}
```

#### 4. 判断一个变量是否为类数组

```javascript
function isArrayLike(obj) {
  var length = !!obj && "length" in obj && obj.length,
      type = toType(obj);

  if (isFunction(obj) || isWindow(obj)) {
    return false;
  }

  return (
    type === "array" ||
    length === 0 ||
    (typeof length === "number" && length > 0 && length - 1 in obj)
  );
}
```

