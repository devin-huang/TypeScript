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
