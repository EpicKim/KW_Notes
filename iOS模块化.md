# iOS开发模块化
> 从第一代码农写下第一行代码开始到上个世纪的80年代的软件危机，码农一直在考虑一个问题，怎么让写代码容易。在PC软代时代就已经有解决这个问题的法宝－－组件化。当然那时候不是那么叫的，是通过两个原则来规范这个问题的，这两个原则就是：内聚性和耦合性。在iOS开发中，我们换一个称呼，叫做模块化。

## 模块化有哪些方式？
- **Cocoapods**

  管理库事实上的标准，也是这篇文章实现模块化用到的主要技术。

- **Projects**

参考[firefox-Swift](https://github.com/mozilla-mobile/firefox-ios)的做法,用workspace管理多个project

![Firefox项目](http://chuantu.biz/t6/58/1506058186x3728889954.png)

新增一个module就新建一个project，类型选`Cocoa Touch framework`，编译过后主project `build phase`中引用即可。




- **Carthage**

用着难受、反人类，所以这里不讨论。

## 为什么要模块化？
- **Swift编译速度问题**

  Swift项目写过的都懂，一段时间后编译奇慢无比。可以仿照Uber开启`whole module optimization`。开启过后编译会以module为粒度。

  比如UI要换个颜色，我们只需要修改`XXUIKit`里的颜色配置，编译也只会重新编译这个Module，不用等好几分钟的时间。

- **各模块解耦**
  - 便于Debug
  - 便于后续扩展

## 项目应该分为哪些模块？
- **XXUIKit**

  里面主要放工程中定义的一些颜色、UI有关的宏、扩展等等。
- **独立的UI控件**

  常用的包括下拉控件、相册、搜索（历史记录存储、展示）、下载等等都可以作为独立项目维护，所用的资源也可以建立相关的bundle、assets等来管理。
- 各类与业务无强相关的算法

  算法都独立出来便于多平台共享。

## 我们该怎么做？

### 创建你的Pods私有仓库
  一句命令很轻松就可以完成
```
pod spec create 你的模块名
```
我们会得到一个xx.podspec，这是我们的Pods库管理文件，编辑即可配置我们的库。

### 各模块建立依赖关系

比如我某个UI控件想用`Autolayout`，那么我们可以依赖`SnapKit`

  podspec中找到这个字段进行相关配置即可
```
s.dependency "Snapkit", "~> 2.0"
```
也可以引用其他本地的pods库，比如我们要在某个UI控件中使用项目统一的主题色。
```
s.dependency "XXUIKit"
```
在代码中引用只需import相关库即可
```
import XXUIKit
```
### 模块中的开放类、方法、属性添加访问修饰符
封装成库后，项目中再调属于跨模块调用，默认的internal访问不到，所以我们得把原来的类改写一下。
- **class**：用open修饰
- **property**: 用public修饰
- **function**: 用open修饰

### 配置资源
- **设置源码位置**

podspec中编辑一下，选择你的源码文件夹
```
s.source_files  = "Classes", "*.{h,m,swift}"
```
- **设置图片等静态资源**

还是podspec
```
s.resources = "*.{sks,xcassets,atlas,png,bundle}"
```
代码中引用得注意一下,以获取sks资源为例，XXX填库内源码的类名
```
let path = Bundle(for: XXXX.self).path(forResource: file, ofType: "sks")
        
let sceneData: Data?
do {
    sceneData = try Data(contentsOf: URL(fileURLWithPath: path!), options: .mappedIfSafe)
} catch _ {
    sceneData = nil
}
```

### 项目中导入独立模块
- **本地模块**
```
pod 'XXUIKit',:path => 'xxxx'
```
- **git上的模块**

  podspec必须放在根目录
```
pod 'XXUIKit',:git => 'xxxx'
```
非常独立的模块可以都放到git服务器上作为独立项目维护，然后用Cocoapods导进来。

## 我还要满足产品的其他很多要求，点击统计怎么办？

我们可以在独立的模块中回调出来，在胶水类中进行处理，回调方式大家都明白
- **block**

- **delegate**

- **notification**

## 模块间相互通信怎么处理？

一个 APP 有多个模块，模块之间会通信，互相调用，例如微信读书有 书籍详情 想法列表 阅读器 发现卡片 等等模块，这些模块会互相调用，例如 书籍详情要调起阅读器和想法列表，阅读器要调起想法列表和书籍详情，等等，一般我们是怎样调用呢，以阅读器为例，会这样写：
```
- (void)gotoDetail {
    WRBookDetailViewController *detailVC = [[WRBookDetailViewController alloc] initWithBookId:self.bookId];
     [self.navigationController pushViewController:detailVC animated:YES];
}
```
看起来挺好，这样做简单明了，没有多余的东西，项目初期推荐这样快速开发，但到了项目越来越庞大，这种方式会有什么问题呢？显而易见，每个模块都离不开其他模块，互相依赖粘在一起成为一坨：
这样揉成一坨对测试/编译/开发效率/后续扩展都有一些坏处，那怎么解开这一坨呢。很简单，按软件工程的思路，下意识就会加一个中间层：
![中间件](http://blog.cnbang.net/wp-content/uploads/2016/03/component2-1024x597.png )
方案有很多，个人偏向蘑菇街的 [MGJRouter](https://github.com/meili/MGJRouter)

`MGJRouter`及几种方案的对比可以参考[组件化架构漫谈](https://juejin.im/entry/57ee1efe2e958a00554132bb)

