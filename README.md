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
- vuex
  - 多个组件共享数据
  - 
- v-model

# Vue 分析
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
- 全局引用（window / document / setTimeout / setInterval / 闭包）

# Object.defineProperty 处理数组
- Object.defineProperty 可以对数组进行数据劫持，但只能检测初始化已拥有key的值，所以设置已监听的下标才会触发
- 数组使用push不会触发`Object.defineProperty`
- 数组使用unshift可能会触发`Object.defineProperty`（因为无论插入到数据哪个位置，都会从数组最后添加占位再逐一往前移动到指定下标），每次移动都需要重新遍历数组更新下标对应的值
  - 数组监听性能消耗大于对象，所以改用一个含有数组构造函数的对象 `Object.create(Array.prototype)`
  - Vue将Array.prototype构造函数原型的push unshift pop shift splice重写
