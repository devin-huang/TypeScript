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
- M Observe：实例化Vue时使用Object.defineProperty && Object.keys遍历data实现数据响应式
- V Compiler：解析HTML中的指令，根据每个元素节点的指令替换数据或绑定更新函数
- VM Watcher / Dep： Observe与Compiler之间桥梁，在Compiler解析指令创建watcher并绑定update方法；Dep用于存储发布订阅的响应依赖，且当所绑定的数据有变更时, 通过dep.notify()通知Watcher

[!](mvvm.png)

## 渲染
- $mount
  - DOM真实挂载位置，可用 DOM 或 String 获取
- render
  - Template 或 HTML 转为 render （Vue最终都转为render, 如果直接使用render性能更优）
  - 将 render 生成 VNode
    - VNode
      - 用原生 JavaScript 对象描述 DOM, virtual DOM 等于 VNode
      - componentOptions 组件 VNode 的配置
      - createElement 把 render 生成 VNode 函数
        - createEmptyVNode 创建注释节点
        - simpleNormalizeChildren / normalizeArrayChildren 将多维数组转为一维数组
        - createTextVNode 创建文件节点
        - new VNode() 实例化有元素VNode
      - createComponent 把 render 生成 VNode 组件
  - 转为 render 后会执行 mountComponent 其实际执行为 updateComponent 初始化、更新组件
  ```
  // update 执行patch的createEle函数真实插入DOM
  updateComponent = () => {
    vm.update(vm._render(), hydrating)
  }
  ```
  - update 
  - patch
  - keepAlive
  - vuex
  - v-model
  - 异步组件
