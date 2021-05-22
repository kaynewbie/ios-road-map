# iOS Road Map

面试主要分为三块内容：**项目**、**iOS底层原理**和**通用编程技能**。

[toc]

## 1. 项目

### 亮点项目



## 2. iOS底层原理



### Block

- Block 数据结构
- 捕获变量
- __block (原始类型、对象类型)
- 内存管理方式（block的内存管理、捕获变量的内存管理）

[block 源码解释内存管理文章](https://www.jianshu.com/p/b3c48a28bed3)

[block Clang 源码解析](https://clang.llvm.org/docs/Block-ABI-Apple.html)



### 内存管理

* 描述引用计数

* MRC 和 ARC

* Autorelease & Autoreleasepool

  * Autoreleasepool 和 run loop 关系

* 参考

* - [探究 autorelease 和 autoreleasepool](https://www.jianshu.com/p/97dd0ae27108)
  - [autoreleasepoll 内存分配和释放](http://tutuge.me/2015/03/17/what-is-autoreleasepool/)
  - [源码解析 autoreleasepool](https://draveness.me/autoreleasepool/)
  - [Clang 有详细的 ARC 文档](https://clang.llvm.org/docs/AutomaticReferenceCounting.html#background)



### Runtime

* 类的结构模型，了解 class_data_bits_t, class_rw_t, class_ro_t

* 方法调用

  * 方法结构
  * 消息发送过程
  * 消息转发

* 关联对象的存储

* 程序启动做了什么事情 （加载镜像）

* 分类的加载时机和方法重载

* +load vs. +initialize

* MetaClass 设计

* - 区分类方法和实例方法的存储：前者存放在元类；后者存放在类
  - 消息发送逻辑能够复用

* 参考
  * [深入解析 ObjC 中方法的结构](https://draveness.me/method-struct/)
  * objc4 源码 （建议多看这个）



### Runloop

* 对象类型 (RunLoop, RunLoopMode, Source, Timer, Observer)
* Runloop 和线程关系
* Runloop 场景比较多，要多讲讲
* 参考
  * [深入理解 run loop](https://blog.ibireme.com/2015/05/18/runloop/)



### KVO

* 实现原理
* 类型重写
* 回调方法的执行线程
* 观察同一对象的同一属性，同时使用自定义的 KVO和系统KVO会有什么效果？



### UI渲染

* 大概了解渲染框架结构 (Core Animation, Core Graphics, Open GL/Metal)

* 了解整个渲染 Pipeline，知道每个过程做些什么

* [iOS界面渲染流程分析](https://www.jianshu.com/p/39b91ecaaac8)，渲染核心原理围绕：前后帧缓存，Vsync信号，CADisplayLink

  * CPU 创建视图，布局，构建视图和图层关系，查询 drawRect

  - - 布局计算（自动布局）
    - 视图创建
    - drawRect:
    - 解压图片
    - 图层打包

  - CPU将图层打包，提交到渲染服务器(Open GL & GPU)

  - 渲染服务器生成纹理并着色，生成前后帧缓存，并根据Vsync信号切换缓存

  - GPU 渲染

* 显示图片（[详解图片加载和优化方案](https://www.jianshu.com/p/7d8a82115060)）

  - 显示图片分三步：加载(UIImage)，解码（生成位图），渲染
  - 开发常规只接触加载，解码和渲染由 UIKit 完成
  - 列表控件中展示多个图片，会造成内存和CPU陡升，优化方案：
    - 降采样（因为界面显示的图片大小通常小于真实图片大小，所以加载缩略图能降低内存占用）
    - 预处理和子线程解码（prefetchItem 代理方法中，异步解码图片）
    - 使用Image Assets Catalogs，因为这是iOS自带的资源加载方案，便于优化

* 图片高级处理系列

  * [图形理论](https://juejin.cn/post/6847902216238399496#heading-12)
  * [图片编解码](https://juejin.cn/post/6847902216540389390)
  * [图片处理实践](https://juejin.cn/post/6846687599591948301)

* [离屏渲染](https://zhuanlan.zhihu.com/p/72653360)

- - 现象：GPU在frame buffer之外额外开辟的内存
  - 原因：因为视图层级问题，无法一次性渲染整个层级，“画家算法”（从下往上一层一层渲染），导致有些元素无法一次性渲染生效，需要等整个层级渲染结束后，再作用，所以需要额外的空间缓存
  - 通过layer的阴影、圆角、mask等属性引发

* 参考
  * [图形图像渲染原理](http://chuquan.me/2018/08/26/graphics-rending-principle-gpu/)
  * [iOS 图像渲染原理](http://chuquan.me/2018/09/25/ios-graphics-render-principle/)

* FastImageCache

- - 解决什么需求：一款应用主要的性能瓶颈在于图片加载。按照传统的方式，从磁盘中单张读取图片效率太低，尤其是在列表控件中。

  - 怎么做

  - - 将图片持久化到磁盘
    - 用 LRU 策略自动管理图片过期
    - 模型化？？？？怎么理解

  - 需求场景

  - - 描述：照片浏览，需要加载图片并滑动浏览

    - 问题：从磁盘中加载压缩过的图片，并展示到屏幕上，是个昂贵的开销

    - 解决的核心技术：

    - - 内存映射
      - 图片解压，保存解压后的图片（所以这个库更适用于小图片，否则磁盘会占用很大）
      - 字节对齐



### 事件的传递和响应

* UIResponder 子类 (UIView, UIViewController, UIApplication)

* 描述传递和响应两个过程

* 怎么扩大按钮点击区域？

  * 子类化，重写 pointInside:withEvents:

* Responder vs. GestureRecognizer vs. UIControl

* - GestureRecognizer 实现原理：内部有 UIGestureRecognizerSubclass，也是通过 touches 方法判定手势是否成功
  - hitTest 过程中添加 gesture recognizer 信息
  - 优先级：UIControl(系统) > GestureRecognizer > Responder



### 通知 (NSNotification)

- 几个类型

- - NSNotification
  - NSNotificationCenter
  - NSNotificationQueue：维护队列，依赖 run loop，适当时机发送通知

- 数据结构

- - 如何存通知

  - - name & object 两个维度
    - Observation 链表结构

- 注册监听

- 发送通知

- 移除监听



### KVC



## 3. 通用编程技能



### 开源库



### 多线程和锁

* iOS中常用的多线程方案 (GCD & NSOperation)，对这两套API要比较熟悉
* 具体场景给出解决方案
* 锁的种类
* [@synchronized](http://yulingtianxia.com/blog/2015/11/01/More-than-you-want-to-know-about-synchronized/)



### 网络

* HTTP 请求的完整链路
  * URL
  * Method
  * Request Header & Response Header
  * Status Code
* HTTPS 加密过程
* 鉴权（OAuth）
* [TCP 握手和挥手过程](https://blog.csdn.net/qzcsu/article/details/72861891)
* HTTP 版本更新



### 架构、设计模式

* 组件化方案，路由设计

* MVC、MVVM、MVP

* 包管理（CocoaPods 做了什么）

* App 组织

  - 网络请求 (NSURLSession, AFNetworking

  - 页面如何组织 (

  - - 架构模式
    - 代码规范
    - 布局
    - VC 派生结构
    - SB, Xib, Code

  - 数据持久化

  - - Plist
    - UserDefault
    - Archive
    - sqlite

  - 动态部署

  - - 热更新
    - 降级方案

  * 团队管理
    * 收集用户数据
    * 组件化和组件通信
    * CI、CD



### 算法

* 数据结构 （数组、链表、栈、队列、二叉树）