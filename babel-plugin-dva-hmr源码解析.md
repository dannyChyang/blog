`babel-plugin-dva-hmr`插件主要做了一件事：**找到代码中`dvaIns.router()`的语句位置，注入hmr相关代码**

通常，在`dva`中的调用代码会写为这样
``` js
dvaIns.router(require('./routers').default);
```
在经过`babel-plugin-dva-hmr`转换后，生成的代码如下
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
生成后的代码，主要包括3部分。
1. `dvaIns.router()`编译为执行环境后的部分
2. `app.use()`部分
3. `if(module.hot)`部分

在其中，第2部分使用`dva`的`onHmr`API处理组件的重载。当hmr发生时，重新渲染一次根组件。
> `module.hot`是`react-hot-loader`的API

第3部分的`if(module.hot)`，主要处理`model`。当hmr发生时，会重新注销并挂载该model。
**`model`部分的if代码会根据model的注册数量生成N份**

## 实现逻辑
编辑后需要注入第2和第3部分的代码，整理一下实现思路
1. 处理`dvaIns.router()`
    1. 找出`dvaIns.router()`函数的调用
    2. 解析出`dvaIns.router()`的参数值(组件的引用路径)
2. 处理`dvaIns.model()`
    1. 找到`dvaIns.model()`函数的调用
    1. 解析出`dvaIns.model`调用时的参数值(`model`的引用路径)
3. 使用`onHmr`API和`module.hot`生成hmr代码
4. 将生成的hmr代码放到源代码中`dvaIns.router()`语句之后。（这里使用的是替换）。

### babel插件
`babel`插件的语法是一个返回了`visitor`对象的函数。

在`visitor`对象上可以定义一系列`hook`函数，**在babel解析AST时，将解析`path`传入相应`hook`函数中进行二次加工**。
``` js
{
    visitor: {
        // hookName: (path) => {}
    }
}
```

### 实现思路
`dvaIns.router`和`dvaIns.model`都是函数调用，因此可以使用`CallExpression hook`来找出所有函数调用语法，进行下一步处理
``` js
CallExpression(path, state) {
  if (isRouterCall(callee, path.scope)) {
    // ...
  } else if (isModelCall(callee, path.scope)) {
    // ...
  }
}
```
上面代码中，使用`isRouterCall`与`isModelCall`来区分出`dvaIns.router()`和`dvaIns.model`的调用。

### `dvaIns.model()`的处理
``` js
// CallExpression
const modelPaths = {};
// ...
if (isModelCall(callee, path.scope)) {
  modelPaths[filename] = modelPaths[filename] || [];
  modelPaths[filename].push(getRequirePath(args[0], path.scope));
}
```
使用`getRequirePath(args[0], path.scope)`解析出`model`的引用路径。然后缓存进`modelPaths`变量中。

在之后生成代码时，会遍历`modelPaths`生成N份`if(module.hot)`代码段
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
上面代码中，使用`getRequirePath(args[0], path.scope)`函数获取渲染组件的引用路径后，连同`dvaIns.model()`函数收集的`routerPath`对象一起，调用`getHmrString()`函数生成注入代码。最后使用`replaceWithSourceString`，替换源代码中`dvaIns.router()`语句为注入代码。

## 工具函数
### getRequirePath
作用：**解析`dvaIns.model()`语法中的实参值。**

要兼容处理以下3种传参类型：
##### 1. 属性
``` js
dvaIns.model(require('./modelA').default)
```
识别出的`N.default`是取`default`属性，使用`MemberExpression`来捕获这种语法。

##### 2. 函数调用
``` js
dvaIns.model(require('./modelA'))
```
此时识别出的是`require()`函数调用，使用`CallExpression`来捕获该语法。

##### 3. 变量标识
``` js
dvaIns.model(modelC)
```
`modelC`是一个变量，使用`Identifier`来捕获标识符。然后使用`getImportRequirePath`进一步处理，找出该变量的值。

### getImportRequirePath
作用：**获取导入语句实参的值**

在上面`getRequirePath`的处理逻辑中，当传参数是`Identifier`类型时，需要进一步查找变量的值。这部分逻辑由`getImportRequirePath`来完成。
``` js
const binding = scope.bindings[identifierName];
const { parent } = binding.path;
```
> `binding`的`path.parent`指向其声明语句

这里要注意以下2种语法场景：
##### 语法场景：导入语句
``` js
import modelA from './modelA';
dvaIns.model(modelA);
```
上例中，`modelA`由`import`声明创建。使用`importPath.source.value`获取引入路径`'./modelA'`。

```js
const binding = scope.bindings[identifierName];
// ...
const { parent } = binding.path;
if (t.isImportDeclaration(parent)) {
    return parent.source.value;
}
```

##### 语法场景：变量声明
``` js
const modelB = require('./modelB');
dvaIns.model(modelB);
```
上例中`modelB`是一个`require`表达式，本质是变量，因此使用`VariableDeclaration`判断其类型。

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
作用：查找变量声明
``` js
const otherModel = {
  namespace: 'other',
}, a = 1;
```
上例中，`otherModel`和`a`由一条语句声明，进行声明查找时需要注意这种情况。
``` js
function findDeclarator(declarations, identifier) {
  for (const d of declarations) {
    if (t.isIdentifier(d.id) && d.id.name === identifier) {
      return d;
    }
  }
}
```

### isRequire和isRequireDefault
用于区分，`getImportRequirePath`的`变量声明`场景中，以下2种语法。
- `require('./modelB')`
- `require('./modelB').default`

### getArguments0
针对函数调用语法，获取第1个参数的值
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
