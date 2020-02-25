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
### HTTP2
  
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
- css2 双飞翼布局 => CSS3 Flex布局: order， 目的：将最重要的HTML放在最顶部优先渲染
- css font
- css 写法规范
  - block / inlink 不需要把默认样式再设置一次（如block  width:100%, inlink  width:50px）
  - 不能使用ID选择器，低端浏览器单个元素不能使用多个class
  - 不能过度使用float/ 去掉空样式标签
 - overflow: hidden => 为什么能填满浮动后父级内容？ 因为：overflow:hidden 会生成BFC（Block Formatting Contect）会让浮动元素重新计算

### Nginx
- 优点：（负载均衡，反向代理，并发处理，低消耗内存资源）
- 缓存策略
### Liunx
 - VMware (Workstation Pro)
  - CentOS 8 liunx版本
    - 安装向导
      - 自定义
      - 安装映像文件
      - CPU核数根据实际CPU核数、内存最小设置为2G、网络设为桥接
      - 将虚拟磁盘存储为单个文件
      - 完成配置后启动VMware进行安装Liunx
      - intel-v 没启动：进入BOIS 启动virtual internel technology
      - pane is dead：CD/DVD 均路径指向Liunx镜像文件
      - 指定硬件位置（必须点击，否则无法下一步）
      - 网络配置：IPV4设为自动（DHCP）保存即显示连接状态
      - root 密码设置为123456等弱密码时点击两次确定即可
 - liunx 常用指令
    - ls 列出文件
    - ll 列出文件与权限
    - :wq 保存并关闭
    - :q! 不保存并关闭
    - :q  直接关闭
    - mv 移动文件
    - tar -xvf 解压
 - liunx 安装插件
   - 切换为root用户
    - yum：CentOS中的Shell前端软件包管理器，默认系统已安装。安装方法：https://rpm.nodesource.com/
    - nodejs 安装步骤：
      - 添加源：curl -sl https://rpm.nodesource.com/setup_11.x | bash -
      - 全局安装：yum install -y nodejs
      - 查看 node -v / npm -v
    - sublime 安装步骤：
      - liunx 内官网下载liunx版本
      - 解压缩 tar -xvvf sublime_text_3_build_**_tar.ba
      - 移动到opt mv sublime_text_3 /opt/
      - 复制快速启动文件到系统菜单目录 cp /opt/sublime_text_3/sublime_text.desktop /usr/share/applications
      - 开打文件 vim /usr/share/applications/sublime_text.desktop
      - 配置快速启动 Exec /Icon 均改为sublime安装目录 => '/opt/sublime_text_3/sublime_text'
    - Nginx（1.16版本）
      - 安装Nginx依赖包：`yum -y install gcc zlib zlib-devel pcre-devel make cmake openssl openssl-devel`
      - liunx 内官网下载Nginx(稳定版)： http://nginx.org/
      - 解压Nginx.**.tar.gz，解压后文件夹内执行`./configure`检查
      - 执行（当make没定义需安装）：`make && make  install`
      - 查看是否安装成功并启动：`/usr/local/nginx/sbin/` => `./nginx`
      - 查看是否启动成功：`ps -ef | grep nginx`
      - 防火墙设置（不设置防火墙的port, 外部无法访问）
        - 查看防火墙：firewall-cmd --list-all
        - 设置外部可访问端口：fire-wall --add-port=80/tcp --permanent
        - 重启防火墙：firewall-cmd --reload
       - `/usr/local/nginx/sbin/nginx` 启动Nginx
       - `/usr/local/nginx/sbin/nginx -s quit` 待Nginx进程处理完毕任务后停止
       - `/usr/local/nginx/sbin/nginx -s stop` 查出Nginx进程再使用kill命令强制杀掉进程
      - 文件目录 `/usr/local/nginx/html`
      - 配置服务器：`/usr/local/nginx/conf/nginx.conf`
        - `worker_processes` 设置CPU核数处理高并发
        - `worker_connections` 最大并发数量
        - http
          - `local_format` 设置日志格式
          - `access_log` 访问日志
          - `keepalive_timeout` 超时时间
          - `gzip` 压缩
          - server
            - 404 页面配置
            - location / （可以另添加其他路由设置：location /img 等）
              - deny 访问权限：禁止指定IP访问或者全部（all）
              - allow 访问权限：允许指定IP访问或者全部（all）
              - proxy_pass 反向代理的到指定的服务器
          - 虚拟主机：将一个服务器主机划分为多个主机称为虚拟主机（`/usr/local/nginx/conf/nginx.conf`中server），以端口区分
      - 根据终端显示对应的页面（PC or Moblie）
### 服务器
> 前端 -> nginx负载均衡 -> Node服务器（过滤后端返回没用的数据）-> redis缓存 -> java服务器 -> 数据库

### Node
  - 以二进制格式传输流 加快请求速度
  - 心跳诊断是否连接中 => setTimeout去Ping
  - Node内存泄漏巨影响性能
  - 高并发处理
    - 每个Node.js进程只有一个主线程在执行程序代码，形成一个执行栈（execution context stack)。
    - 主线程之外，还维护了一个"事件队列"（Event queue）。当用户的网络请求或者其它的异步操作到来时，node都会把它放到Event Queue之中，此时并不会立即执行它，代码也不会被阻塞，继续往下走，直到主线程代码执行完毕。
    - 主线程代码执行完毕完成后，然后通过Event Loop，也就是事件循环机制，开始到Event Queue的开头取出第一个事件，从线程池中分配一个线程去执行这个事件，接下来继续取出第二个事件，再从线程池中分配一个线程去执行，然后第三个，第四个。主线程不断的检查事件队列中是否有未执行的事件，直到事件队列中所有事件都执行完了，此后每当有新的事件加入到事件队列中，都会通知主线程按顺序取出交EventLoop处理。当有事件执行完毕后，会通知主线程，主线程执行回调，线程归还给线程池
