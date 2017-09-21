# iOS组件化

主要用CocoaPods来实现组件化。

## 组件化有哪些方式？
- Cocoapods
  不多赘述

- Projects

- Github黑科技

## 为什么要组件化？
- Swift编译速度问题
	Swift项目写过的都懂，一段时间后编译奇慢无比。可以仿照Uber开启`whole module optimization`。开启过后编译会以module为粒度。

- 各模块解耦
  - 便于Debug
  - 便于后续扩展

## 项目应该分为哪些模块？
- XXUIKit
  里面主要放工程中定义的一些颜色、UI有关的宏、扩展等等。
- 独立的UI控件
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

### 各模块确定依赖关系
  podspec中找到这个字段进行相关配置即可
```
s.dependency "JSONKit", "~> 1.4"
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
- class：用open修饰
- property: 用public修饰
- function: 用open修饰

### 配置资源
- 设置源码位置
podspec中编辑一下，选择你的源码文件夹
```
s.source_files  = "Classes", "*.{h,m,swift}"
```
- 设置图片等静态资源
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
- 本地模块
```
pod 'XXUIKit',:path => 'xxxx'
```
- git上的模块
	podspec必须放在根目录
```
pod 'XXUIKit',:git => 'xxxx'
```
## 我还要满足产品的其他很多要求，点击统计怎么办？
我们可以在独立的模块中回调出来，在胶水类中进行处理，回调方式大家都明白
- block

- delegate

- notification


