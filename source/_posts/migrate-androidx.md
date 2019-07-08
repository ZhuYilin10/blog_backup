---
title: 项目迁移至 AndroidX
date: 2019-07-08 15:19:57
tags: Android AndroidX
---
根据 Android Developer 的文档，我们可以看到：
```
AndroidX 是 Android 团队用于在 Jetpack 中开发、测试、打包和发布库以及对其进行版本控制的开源项目。
AndroidX 对原始 Android 支持库进行了重大改进。与支持库一样，AndroidX 与 Android 操作系统分开提供，并与各个 Android 版本向后兼容。AndroidX 完全取代了支持库，不仅提供同等的功能，而且提供了新的库。
```
也就是说，由于在后续版本中，会逐步放弃对 support lib 的升级和维护，所以我们必须迁移到 AndroidX。
对于一个项目，迁移到 AndroidX 的步骤非常简单。
我们选择 Refactor -> Migrate to AndroidX
![refactor](https://s2.ax1x.com/2019/07/08/Zr0gaD.png)
![refactor](https://s2.ax1x.com/2019/07/08/ZrBeQ1.png)
依次点击 Refactor 我们可以发现经过长时间的 index 后，项目将成功地迁移至 AndroidX，此时我们可以看到项目中原有的包名为 support 的库将依次迁移至 Androidx 对应的包名。
另外我们可以观察发现，在项目的 gradle.properties 文件中多了两行：
```
android.useAndroidX=true
android.enableJetifier=true
```
在官方文档中我们可以看到
```
android.useAndroidX：如果设置为 true，Android 插件会使用相应的 AndroidX 库，而非支持库。如果未指定，则该标记默认为 false。
android.enableJetifier：如果设置为 true，Android 插件会重写其二进制文件，自动迁移现有的第三方库以使用 AndroidX。如果未指定，则该标记默认为 false。
```
不过再给公司的项目迁移至 AndroidX 的时候还是发现了很多意料之外的问题。
第一个也是最主要的就是大量第三方库的报错与失效。
很多的库提供了专门的适配了 AndroidX 的版本，比如运用于 RecycleView 的著名框架 CymChad:BaseRecyclerViewAdapterHelper，非常优秀的框架，并且良心的提供了 AndroidX 对应版本。
不过也有比较麻烦的事情，比如我们在使用 kotlin 时使用的 kotterKnife。
在我迁移项目之前，kotterKnife 还并没有支持 AndroidX（不过不知道现在是否支持)，于是我就动了把整个 ButterKnife.kt 拷进项目加以修改的年头。如下：
```
public fun <V : View> AppCompatActivity.bindView(id: Int)
        : ReadOnlyProperty<AppCompatActivity, V> = required(id, viewFinder)
public fun <V : View> Fragment.bindView(id: Int)
        : ReadOnlyProperty<Fragment, V> = required(id, viewFinder)
public fun <V : View> Fragment.bindOptionalView(id: Int)
        : ReadOnlyProperty<Fragment, V?> = optional(id, viewFinder)
public fun <V : View> AppCompatActivity.bindOptionalView(id: Int)
        : ReadOnlyProperty<AppCompatActivity, V?> = optional(id, viewFinder)

private val AppCompatActivity.viewFinder: Finder<AppCompatActivity>
        get() = { findViewById(it) }
private val Fragment.viewFinder: Finder<Fragment>
        get() = { view!!.findViewById(it) }
```
就这样适配了 AndroidX 中 Fragment 和 Activity。
另外的问题就是一些 UI 方面的失效，不过修起来还是比较容易的，毕竟都是可以通过代码来控制的。
