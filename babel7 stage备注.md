##### @babel/plugin-proposal-export-default-from
> 支持`export default from`语法
``` js
export A from 'a.js'
```
---
##### @babel/plugin-proposal-logical-assignment-operators
> 支持`&&=`和`||=`的语法
``` js
a ||= b;
obj.a.b ||= c;

a &&= b;
obj.a.b &&= c;
```
---
##### @babel/plugin-proposal-optional-chaining
> 支持链式取值的`?.`语法
``` js
const obj = {
  foo: {
    bar: {
      baz: 42,
    },
  },
};
const baz = obj?.foo?.bar?.baz; // 42
const safe = obj?.qux?.baz; // undefined
// 可混合使用
obj?.foo.bar?.baz; // 碰到链中的第一个undefined节点时即返回

// 支持表达式取值方式
obj?.['foo']?.bar?.baz // 42
```
---
##### @babel/plugin-proposal-pipeline-operator
支持`|>`管道运算符(`pipeline-operator`)
``` js
function doubleSay (str) {
  return str + ", " + str;
}
function capitalize (str) {
  return str[0].toUpperCase() + str.substring(1);
}
function exclaim (str) {
  return str + '!';
}
// 原来的方式
let result = exclaim(capitalize(doubleSay("hello")));

>>> Hello, hello!

// 使用管道函数重构
let result = "hello"
  |> doubleSay
  |> capitalize
  |> exclaim;

>>> Hello, hello!
```
---
##### @babel/plugin-proposal-nullish-coalescing-operator
支持语法糖`??`，用于`null`与`undefined`的全等判断
``` js
var foo = object.foo ?? "default";

// 等同于
var foo = (object.foo !== null && object.foo !== void 0) ? object.foo : 'default';
```
---
##### @babel/plugin-proposal-do-expressions
支持`do{...}`语法
``` js
let a = do {
  if(x > 10) {
    'big';
  } else {
    'small';
  }
};
// 等同于
let a = x > 10 ? 'big' : 'small';
```
##### @babel/plugin-proposal-decorators
支持`Class`与`Class method`的`@修饰`语法
``` js
@isTestable(true)
class MyClass { }

function isTestable(value) {
   return function decorator(target) {
      target.isTestable = value;
   }
}


class C {
  @enumerable(false)
  method() { }
}

function enumerable(value) {
  return function (target, key, descriptor) {
     descriptor.enumerable = value;
     return descriptor;
  }
}
```
---
##### @babel/plugin-proposal-function-sent
在generator函数中支持`function.sent`语法
``` js
function* generator() {
    console.log("Sent", function.sent);
    console.log("Yield", yield);
}

const iterator = generator();
iterator.next(1); // Logs "Sent 1"
iterator.next(2); // Logs "Yield 2"
```
---
##### @babel/plugin-proposal-export-namespace-from
支持`export * as xxx`语法
``` js
export * as ns from 'mod';
```
---
##### @babel/plugin-proposal-throw-expressions
支持`throw-expressions`语法
``` js
function getEncoder(encoding) {
  const encoder = encoding === "utf8"
  ? new UTF8Encoder()
  : (
    encoding === "utf16le"
    ? new UTF16Encoder(false)
    : (
        encoding === "utf16be"
        ? new UTF16Encoder(true)
        : throw new Error("Unsupported encoding")
      )
   );
}
// 转义后
const __throw = err => { throw err; };

function getEncoder1(encoding) {
  const encoder = encoding === "utf8" ? new UTF8Encoder() 
                : encoding === "utf16le" ? new UTF16Encoder(false) 
                : encoding === "utf16be" ? new UTF16Encoder(true) 
                : __throw(new Error("Unsupported encoding"));
}
```
---
##### @babel/plugin-syntax-dynamic-import
支持模块的动态导入语法`import()`
``` js
import('./moduleA.js')
```
---
##### @babel/plugin-syntax-import-meta
支持资源的动态导入语法`import()`
``` js
import('./logo.png')
```
---
##### @babel/plugin-proposal-class-properties
支持类的属性声明语法
``` js
class Bork {
  // 声明属性并初始化
  instanceProperty = "bork";
}
```
---
##### @babel/plugin-proposal-json-strings
解决`U+2028`和`U+2029`
``` js
const ex = "before
after";
//                ^ There's a U+2028 char between 'before' and 'after'

>>> 
const ex = "before\u2028after";
//                ^ There's a U+2028 char between 'before' and 'after'
```

#### 参考：
> [Babel Plugins docs](https://babeljs.io/docs/en/plugins#plugin-options)
