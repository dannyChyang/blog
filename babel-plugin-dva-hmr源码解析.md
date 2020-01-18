`babel-plugin-dva-hmr`插件主要做了一件事：**找到代码中`dvaIns.router()`的语句位置，注入hmr相关代码**

在`dva`中的路由注册代码通常是这样
``` js
dvaIns.router(require('./routers').default);
```
在经过`babel-plugin-dva-hmr`转换后，生成代码如下
```
(function () {
  var router = require('./routes');
  dvaIns.router(router.default || router);
  app.use({
    onHmr: function onHmr(render) {
      if (module.hot) {
        // newRender function declaration
        module.hot.accept('./routes', function () {
          var router = require('./router2');
          newRender(router.default || router);
        });
      }
    }
  });

  if (module.hot) {
    //  ...
    module.hot.accept('./models/modelA', function () {
                    app.unmodel(modelNamespaceMap['./models/modelB']);
    //  ...
    dvaIns.model(_model);
  }
})();
```
生成后的代码，主要分为三个部分。
1. `dvaIns.router()`编译为执行环境中可运行代码部分
2. `app.use({ onHmr: ...})`部分
3. `if(module.hot)`部分

第2部分主要用来处理组件的热重载。用到了`dva`提供的`onHmr`API。当hmr发生时，会重新渲染一次路由组件。
> `module.hot`是`react-hot-loader`提供的API

第3部分则主要处理`model`相关的重载。当`model`触发hmr，首先调用`dvaIns.unmodel()`注销该`model`，重新引入`model`文件后，使用`dvaIns.model()`重新挂载新的model。

> **这部分的if代码会根据`dvaIns.model()`的挂载的`model`数量生成N份**

## 思路
为了实现在编译阶段，注入第2和第3部分的代码，首先整理一下实现思路
1. 处理渲染组件部分
    1. 查找`dvaIns.router()`调用语句
    2. 解析出`dvaIns.router(arg)`中`arg`参数的值(即路由组件的导入路径)
2. 处理`model`部分
    1. 查找`dvaIns.model()`调用语句
    2. 解析出`dvaIns.model(arg)`中`arg`参数的值(即挂载`model`的导入路径)
3. 使用`onHmr`API和`module.hot`生成hmr代码
4. 将源代码中`dvaIns.router()`的语句替换为`hmr`相关代码

## babel插件
`babel`插件是一个返回了`visitor`对象的函数。

在`visitor`对象上可以定义一系列`hook`函数，**在babel解析为AST之后，可以将解析出的语句的路径对象`path`，传入相应`hook`函数中进行二次加工**。
``` js
{
    visitor: {
        // hookName: (path) => {}
    }
}
```

## 实现逻辑
`dvaIns.router()`和`dvaIns.model()`都是函数调用，因此可以使用`CallExpression hook`来找出所有函数调用语法，进行下一步处理。

下面代码中，首先使用`isRouterCall()`与`isModelCall()`来区分出`router()`和`model()`的调用。
``` js
CallExpression(path, state) {
  if (isRouterCall(callee, path.scope)) {
    // ...
  } else if (isModelCall(callee, path.scope)) {
    // ...
  }
}
```

### `dvaIns.model()`的处理
首先是代码中`model`导入路径的收集，使用`getRequirePath()`来解析`model`的导入路径，然后放入缓存`modelPaths`变量中。
``` js
// CallExpression
const modelPaths = {};
// ...
if (isModelCall(callee, path.scope)) {
  modelPaths[filename] = modelPaths[filename] || [];
  modelPaths[filename].push(getRequirePath(args[0], path.scope));
}
```
取得代码中所有`model`的导入路径之后，在解析`dva.router()`并生成hmr代码时，将遍历`modelPaths`并生成 N 份`if(module.hot)`代码段。
``` js
// getHmrString()
modelPaths.map(modelPath => `
  if (module.hot) {
    const modelNamespaceMap = {};
    let model = require('${modelPath}');
    if (model.default) model = model.default;
    modelNamespaceMap['${modelPath}'] = model.namespace;
    module.hot.accept('${modelPath}', () => {
      try {
        app.unmodel(modelNamespaceMap['${modelPath}']);
        let model = require('${modelPath}');
        if (model.default) model = model.default;
        dvaIns.model(model);
      } catch(e) { console.error(e); }
    });
  }
`)
```

### `dvaIns.router()`的处理
针对`router组件`的处理与`model()`相似，使用`getRequirePath()`获取渲染组件`module`的导入路径后，连同`routerPath`对象一起，调用`getHmrString()`函数生成注入代码。

最后使用`replaceWithSourceString`API，将源代码中`dvaIns.router()`的语句替换为`hmr`代码替。
``` js
// CallExpression
if (isRouterCall(callee, path.scope)) {
  const routerPath = getRequirePath(args[0], path.scope);
  if (routerPath) {
    // ...
    path.parentPath.replaceWithSourceString(
      getHmrString(
        callee.object.name,
        routerPath,
        modelPaths[filename],
        opts.container,
        !opts.disableModel
      )
    );
  }
}
```
在了解了`babel-plugin-dva-hmr`的实现逻辑后，做一下小结：
- 该插件的实现，主要是找到代码中`dvaIns.router()`的语句位置，注入hmr相关代码
- 基于`module.hot`API，实现`hmr`的本质是收集`router组件`与`model`的导入路径，通过重新渲染、重新挂载的方式实现热重载
- 该插件只能针对`dvaIns.model()`的`model`实现`hmr`，没有实现`dva/dynamic`引入的model。
- `module.hot.accept`API通过二次导入来实现重载。因此要将model定义在单独的文件中，不要与`dvaIns.model()`调用在同一作用域。

> 由于 `dva/dynamic` 中 `model` 没有实现`hmr`，这部分逻辑需要自己实现。

## 工具函数
### getRequirePath
作用：**解析导入语法中实参的值。**

要考虑处理以下三种参数类型场景：
#### 1. 属性
``` js
dvaIns.model(require('./modelA').default)
```
识别出的参数`X.default`是`member表达式`，因此使用`MemberExpression`来区分处理这种语法。

#### 2. 函数调用
``` js
dvaIns.model(require('./modelA'))
```
参数`require()`是函数调用，使用`CallExpression`来捕获该语法。

#### 3. 变量标识
``` js
dvaIns.model(modelC)
```
`modelC`是一个变量，使用`Identifier`来匹配标识符场景。然后使用`getImportRequirePath`进一步处理，找出该变量的值。

### getImportRequirePath
作用：**获取导入语句实参的值**

在上面`getRequirePath`的处理逻辑中，当传参数是`Identifier`标识符类型时，需要进一步找到变量的具体值。这部分逻辑在`getImportRequirePath`来完成。

首先在当前作用域中找到该变量声明语句的位置
``` js
const binding = scope.bindings[identifierName];
const { parent } = binding.path;
```
> `binding`的`path.parent`指向其声明语句

在寻找变量值的过程中，需要注意以下两种声明语法场景：
#### 语法场景：导入语句
``` js
import modelA from './modelA';
dvaIns.model(modelA);
```
`modelA`的声明语句是`import`类型，使用`isImportDeclaration`来区分这种语法。然后在`importPath.source.value`中获取其导入路径`'./modelA'`。
``` js
const binding = scope.bindings[identifierName];
// ...
const { parent } = binding.path;
if (t.isImportDeclaration(parent)) {
    return parent.source.value;
}
```

##### 语法场景：导入表达式
``` js
const modelB = require('./modelB');
dvaIns.model(modelB);
```
上例中`modelB`是一个`require`表达式，声明语句为变量声明，使用`isVariableDeclaration`以区分该类型。

``` js
const binding = scope.bindings[identifierName];
// ...
const { parent } = binding.path;
// ...
if (t.isVariableDeclaration(parent)) {
  const declarator = findDeclarator(parent.declarations, identifierName);
  if (declarator) {
    if (isRequire(declarator.init)) {
      return getArguments0(declarator.init);
    }
    if (isRequireDefault(declarator.init)) {
      return getArguments0(declarator.init.object);
    }
  }
}
```

### findDeclarator
作用：查找变量的声明
在JS语法规范中，一条变量声明语句可以同时创建多个变量。因此需要注意以下情况，`otherModel`和`a`由一条语句声明。
``` js
const otherModel = {
  namespace: 'other',
}, a = 1;
```

### isRequire和isRequireDefault
用于区分，`getImportRequirePath`的`变量声明`场景中，以下两种语法。
- `require('./modelB')`
- `require('./modelB').default`

### getArguments0
针对函数调用语句，获取首个参数的值
``` js
function getArguments0(node) {
    if (t.isLiteral(node.arguments[0])) {
      return node.arguments[0].value;
    }
}
```

### isModelCall
识别`dvaIns.model()`调用

### isRouterCall
识别`dvaIns.router()`调用

### isDvaInstance & isDvaCallExpression
用于判断在`dvaIns.xxx()`的调用中，`dvaIns`是`dvaJs module`的实例

## 小结
以上，通过`babel-plugin-dva-hmr`的源码解析，了解其具体实现及限制。
### 限制
- 只能针对`dvaIns.model()`的`model`实现`hmr`，不支持`dva/dynamic`等方式引入的model。
- 在实现`hmr`时，`module.hot.accept`API通过二次引入来实现重载。因此要将model定义在单独的文件中，不要与`dvaIns.model()`调用在同一作用域。（在同一文件中声明model并在下面调用`dvaIns.model()`）

### 涉及知识点：
- 使用`babel`插件解析并二次处理JS语法。
- 学习AST中的常见逻辑处理。
    - 解析模块导入(import、require、require().default)
    - 查找变量声明(binding、declarations)
    - 区分参数类型，函数调用(function expression)、属性(member expression)、字面量(identifier)
