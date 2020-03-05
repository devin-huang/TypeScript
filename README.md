# TypeScript

- 数据类型
  - Sring / Number /Boolean / Array / Object(少用) / any / void（函数无返回值）
  - 枚举： 键值对一一对应 （网络状态码）
  - 元组：
    - 属于数组的一种
    - 多种数据类的数组
    
- 抽象类
  - 提供其他类继承的基类，自身不能被实例化
  - 子类必须实现父类属性或方法
  - 用于定义标准
  - 仅针对类
  
- 接口
  - 定义行为与动作规范
  - 比抽象类强大，包括属性、函数、可索引和类等
  - 属性接口
    ```
      interface FullName {
        firstName: String
        LastName: String
      }
    ```
    - 函数接口
    ```
      interface encnpty {
        (key: String, value: String):String;
      }
    ```
    - 可索引接口（约束数组键值类型）
    ```
      interface userArr {
        [index: Number]: String
      }
    ```
    - 接口扩展（接口继承接口）
    ```
      interface Animal {
        eat(): void
      }
      interface Person extends Animal {
        work(): void;
      }
      Class web inplements Person {
        eat()
        work()
      }
    ```
- 泛型
  - 实现同一函数可返回指定类型
  ```
  function getDate<T>(value: T): T{
    return value
  }
  ```

# Vue 源码

## MVVM
- M => Observe：实例化Vue时使用`Object.defineProperty` && `Object.keys` 遍历data实现数据响应式，Observer是用来给数据添加Dep依赖
- V => Compiler：解析HTML中的指令，根据每个元素节点的指令替换数据或绑定更新函数
- VM => Watcher / Dep： Observe与Compiler之间桥梁，在Compiler解析指令创建watcher并绑定`update`方法
- Dep 用于存储发布订阅的响应依赖，且当所绑定的数据有变更时, 通过`dep.notify()`通知Watcher

> 1.通过Object.defineProperty对Vue实例化后的Data对象遍历绑定为相应数据，get负责添加到Dep对象的订阅者中，set负责触发watcher更新视图

> 2.vue实例以参数传入Compile并编译HTML中的指令（v-text/v-html/{{}}等），对每个节点（元素节点/属性节点/文本节点）绑定的变量、函数通过watcher更新（或初始化）更新视图

> 3. MVVM模式中都会传入Vue实例（vm）贯穿整个流程，通过watcher操作vm(Compile时传入)使第一步与第二步连接起来

> 4. 防止频繁更新视图：异步更新队列，触发方法时将一系列的数据改变操作存入到任务队列中，等同步任务完成后执行异步队列更新视图

[!](mvvm.png)

## 渲染
- $mount
  - 渲染组件真实挂载位置，可用 DOM 或 String 获取
  - 第一步：如果是有定义render函数直接输出第三步结果，否则Template 或 HTML 转为 AST抽象语法树
  - 第二步：optimize 标记静态节点用于优化后续diff算法中会被直接忽略
  - 第三步：generate 将AST 转为 render （Vue最终都转为render, 如果直接使用render性能更优）
  ```js
    render (creatElement) {
      return creatElement('div',
        attr: {
          id: 'id'
        },
        'message'
      )
    }
  ```
- render
  - 将 render 生成 VNode
    - Template / HTML 转换成 render 再使用 createElement 生成 VNode
    - VNode
      ```
      {
        tag: 'div'
        children: [{
            tag: 'span',
            text: 'hello,VNode'
        }]
      }
      ```
      - 用原生 JavaScript 对象描述 DOM, virtual DOM 等于 VNode
      - createElement 通过 render 生成 VNode 函数
        - createEmptyVNode 创建注释节点
        - childrens使用`simpleNormalizeChildren / normalizeArrayChildren`多维VNode数组递归转为一维数组
        - createTextVNode 创建文件节点
        - new VNode() 根据childrens创建元素节点 VNode
      - createComponent Tag标签为组件时通过实例化组件后render生成`VNode`
        - Ctor为组件构造器并通过extend继承于Vue, 实例化组件
        - 缓存机制优化： 已经构造生成的`Vnode组件`会被缓存，当下次引用时会直接返回缓存内 VNode ，无需再次实例
        - 组件生命周期： `installComponentHooks`为组件添加默认生命周期
        - new VNode() 创建以'vue-component'开头的vnode，参数`componentOptions`含有 Ctor / children 等数据
  - 转为 render 后会执行 mountComponent 其实际执行为 updateComponent 初始化、更新组件
  ```
  // update 执行patch的createEle函数真实插入DOM
  updateComponent = () => {
    vm.update(vm._render(), hydrating)
  }
  ```
- update 
  - patch： createPatchFunction 使用柯里化（优点： 预先配置好差异设置，以参数传入从而不累赘函数内部逻辑）
  - createEle 真实创建DOM
    - 初始化时oldVNode参数为 mount 挂载的 DOM，需要使用`emptyNodeAt`转为 VNode, 后续更新时 oldVNode 已经是 VNode
    - createChildren 在 children 为数组先遍历创建空VNode父节点并递归`createEle`，并由 insert 生成真实DOM （所以先渲染子节点再插入到对应的父节点）
    - insert 将生成的 DOM 插入到父节点，再生成完整 DOM 后在mount设置的对象中插入，并删除之前旧的 DOM
- keepAlive
  - 定义变量`vm.cache = []`, 用于存放缓存`组件VNode`
  - 当在重新调用`VNode组件`时会插到`vm.cache`数组最后，当超出数组最大个数会把首个元素删除，有助于合理使用内存
  - 根据vm.cache[key]渲染对应组件，既是使用 render 中的 updateComponent

# Vue 高级处理
内存泄漏
- 外部引用的对象不能直接data中定义（因为最终会通过`Object.defineProperty`绑定全局Vue内，而外部对象嵌套复杂度无法控制）
```html
<div v-html="text">
  <div v-if="false">
    <span v-on="fn"></span>
  </div>
</div>
```
- 解析HTML中的指令并用数组保存（如：`['v-html','v-if', 'v-on']`）, 在项目庞大时重新渲染如果`v-if`条件不满足`v-on`也不会执行就会导致`[watcher, undefined, undefined]`处理数组代码没考虑健壮性时容易产生内存泄漏；因为数组中间出现断层，占内存比较大时该数组中不断累加
- 全局引用（window / document / setTimeout / setInterval / 闭包 / 全局事件）
- Keep-alive 实际把组件VNode保存在Vue实例cache中，当缓存过多VNode会泄漏，所以设置keep-alive设置max值或者合理销毁cache

# Object.defineProperty 处理数组
- `Object.defineProperty`可以对数组进行数据劫持，但只能检测初始化已拥有key的值，所以设置已监听的下标才会触发
- 数组使用push不会触发`Object.defineProperty`
- 数组使用unshift可能会触发`Object.defineProperty`（因为无论插入到数据哪个位置，都会从数组最后添加占位再逐一往前移动到指定下标），每次移动/更新都需要重新监听遍历数组更新下标对应的值
  - 数组监听性能消耗大于对象，所以巧妙定义一个含有数组构造函数的对象 `Object.create(Array.prototype)`
  - 将Array.prototype构造函数原型的push unshift pop shift splice重写扩展（在新增的数组下标添加到observer中并dev.notify）
  - $set 基于`Object.defineProperty` 新增后监听对象更新
- vue 3
  - proxy 优点： 1.无需额外对数据循环（数组）/递归（对象）添加为响应式函数； 2.无需重写数组的方法； 3.除了get/set另增加几种数据响应的方式；

# 优化
### localStorage

> indexedDB 非关系型浏览器数据库 webSQL 关系型浏览器数据库；可有50M存储空间，但属于异步请求就有可能读取时间大于网络请求，所以一般用于离线缓存、广告位
> localstorage 可存储5M，但一般浏览器缓存控制在2.5M中，考虑到低端计算机处理过多会造成页面卡顿加载过慢
> 插件：`localforage.js` 与 `basket.js`
> Webpack生成打包后版本文件依赖关系lcoalstorage.js
> 再根据lcoalstorage.js 版本保存JS代码
> 使用active.js逐一调用JS文件达到localstorage缓存JS/CSS
>
  ```
  // localstorage.js
  let betchObj = {
    runtime: 'runtime.**.js',
    render: 'render.**.js'
  }
  ```
  ```
  // 再保存到lcoalstorage
  localstorage['runtime'] = 'runtime.**.js'
  localstorage['render'] = 'render.**.js'
  // 接着根据JS版本保存JS代码
  let storageObj = {
    'runtime.**.js': '...',
    'render.**.js': '...'
  }
  ```
> active.js
  ```
  export active = (js) => {
    Object.Key(betchObj).forEach(key) {
      if (!js) {
        const __script = document.createElement("SCRIPT");
        __script.type = 'text/javascript';
        __script.src = js;
        document.body.appendChild(__script);
        window.localStorage[key] = storageObj[key]
        window.localStorage[storageObj[key]] = storageObj[storageObj[key]]
      } else if ( window.localStorage[] == obj[] ) {
        eval(window.localStorage[])
      } else {
        const __script = document.createElement("SCRIPT");
        __script.type = 'text/javascript';
        __script.src = js;
        document.body.appendChild(__script);
        window.localStorage[key] = storageObj[key]
        window.localStorage[storageObj[key]] = storageObj[storageObj[key]]
      }
    }
  }
  ```
### HTTP

> HTTP1.1 与 HTTP2.0：

> 1.1采用文本串行按顺序输出，2.0采用并行流的方式传输（1.1时间比2.0慢）

> 1.1高并发球球时最大请求数为6个， 2.0是基于流只建立一个连接即可处理高并发请求
  
### 渲染机制

### CSS技巧
- ftp 1000/60 = 16.67 每16.67毫秒绘制一帧为最优（最新浏览器会每16.67毫秒与帧同步刷新页面，但是旧浏览器是15.8毫秒，这样旧浏览器刷新2次才会执行一次帧大大浪费性能，所以使用requireAnimationFrame减轻当前帧的压力，放到下一帧执行）
- 渲染过程
  - 根据HTML生成DOM Tree / 根据CSS 生成 CSSOM（样式模型）
  - 将DOM和CSSOM合并为渲染树
  - 根据节点在图层上布局元素形状与位置
  - 根据节点在图层上绘制对应的样式
  - 将图层传递给GPU（图像处理器）
  - 将图层合生成图像并处理
    - GPU处理方式 CSS 3D/ Video / canvas / transform / transition
    - 注意： width/ height / offset /client / scrollTop会立刻触发重排，因为这些需要立即返回准确的数值防止重排后排列位置不对，所以最优处理时采用requireAnimationFrame（执行下一帧时触发）统一写、统一读
- css2 双飞翼布局 => CSS3 Flex布局， 目的：将最重要的HTML放在最顶部优先渲染
- css font
- css 写法规范
  - block / inlink 不需要把默认样式再设置一次（如block  width:100%, inlink  width:50px）
  - 不能使用ID选择器，低端浏览器单个元素不能使用多个class
  - 不能过度使用float/ 去掉空样式标签
 - overflow: hidden => 为什么能填满浮动后父级内容？ 因为：overflow:hidden 会生成BFC（Block Formatting Contect）会让浮动元素重新计算
   
### 服务器集群

> 前端 -> nginx负载均衡 -> Node服务器（过滤后端返回没用的数据）-> redis缓存 -> java服务器 -> 数据库

### 移动端自适应尺寸
  - rem计算方式：
    - 1. 以Iphone6宽度375为准, 并设置HTML标签 `1rem = 10px`
    - 2. 假设Iphone6中需要绘制宽度200px的长方形：`20rem` (20rem = 200px)
    - 3. 由上可得在不同尺寸的移动端中改变HTML标签 `1rem = (当前尺寸宽度/375) * 10px` 即可不同尺寸自适应
  - 事件穿透；有上下重叠的元素，当上层元素点击隐藏后会自动点击下层的绑定事件或默认事件 （click 延迟300sm造成）
    - fastClick 插件
    - 阻止默认事件
  - position:flex在部分Iphone中会产生位置错乱（弹出键盘、滚动）
    - 使用relative/absolute定位
### 小程序
  - rpx 
    - 以iphone6为准：`1rpx = 屏幕宽度375/物理像素750 = 0.5px` 从而自适应其他设配宽度
