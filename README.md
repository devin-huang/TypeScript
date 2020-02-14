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
- M Observe：实例化Vue时使用Object.defineProperty && Object.keys遍历data实现数据响应式，Observer是用来给数据添加Dep依赖
- V Compiler：解析HTML中的指令，根据每个元素节点的指令替换数据或绑定更新函数
- VM Watcher / Dep： Observe与Compiler之间桥梁，在Compiler解析指令创建watcher并绑定update方法
- Dep 用于存储发布订阅的响应依赖，且当所绑定的数据有变更时, 通过dep.notify()通知Watcher

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
      - componentOptions 组件 VNode 的配置
      - createElement 通过 render 生成 VNode 函数
        - createEmptyVNode 创建注释节点
        - childrens使用simpleNormalizeChildren / normalizeArrayChildren 多维数组递归转为一维数组
        - createTextVNode 创建文件节点
        - new VNode() 根据childrens创建元素节点 VNode
      - createComponent 通过 render 生成 VNode 组件
        - 构造器Ctor是继承Vue构造器
        - 缓存机制优化： 已经构造生成的 Vnode 组件会被缓存，当下次引用时会直接返回缓存内 VNode ，无需再次实例
        - childrens使用simpleNormalizeChildren / normalizeArrayChildren 多维数组递归转为一维数组
        - 组件生命周期： 将组件内置 hook 合并，使每个新生成组件有生命周期
        - componentOptions 含有 Ctor / children 等重要数据用于生成组件
        - new VNode() 根据childrens创建元素节点 VNode 组件
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
    - 初始化时oldVNode参数为 mount 挂载的 DOM，需要使用 emptyNodeAt 转为 VNode, 后续更新时 oldVNode 已经是 VNode
    - createChildren 在 children 为数组先遍历并递归 createEle，并由 insert 生成真实DOM （所以先渲染子节点再插入到对应的父节点）
    - insert 将生成的 DOM 插入到父节点，再生成完整 DOM 后在mount设置的对象中插入，并删除之前旧的 DOM

- keepAlive
  - 定义变量`vm.cache = []`, 用于存放缓存`组件VNode`
  - 当在重新调用`VNode组件`时会插到`vm.cache`数组最后，当超出数组最大个数会把首个元素删除，有助于合理使用内存
  - 根据vm.cache[key]渲染对应组件，既是使用 render 中的 updateComponent
- vuex
  - 多个组件共享数据
  - 
- v-model

