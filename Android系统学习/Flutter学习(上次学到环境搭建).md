# Flutter
>电子书地址：https://book.flutterchina.club/

>源码地址：https://github.com/wendux/flutter_in_action_source_code

>原生Android程序的优缺点：
* 优势：
可访问平台全部功能（GPS、摄像头）；
速度快、性能高、可以实现复杂动画及绘制，整体用户体验好；
* 主要缺点：
平台特定，开发成本高；不同平台必须维护不同代码，人力成本随之变大；
内容固定，动态化弱，大多数情况下，有新功能更新时只能发版；
>原生程序不能满足现代移动互联网的需求：动态化和开发成本

## 跨平台技术（框架）
### H5+原生（Cordova、Ionic、微信小程序）
>将需要动态变化的内容通过H5实现，使用原生的网页加载控件WebView（IOS的WKWebView）来加载。这种开发模式称为混合开发。

>典型代表：Cordova、Ionic 和微信小程序。

>H5代码是运行在WebView中，而WebView实质上就是一个浏览器内核，其JavaScript依然运行在一个权限受限的沙箱中，所以对于大多数系统能力都没有访问权限，如无法访问文件系统、不能使用蓝牙等。依赖于WebView的用于在JavaScript与原生之间通信并实现了某种消息传输协议的工具称之为WebView JavaScript Bridge, 简称 JsBridge。混合应用无非就是在第一步中预先实现一系列API供JavaScript调用，让JavaScript有访问系统的能力。

>优点是动态内容是H5，web技术栈，社区及资源丰富，缺点是性能不好，对于复杂用户界面或动画，WebView不堪重任。
### JavaScript开发+原生渲染 （React Native、Weex、快应用）
>React Native (简称RN)是Facebook于2015年4月开源的跨平台移动应用开发框架，是Facebook早先开源的JS框架 React 在原生移动应用平台的衍生产物，目前支持iOS和Android两个平台。RN使用Javascript语言，类似于HTML的JSX，以及CSS来开发移动应用，因此熟悉Web前端开发的技术人员只需很少的学习就可以进入移动应用开发领域。

>Weex是阿里巴巴于2016年发布的跨平台移动端开发框架，思想及原理和React Native类似，最大的不同是语法层面，Weex支持Vue语法和Rax语法，Rax 的 DSL(Domain Specific Language) 语法是基于 React JSX 语法而创造。

>快应用是华为、小米、OPPO、魅族等国内9大主流手机厂商共同制定的轻量级应用标准，目标直指微信小程序。
### 自绘UI+原生(QT for mobile、Flutter)
>通过在不同平台实现一个统一接口的渲染引擎来绘制UI，而不依赖系统原生控件，所以可以做到不同平台UI的一致性。如果涉及其它系统能力调用，依然要涉及原生开发。

>和QT mobile做一个对比：
* 生态：从Github上来看，目前Flutter活跃用户正在高速增长。从Stackoverflow上提问来看，Flutter社区现在已经很庞大。Flutter的文档、资源也越来越丰富，开发过程中遇到的很多问题都可以在Stackoverflow或其github issue中找到答案。
* 技术支持：现在Google正在大力推广Flutter，Flutter的作者中很多人都是来自Chromium团队，并且github上活跃度很高。另一个角度，从今年上半年Flutter频繁的版本发布也可以看出Google对Flutter的投入的资源不小，所以在官方技术支持这方面，大可不必担心。
* 开发效率：Flutter的热重载可帮助开发者快速地进行测试、构建UI、添加功能并更快地修复错误。在iOS和Android模拟器或真机上可以实现毫秒级热重载，并且不会丢失状态。这真的很棒，相信我，如果你是一名原生开发者，体验了Flutter开发流后，很可能就不想重新回去做原生了，毕竟很少有人不吐槽原生开发的编译速度。
### 三者对比
技术类型|UI渲染方式|性能|开发效率|动态化|框架代表
---|---|---|---|---|---
H5+原生|WebView渲染|一般|高|支持|Cordova、Ionic
JavaScript+原生渲染|原生控件渲染|好|中|支持|RN、Weex
自绘UI+原生|调用系统API渲染|好|Flutter高, QT低|默认不支持|QT、Flutter

## 简介
>使用dart云语言开发。跨平台（使用Skia作为2D渲染引擎）、高性能（Dart在JIT模式与js性能基本持平，在AOT模式比js强很多；使用自己的渲染引擎，不需像RN在js和Native之间通信，在滑动和拖动场景有明显优势）
### JIT和AOT
>AOT: Ahead of time, 提前编译，典型代表是c/c++（静态编译）开发应用，在执行前先编译成机器码。

>JIT: Just-in-time,即时编译，如js，python（动态解释）等，边翻译边执行。

>也有些语言是都支持AOT和JIT，比如Java和Python。
### Dart语言
* 开发效率高
>Dart运行时和编译器支持Flutter的两个关键特性的组合：
基于JIT的快速开发周期：Flutter在开发阶段采用，采用JIT模式，这样就避免了每次改动都要进行编译，极大的节省了开发时间；
基于AOT的发布包: Flutter在发布时可以通过AOT生成高效的ARM代码以保证应用性能。而JavaScript则不具有这个能力。

* 高性能
>Flutter旨在提供流畅、高保真的的UI体验。为了实现这一点，Flutter中需要能够在每个动画帧中运行大量的代码。这意味着需要一种既能提供高性能的语言，而不会出现会丢帧的周期性暂停，而Dart支持AOT，在这一点上可以做的比JavaScript更好。

* 快速内存分配
>Flutter框架使用函数式流，这使得它在很大程度上依赖于底层的内存分配器。因此，拥有一个能够有效地处理琐碎任务的内存分配器将显得十分重要，在缺乏此功能的语言中，Flutter将无法有效地工作。当然Chrome V8的JavaScript引擎在内存分配上也已经做的很好，事实上Dart开发团队的很多成员都是来自Chrome团队的，所以在内存分配上Dart并不能作为超越JavaScript的优势，而对于Flutter来说，它需要这样的特性，而Dart也正好满足而已。

* 类型安全
>由于Dart是类型安全的语言，支持静态类型检测，所以可以在编译前发现一些类型的错误，并排除潜在问题，这一点对于前端开发者来说可能会更具有吸引力。与之不同的，JavaScript是一个弱类型语言，也因此前端社区出现了很多给JavaScript代码添加静态类型检测的扩展语言和工具，如：微软的TypeScript以及Facebook的Flow。相比之下，Dart本身就支持静态类型，这是它的一个重要优势。

* Dart团队就在你身边
>看似不起眼，实则举足轻重。由于有Dart团队的积极投入，Flutter团队可以获得更多、更方便的支持，正如Flutter官网所述“我们正与Dart社区进行密切合作，以改进Dart在Flutter中的使用。例如，当我们最初采用Dart时，该语言并没有提供生成原生二进制文件的工具链（这对于实现可预测的高性能具有很大的帮助），但是现在它实现了，因为Dart团队专门为Flutter构建了它。同样，Dart VM之前已经针对吞吐量进行了优化，但团队现在正在优化VM的延迟时间，这对于Flutter的工作负载更为重要。
## 框架架构
### Flutter Framework
* 底下两层（Dart UI层）：Foundation, Animation, Painting, Gestures。对应dart.ui包，是Flutter引擎暴露的底层UI库，提供动画、手势和绘制能力。
* Rendering层: 抽象的布局层，依赖于dart UI层，Rendering层会构建一个UI树，当UI树有变化时，会计算出有变化的部分，然后更新UI树，最终将UI树绘制到屏幕上，这个过程类似于React中的`虚拟DOM`。Rendering层可以说是Flutter UI框架最核心的部分，它除了确定每个UI元素的位置、大小之外还要进行坐标变换、绘制(调用底层dart:ui)。
* Widghts层：基础组件库，在此基础上还提供了Material 和Cupertino两种视觉风格的组件库。Flutter开发的大多数场景，只是和这两层打交道。
### Flutter Engine
* 纯 C++实现的 SDK，其中包括了 Skia引擎、Dart运行时、文字排版引擎等。在代码调用 dart:ui库时，调用最终会走到Engine层，然后实现真正的绘制逻辑。
## 开发环境



