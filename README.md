## 前言

作为 Android 开发者，不知你是否也有这样的体验，随着项目变得越来越大，各种不同圆角的 shape，不同透明度的 color，不同大小的阴影效果，它们使资源文件越来越多

我认为造成这种问题的原因有两个：一个是产品设计的不规范，整个 app 没有统一的设计风格；第二个便是开发者在开发过程中编码的不规范



Android Dev Summit '19  有一场关于 Style 与 Theme 的演讲，它的 [中文字幕视频在这里](https://www.bilibili.com/video/BV1uJ411h7jh)

我为你整理了每个主题所在的位置

| **时间** |     **内容**     |
| :------: | :--------------: |
|  02：14  | Styling vs Theme |
|  08：55  |  Theme Overlay   |
|  12：36  |      Color       |
|  17：35  |  使用及三个技巧  |
|  24：00  |  Material 颜色   |
|  28：06  |  Material 排版   |
|  30：07  |  Material 形状   |
|  34：41  |    Dark Theme    |

在视频下方的评论区，点击相应时间即可跳转到指定内容

![视频下方评论区](https://gitee.com/flywith24/Album/raw/master/img/20200627112923.png)

与之对应的有一个 Styling 的系列文章，我最近翻译成了中文

- [【译】Android Styling 1： Themes vs Styles](https://juejin.im/post/5eead9196fb9a058734e3b03)


- [【译】Android Styling 2： 常用主题属性](https://juejin.im/post/5eec07416fb9a058835d0306)


- [【译】Android Styling 3： 使用主题和主题属性的优势](https://juejin.im/post/5eed67b4f265da02a642bd57)


- [【译】Android Styling 4： 主题实战](https://juejin.im/post/5eeff86cf265da02e8177eba)



本文整理了视频与文章中的内容，介绍在开发过程中，我们应如何利用 theme 与 style 更优雅地管理资源文件，并提供了很多实用的技巧，在标题中找到技巧相关的查看即可

并且提供了演示 demo，效果如下

![demo](https://user-gold-cdn.xitu.io/2020/6/24/172e4f36ad93089b?w=429&h=957&f=gif&s=1172512)



## 理解 Style 与 Theme 的区别

这部分内容在视频的 02.14 处

Style 是 View 属性的集合，可以将 Style 视为  Map<**View** Attribute, Resource>，其中 key 为 View 的属性，value 为资源



![sytle key 为 View Attributes](https://gitee.com/flywith24/Album/raw/master/img/20200624141439.png)

![style value 为 Resources](https://gitee.com/flywith24/Album/raw/master/img/20200624141526.png)

Resource 可以为以下类型

![](https://gitee.com/flywith24/Album/raw/master/img/20200624141844.png)



而 Theme 则不同，它的 key 是 「主题属性」，很显然下图中的 colorPrimary 不是任何 View 中的属性

![主题的定义](https://gitee.com/flywith24/Album/raw/master/img/20200624142007.png)

主题属性有点像把配置抽象为语义化的命名的变量，并把它们塞到 map 中，以便未来使用，主题属性 与 View 属性很像，它们在 attr 中定义的方式以及对应的类型都是类似的，但二者仍有差异

![主题属性的定义](https://gitee.com/flywith24/Album/raw/master/img/20200624142058.png)



在引用主题属性时，可以使用 `?.attr` 语法，其中 ？代表在当前主题中搜索

![](https://gitee.com/flywith24/Album/raw/master/img/20200624143028.png)



### 主题属性带来的优势

如果我们 app 需要支持普通版本和 Pro 版本，它们的主色不同，我们只需定义两个主题，配置不同的 colorPrimary。接着我们需要适配深色主题，那么只需提供不同的数值即可

![](https://gitee.com/flywith24/Album/raw/master/img/20200624143616.png)

![values文件夹下主题配置](https://gitee.com/flywith24/Album/raw/master/img/20200624144329.png)

![values-night文件夹下主题配置](https://gitee.com/flywith24/Album/raw/master/img/20200624144450.png)



这就好比我们有一个 Theme 抽象类，而其中有一个抽象属性 colorPrimary，它有四个实现类，分别重写了 colorPrimary 属性，这样我们便得到了 四个变体，未来想加入新的变体，只需继承该抽象类并重写属性即可。看过 [第一篇译文](https://juejin.im/post/5eead9196fb9a058734e3b03#heading-6) 的小伙伴知道，主题的作用范围是 「树」中的所有子节点。这样我们便很轻松地实现了更改程序主色的功能



如果要使用 Style 实现这一功能，首先，我们需要定义四种 Style。由于 Style 的作用范围是特定 View，因此我们要为每个 View 均定义四套 Style



![两种方案对比](https://gitee.com/flywith24/Album/raw/master/img/20200608153101.png)



### 小结

简单来说，Style 与 Theme 的作用范围不同



Style 只会作用于单一的 View 中，使用时用 `style` 标签

![Style 作用范围](https://gitee.com/flywith24/Album/raw/master/img/20200624150457.png)



Theme 会作用于「树」中的所有子节点，使用时用 `theme` 标签

![theme 作用范围](https://gitee.com/flywith24/Album/raw/master/img/20200624150704.png)



在任意时刻，程序都是运行在某一特定主题下的，例如 activity 被设置了特定主题

![](https://gitee.com/flywith24/Album/raw/master/img/20200624150921.png)



我们在使用时应该注意 theme 与 style 各自的优势，灵活运用二者



## Theme Overlay

这部分内容在视频的 08.55 处



主题是有继承关系的，当该继承关系链中有多个主题配置了同一属性，那么最继承链最底部的内容会生效，在下图中，如果多个主题都声明了 colorPrimary，那么 Theme.Owl.Pink 中的内容会生效（这有点像 Java 的继承关系）

![](https://gitee.com/flywith24/Album/raw/master/img/20200624151431.png)



利用这种继承关系我们可以实现在粉色主题下将部分界面使用蓝色主题



![粉色主题下，某部分需要一个蓝色主题](https://user-gold-cdn.xitu.io/2020/6/18/172c74f73785df66?w=360&h=540&f=gif&s=2076033)



![](https://gitee.com/flywith24/Album/raw/master/img/20200624151353.png)



我们看一下两种主题的继承关系，这两种主题的父级应该是比较相似的，这看起来比较浪费，因为很多属性是相同的

![](https://gitee.com/flywith24/Album/raw/master/img/20200624152345.png)

另外，当你把一个主题设置在另一个主题之上，你需要注意不能将自己想要保留的东西被覆盖掉



`Theme Overlay` 可以很好地解决这一问题，它并不是一项新技术，而是属于一种技巧

![Theme Overlay](https://gitee.com/flywith24/Album/raw/master/img/20200624153011.png)

接下来我们只需关注想要更改的东西，应用下图的声明的主题，则会只改变 colorPrimary 和 colorSecondary 两项属性的值，而其它的所有属性均不变

![](https://gitee.com/flywith24/Album/raw/master/img/20200624153150.png)

### 技巧1：反转颜色

MaterialComponents 提供了 暗色的 `Theme Overlay`

![](https://gitee.com/flywith24/Album/raw/master/img/20200624153900.png)

使用该 `Theme Overlay` 可以将浅色主题中的某部分做成暗色主题



### 技巧2：使用正确的 Context

我们知道主题与 Context 相关，由于上文我们提到的主题的继承关系，使用正确的 Context 很重要

![](https://gitee.com/flywith24/Album/raw/master/img/20200624154828.png)

记住：使用距离最近 View 所在的 Context

![](https://gitee.com/flywith24/Album/raw/master/img/20200624155819.png)



当然更好的做法是使用主题属性

![](https://gitee.com/flywith24/Album/raw/master/img/20200624155924.png)



### 技巧3：在代码中使用 Theme Overlay

如果想要在代码中使用 `Theme Overlay` ，可以将其包裹为 ContextThemeWrapper，这也是 `android:theme` 标签内部做的事情

![](https://gitee.com/flywith24/Album/raw/master/img/20200624160158.png)



## 使用 Theme 和 Style

### Color

这部分内容在视频的 12.36 处



程序内最重要的资源便是 Color，Android 定义 Color 有很多种方式，比如 Color tag，它主要由 ARGB 色值组成

![](https://gitee.com/flywith24/Album/raw/master/img/20200624161608.png)



Color tag 也可以引用其他的 Color tag。需要注意的是，你不能在 Color tag 引用主题属性

![](https://gitee.com/flywith24/Album/raw/master/img/20200624162722.png)



以上是静态的颜色，下面我们谈一谈有状态的颜色

Color state list 允许你在不同状态下定义不同的颜色，它们也可以用作 drawable，例如下面的例子，在 button 按下或禁用时都有相应的颜色

![](https://user-gold-cdn.xitu.io/2020/6/24/172e5751055aa09e?w=902&h=583&f=gif&s=1641505)



下面我们来看一下 Color state list 是如何定义的，我们看到这里使用了 `selector` 标签，本示例中有两个 item，第一项定义了一种颜色，并指定它是在选中状态下才被使用；第二项 没有定义任何状态，这意味着它是默认颜色。如果没有其他颜色可以匹配当前状态时，它就会被使用



![](https://gitee.com/flywith24/Album/raw/master/img/20200624163629.png)



> 这里有一个小技巧，可以将 Color state list 按照最容易出现——最不容易出现的顺序进行排序，这由其背后的实现原理决定，系统会遍历每个 item 直至找出匹配项



在 API 21 官方引入了 `android:alpha` tag 来设置透明度，并在 API 23 中可以引用主题属性。如果你使用 AppCompat 的话，它可以向下兼容到 API 14

![](https://gitee.com/flywith24/Album/raw/master/img/20200624164258.png)

#### 技巧1：设置透明度

我们在开发中可能会遇到这种情况：对于同一个颜色，我们需要不同的透明度。因此我们可能会复制不同透明度的色值

![](https://gitee.com/flywith24/Album/raw/master/img/20200624164833.png)



你可能不想在更改一个颜色后然后再逐一更改其相应透明度的色值，此处我们可以使用 ColorStateList，我们可以使用默认颜色的功能，只配置一个 item，并在此处配置透明度（从 0 到 1）

![](https://gitee.com/flywith24/Album/raw/master/img/20200624165212.png)



#### 技巧2：ColorStateList 与 Drawable 的转换

我们在 View 配置 background 等属性时可以直接传入 color，在内部系统会填充该颜色并将其包装为 ColorDrawable

![](https://gitee.com/flywith24/Album/raw/master/img/20200624165435.png)



但如果你将 ColorStateList 传入是不行的，在 API 28 及之前的设备会崩溃

![](https://gitee.com/flywith24/Album/raw/master/img/20200624165651.png)

这是因为 ColorDrawable 是无状态的，在 Android 10 中，官方加入了 ColorStateListDrawable 解决了这一问题



为了在所有 API 中获得相同的体验，我们可以使用一种变通的做法，使用 `backgroundTint`

![](https://gitee.com/flywith24/Album/raw/master/img/20200624170218.png)

此处使用纯色设置了一个矩形，接着使用 `backgroundTint` 指向了 ColorStateList



### 常用技巧

这部分内容在视频的 17.35 处

#### 技巧1：正确命名资源

我们的项目中肯定有这样命名的资源，它们是按照主题属性命名的。Android Studio 新建项目默认的资源就是这样命名的

![](https://gitee.com/flywith24/Album/raw/master/img/20200624171109.png)



而你的主题大概是这样，主题属性指向同名的颜色资源

![](https://gitee.com/flywith24/Album/raw/master/img/20200624171444.png)



这样做是不推荐的

![](https://gitee.com/flywith24/Album/raw/master/img/20200624171617.png)

我们需要的不是一个语义的命名，而是需要一个文字的命名，我们可以用品牌颜色命名，也可以像 Material Color System，根据色调命名

![根据品牌命名](https://gitee.com/flywith24/Album/raw/master/img/20200624171933.png)

![根据色调命名](https://gitee.com/flywith24/Album/raw/master/img/20200624172108.png)



#### 技巧2：使用统一的 style 名称

大家可能见过这样的样式，一个叫 AppTheme，一个叫 Toolbar，从命名便可以看出它们的用途

![](https://gitee.com/flywith24/Album/raw/master/img/20200624172304.png)



但是如果我们加入了第三种样式，它的用途不是很明显，我们无法区分它是一个主题还是样式

![](https://gitee.com/flywith24/Album/raw/master/img/20200624172431.png)



为此我们可以约定一个命名规则

第一部分为 Style type：主题，样式，文本外观，Theme Overlay，形状外观等等

![第一部分](https://gitee.com/flywith24/Album/raw/master/img/20200624172622.png)



第二部分为 Group name：通常采用应用名称，如果是多 module 也可以为 module 名

![第二部分](https://gitee.com/flywith24/Album/raw/master/img/20200624172735.png)



第三部分为 Sub-group name：这名字通常用于 Widget，也就是使用样式的 View 的名称

![第三部分](https://gitee.com/flywith24/Album/raw/master/img/20200624172903.png)



第四部分为 Variant name：这是可选的，它是主题的变量

![第四部分](https://gitee.com/flywith24/Album/raw/master/img/20200624182341.png)



回到最初的例子，按照我们约定的命名规范改造就变成了这样

![按照命名规范命名](https://gitee.com/flywith24/Album/raw/master/img/20200624183125.png)



这里有一个值得注意的地方，在 Android System 中，`.` 是一个十分神奇的符号，这里有一个基于它的隐含的继承系统

上图的 Widget.MyApp.Toolbar.Blue 实际上继承了中间的这个主题

![](https://gitee.com/flywith24/Album/raw/master/img/20200624183530.png)



这种命名规范可以在 code review 时直观地判断出 style 或 theme 是否用错。如下图，很明显这里使用了 style 标签，却传入了一个主题

![](https://gitee.com/flywith24/Album/raw/master/img/20200624183708.png)

你甚至可以使用 Lint 来解决此问题，[详情移步](https://proandroiddev.com/making-android-lint-theme-aware-6285737b13bc)



#### 技巧3：拆分多个文件



##### 简单模式

将资源类型文件进行标准的分类

- `theme.xml` ：Theme 和 Theme Overlay
- `type.xml`：字体，文本外观，文本尺寸，字体文件等
- `style.xml`：只有 Widget style
- `dimens.xml` `colors.xml` `strings.xml`：其它类型归类于实际的资源类型

![简单模式](https://gitee.com/flywith24/Album/raw/master/img/20200624185037.png)



##### 复杂模式

复杂模式是按照逻辑进行分类，例如形状相关的放入 `shape.xml`，如果想要实现全屏的UI，可以在 `sys_ui.xml` 中控制状态栏/导航栏颜色，以及是否显示等等



![复杂模式](https://gitee.com/flywith24/Album/raw/master/img/20200624185154.png)



在 Android Studio 的 Android 视图下，这样做的效果是很好的。如下图，可以很清晰的看到 light 主题和 dark 主题的主题文件



![](https://gitee.com/flywith24/Album/raw/master/img/20200624185832.png)





## Material

### 颜色系统

这部分内容在视频的 24:00 处

该系统构建基础是大量使用语义命名的变量，这些变量都属于「主题属性」。它的运作原理是 library 展示与这些使用语义命名的颜色相关的主题属性，而开发者负责为这些颜色提供数值。在 library 内，用这些颜色构建所有的 Widget



对于颜色系统，开发者需要了解一些常用的颜色

![](https://gitee.com/flywith24/Album/raw/master/img/20200624190951.png)

`colorPrimary` 和 `colorSecondary` 是 app 品牌的主要颜色，Variant 为主色的对比色；`colorSurface` 十分有用，它负责在某些控件表面的颜色；`colorError` 是错误的警示色，因此你没必要在使用时硬编码这些颜色



颜色系统还会提供一些 `On` 命名的颜色，这种颜色会保证拥有和类似名称颜色形成对比的颜色，例如 `colorOnPrimary` 永远会和 `colorPrimary` 形成对比

![](https://gitee.com/flywith24/Album/raw/master/img/20200624191510.png)



你可以在自己的主题中配置这些颜色，注意这里不必配置所有的颜色，如果你继承了一些 Material 主题，它们会提供所有颜色的默认色，比如下图中没有设置 `colorSurface`，则会使用 Material Light 主题内定义的 `colorSurface`

![](https://gitee.com/flywith24/Album/raw/master/img/20200624191754.png)



之后你便可以在 layout 或 style 中使用这些颜色了

![](https://gitee.com/flywith24/Album/raw/master/img/20200624192323.png)

#### 技巧

一个有用的技巧是可以将这些颜色与 ColorStateList 结合

比如我们想做一个分割线，不必创建名为 colorDivider 的新颜色，直接从 colorOnSurface 中取一个颜色即可，这个颜色肯定会和背景色形成对比

![](https://gitee.com/flywith24/Album/raw/master/img/20200624192538.png)



而且它会响应不同的主题，在浅色主题下，20% `colorOnSurface` 是一种黑中带白的颜色。在暗色主题下，`colorOnSurface` 会变成白色，此时的 20% `colorOnSurface` 会提供合适的对比



![](https://gitee.com/flywith24/Album/raw/master/img/20200624192904.png)



以语义命名的颜色是十分有用的，你可以省去大量的颜色定义



### 排版系统

这部分内容在视频的 28:06 处

在设计中，通常使用固定的几种字号进行排版，例如大标题1，大标题2，文本主体，副标题，按钮等等

![](https://gitee.com/flywith24/Album/raw/master/img/20200624194528.png)

而这些都是作为主题属性实现的

![](https://gitee.com/flywith24/Album/raw/master/img/20200624194646.png)

然后便可以在应用中引用这些主题属性

![](https://gitee.com/flywith24/Album/raw/master/img/20200624194743.png)

Material 的 Text 十分强大，它可以设置行高，如果你遇到了设置行高却不生效的问题，使用 Material 组件可以解决这一问题

### 形状系统

这部分内容在视频的 30:07 处

Material 采用了 [shape system](https://material.io/design/shape)，该系统为小型，中型和大型组件 [提供](https://material.io/develop/android/theming/shape/) 了主题属性。请注意，如果要在自定义组件上设置 shape，则可能要使用 `MaterialShapeDrawable` 作为其背景，它可以理解并实现 shape

这里是通过 ShapeAppearance 来定义的，ShapeAppearance 和 TextAppearance 类似，是一种针对形状系统的配置

![](https://gitee.com/flywith24/Album/raw/master/img/20200624195222.png)

它由几个组件组成

首先是 cornerFamily，支持圆角和切角，圆角的方向等等

![](https://gitee.com/flywith24/Album/raw/master/img/20200624195531.png)



ShapeAppearance  还支持 overlay，可以更改特定的 Widget

![](https://gitee.com/flywith24/Album/raw/master/img/20200624195830.png)



在使用 overlay 时要注意的是很多 Material 组件是有着自己的 ShapeAppearance overlay，例如 BottomSheet，它会取消底部的圆角

![](https://gitee.com/flywith24/Album/raw/master/img/20200627083254.png)

#### 技巧

形状系统是由 MaterialShapeDrawable 实现的

![](https://gitee.com/flywith24/Album/raw/master/img/20200627083556.png)



而 MaterialShapeDrawable 有一个强大的功能就是它有着一个属性叫 interpolation 

![](https://gitee.com/flywith24/Album/raw/master/img/20200627083945.png)



使用它可以为形状系统做动画，如果它的值为0，那么所配置形状不会生效，如果值为1，那么形状系统会完整地应用到 drawable 上



![](https://user-gold-cdn.xitu.io/2020/6/27/172f33edf584e8e5?w=540&h=648&f=gif&s=2815174)



## 暗黑主题

这部分内容在视频的 34:41 处



暗黑主题的适配很简单，可以通过代码设置当前主题和获取当前主题

![](https://gitee.com/flywith24/Album/raw/master/img/20200627085246.png)



可以使用 Material 组件的 DayNight，这样在打开/关闭 暗黑主题时相应的主题属性的色值都会跟随变化

![](https://gitee.com/flywith24/Album/raw/master/img/20200627085427.png)



#### 技巧1 ：抽取主题



很多时候只做上面的两步并不能很好地适配暗黑主题，例如我们的应用在浅色主题下是这样的，深色的内容在浅色的背景上



![](https://gitee.com/flywith24/Album/raw/master/img/20200627105725.png)



而使用了夜间主题，可能会变成这样



![](https://gitee.com/flywith24/Album/raw/master/img/20200627105830.png)



而我们想要的效果是这样的



![](https://gitee.com/flywith24/Album/raw/master/img/20200627105903.png)



这是由于设置颜色时硬编码导致的

![](https://gitee.com/flywith24/Album/raw/master/img/20200627105942.png)



实际上，这种情况使用主题属性会有更好的效果

![](https://gitee.com/flywith24/Album/raw/master/img/20200627110011.png)



如果想要 `colorPrimary` 在不同的主题下使用不同的颜色，我们应该如何设置？

![](https://gitee.com/flywith24/Album/raw/master/img/20200627110254.png)



或许你会在 `values-night/colors.xml` 为暗色主题定义色值，**但不建议这样做！**

![不建议这样！](https://gitee.com/flywith24/Album/raw/master/img/20200627110717.png)



**最好的做法是抽取公共部分到基础主题，然后在此基础上对浅色和深色主题分别配置差异化的属性**

![](https://gitee.com/flywith24/Album/raw/master/img/20200627110812.png)



#### 技巧2：ColorPrimary 的使用

有些时候我们的 colorPrimary 是一种亮色，例如下图中的蓝色，但在暗黑主题下我们想使用相对较暗的颜色，例如 `?attr/colorSurface`，Material 组件内部为我们做好了转换，直接使用 `?attr/colorPrimarySurface` 即可

![](https://user-gold-cdn.xitu.io/2020/6/27/172f3c75e62305d5?w=2512&h=1568&f=png&s=2254379)



## Demo

[demo地址在这里](https://github.com/Flywith24/Flywith24-Theme-demo)，如果感觉对你有帮助的话，点一颗小星星吧～ 😉




## 关于我

我是 [Flywith24](https://flywith24.gitee.io/)，我的博客内容已经分类整理 [在这里](https://github.com/Flywith24/BlogList)，点击右上角的 Watch 可以及时获取我的文章更新哦 😉

- [掘金](https://juejin.im/user/57c7f6870a2b58006b1cfd6c)

- [简书](https://www.jianshu.com/u/3d5ad6043d66)

- [Github](https://github.com/Flywith24)

  

![](https://gitee.com/flywith24/Album/raw/master/img/20200508153754.jpg)  
