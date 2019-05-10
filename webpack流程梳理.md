# webpack随记

## assets

指项目中被引用的资源，通常为各种格式的图片和字体文件，当然也可能包含各式各样其他扩展名的文件（.json,.xml等）。
webpack通过file-loader处理资源文件，它会将rules规则命中的资源文件按照配置的信息（路径，名称等）输出到指定目录，并返回其资源定位地址（输出路径，用于生产环境的publicPath路径）

## loaders

- 释义
    webpack 可以使用 loader 来预处理文件。这允许你打包除 JavaScript 之外的任何静态资源。你可以使用 Node.js 来很简单地编写自己的 loader。
- 作用
    处理资源，万物皆可盘；根据设定的资源后缀和解析目录进行资源处理
- 常见
    url-loader  如果文件小于限制，可以返回 data URL
    file-loader 将文件发送到输出文件夹，并返回（相对）URL
    style-loader 将模块的导出作为样式添加到 DOM 中
    css-loader 解析 CSS 文件后，使用 import 加载，并且返回 CSS 代码
    less-loader 加载和转译 LESS 文件
    sass-loader 加载和转译 SASS/SCSS 文件
    postcss-loader 使用 PostCSS 加载和转译 CSS/SSS 文件
- 参考
    [webpack loader文档](https://www.webpackjs.com/loaders)

## plugins

- 释义
    插件(Plug-in,又称addin、add-in、addon或add-on,又译外挂)是一种遵循一定规范的应用程序接口编写出来的程序。其只能运行在程序规定的系统平台下（可能同时支持多个平台），而不能脱离指定的平台单独运行。
- 作用
    扩展webpack的打包流程，按照webpack开放的规范，插件可以订阅特定的过程事件，当webpack流程进度走到特定位置时，会将流程转交给插件处理后，再继续执行
- 参考
    [webpack plugin文档](https://www.webpackjs.com/plugins)

## chunks 和 module

![关系图](https://img.alicdn.com/tps/TB1B0DXNXXXXXXdXFXXXXXXXXXX-368-522.jpg)

- chunk
  代码分割后的产物，也就是按需加载的分块

- module
  每个import-from,require()语法所引用的模块

## tapable库

- 作用
    定义规范接口的调用方式及调度
        串行
        并行
        异步串行
        异步并行

## webpack的常见流程

![流程图](https://img.alicdn.com/tps/TB1GVGFNXXXXXaTapXXXXXXXXXX-4436-4244.jpg)

- `compile`：开始编译
- `make`：compiler对象开始从entry进行模块分析以及依赖分析
- `build-module`：compilation对象开始构建模块。这个时间点模块还没开始构建，入口点已经被分析完，依赖已经分析完。使用loader加载文件并build模块
- `normal-module-loader`：compilation对象对每个模块构建并载入loader信息。这个节点在每个模块载入loader信息触发。
    `program`：开始对AST进行遍历，当遇到require时触发call require事件
- `after-compile`： 完成构建任务
- `seal`：所有依赖build完成，compilation对象开始封装构建结果，开始对chunk进行优化（抽取公共模块、加hash等），生成最后的JS
- `emit`：把各个chunk输出到结果文件
- `after-emit`：把各个chunk输出到结果文件

## webpack语句执行了什么

  ``` shell
  webpack --config=webpack.config.js -w
  ```

  1. 构建compiler对象
    `let compiler = new Webpack(options)`

  2. 注册NodeEnvironmentPlugin插件
    `new NodeEnvironmentPlugin().apply(compiler);`

  3. 挂在options中的基础插件，调用WebpackOptionsApply库初始化基础插件
  
      ``` js
      if (options.plugins && Array.isArray(options.plugins)) {
          for (const plugin of options.plugins) {
              if (typeof plugin === "function") {
                  plugin.apply(compiler);
              } else {
                  plugin.apply(compiler);
              }
          }
      }
      compiler.hooks.environment.call();
      compiler.hooks.afterEnvironment.call();
      compiler.options = new WebpackOptionsApply().process(options, compiler);
      ```
  
  4. run 开始编译

      ``` js
          if (firstOptions.watch || options.watch) {
              const watchOptions = firstOptions.watchOptions || firstOptions.watch || options.watch || {};
              if (watchOptions.stdin) {
                  process.stdin.on("end", function(_) {
                      process.exit(); // eslint-disable-line
                  });
                  process.stdin.resume();
              }
              compiler.watch(watchOptions, compilerCallback);
              if (outputOptions.infoVerbosity !== "none") console.log("\nwebpack is watching the files…\n");
          } else compiler.run(compilerCallback);
      ```

## run过程

 1. 在run的过程中，已经触发了一些钩子：beforeRun->run->beforeCompile->compile->make->seal (编写插件的时候，就可以将自定义的方挂在对应钩子上，按照编译的顺序被执行)
 2. 构建了关键的 Compilation对象

- this.compile()中创建了compilation

  ``` js
  this.hooks.beforeRun.callAsync(this, err => {
    ...
    this.hooks.run.callAsync(this, err => {
      ...
      this.readRecords(err => {
        ...
        this.compile(onCompiled);
      });
    });
  });
  ```

## Compilation

- 作用
  1. 负责组织整个打包过程，包含了每个构建环节及输出环节所对应的方法，可以从图中看到比较关键的步骤，如 addEntry() , _addModuleChain() , buildModule() , seal() , createChunkAssets() (在每一个节点都会触发 webpack 事件去调用各插件)。
  
  2. 该对象内部存放着所有 module ，chunk，生成的 asset 以及用来生成最后打包文件的 template 的信息。

## 编译构建

- 待补充
  
## 打包输出

  在所有模块及其依赖模块 build 完成后，webpack 会监听 seal 事件调用各插件对构建后的结果进行封装，要逐次对每个 module 和 chunk 进行整理，生成编译后的源码，合并，拆分，生成 hash。同时这是我们在开发时进行代码优化和功能添加的关键环节。

  ``` js
  compile(callback) {
    const params = this.newCompilationParams();
    this.hooks.beforeCompile.callAsync(params, err => {
      ...
      this.hooks.compile.call(params);
      const compilation = this.newCompilation(params);
      this.hooks.make.callAsync(compilation, err => {
              ...
        compilation.finish();
        compilation.seal(err => {
                  ...
          this.hooks.afterCompile.callAsync(compilation, err 
              ...
            return callback(null, compilation);
          });
        });
      });
    });
  }
  ```