### 做搭建-如何进行模块化设计

![页面搭建](/assets/page-build.png)

有时候组件和模块是一个不太好区分的概念，特别是在前端领域，大家对于功能、方法的划分粒度都有各自的理解。

在Angular里，module是一些类、函数或者值的集合。component则更像一个区块，比如某一段负责渲染模板到html的代码，多个component可以组成一个app。module之间可以相互引用，module也可以被component引用。

其实在Web标准里，也可以看到一些相同的思路，比如ESM(EcmaScript modules)和Web Component，分别归属于ECMAScript规范和W3C规范。在极简的场景下，module负责提供一些方法，component负责渲染到页面上。

比如一个React Component发布到npm就是一个React module。在搭建的场景下，组件和模块有一定的差异性：

* 组件：一个react component或者node.js module，在搭建体系下，都是一个组件；
* 模块：模块是组成页面的最小结构单位，一个模块会包含模板（jsx、xtpl等等）、样式（css）、脚本逻辑、数据描述（data-schema）、模拟数据（mock-data）、表单描述（form-schema）、数据逻辑（faas-function）、依赖的描述（deps）等，一个模块甚至可以承接一个页面的逻辑。
 * 组件加上数据描述，就是一个模块；

以下是一个模块的生产流程：

![模块生产流程](/assets/page-build1.png)

### 一、从组件到模块

原则上所有npm上的module，都应该低成本的引用。实际上，低成本本身确是一个很大的挑战。从研发到线上渲染，必须思考和解决下面的问题：

1、模块如何引用组件？
2、如何合理引入npm的资源？
3、如何引入和编译本地文件系统里的文件？
4、如何渲染一个页面？

### 如何解决研发问题

#### 1.1 模块、组件之间的引用

模块加载器是前端工程里非常重要的一部分。加载器本质上就是通过一个去中心化的引用关系来描述细粒度模块之间的依赖，开发者使用模块A的时候，只需要声明依赖模块A，而模块A本身依赖了模块B，是在模块A自己内部管理的。在最终浏览器端，加载器把一个个引用关系进行合并，最后用合并后的引用关系表来生成一串combo url。

引用关系实际对应的就是一个schema文件

```js
// xxx.json
{
  modules: {
    one: {
      async: false,
      fullpath: './one.js'
    },
    two: {
      async: false,
      fullpath: './two.js',
      requires: [ 'one' ]
    }
  }
}
```

因为有一份这样的依赖描述，也可以通过同名取最新版本的策略，来做到同名模块只会加载一次，一定程度上解决js体系过大的问题。同时，如果开发者有能力或者有一个好的策略对combo url进行管理，是可以做到多页面之间共享资源缓存的，提升全链路的用户体验。

这种依赖关系的设计方案，可以在服务端去重输出代码bundle。

![模块、组件之间的引用](/assets/fe-module.png)

