---
title: Material Design 学习笔记 2
date: 2016-12-15 15:41:14
tags: Android
---
好久没更新Blog了，当初说想保持常更新来记录自己在开发道路上的事情，结果也是没做到，作为一个有女朋友爱看电视爱写代码的开发人员来说，每天的时间都花在了陪女朋友、看电视剧和写代码上，好吧我知道开发人员怎么能有女朋友呢。。。但我就是有，而且在我眼中是非常可爱漂亮我很爱她（捂脸）～～～
好了以上依旧是废话。==
前几天在北京举办的的Google开发者大会（Google Develop Day），官方宣布了Google Developers中国网站发布，怀着激动的心情关掉了代理，登录了developer.android.google.cn。快的令人感动，404世界什么时候回来还不知道，但是，至少我们看到了希望。
继续说说Material Design。
首先要说的就是Toolbar。这是一个标准的Material Design组件，在Android应用中，被大量使用。在Android 3.0时Android推出了ActionBar，ActionBar 过去最多人使用的两大套件就是 ActionBarSherlock 以及官方提供在 support library v7 里的 AppCompat。而Toolbar的推出，很大程度就是用来取代ActionBar的。

当我们的Activity继承自AppCompat的时候，会默认看到ActionBar，为了使用全新的Toolbar或者其他布局我们可以选择在Style中，对Theme进行设置，将AppTheme 的parent设置为Theme.AppCompat.Light.NoActionBar。这样我们可以看到一个没有ActionBar的空白Activity。然后我们就可以在XML文件中放进Toolbar组件。
当然我们会注意到一个问题，就是toolbar的阴影。我们知道在Material Design中由于材料的层次高度会像自然界中一样存在着阴影，具体可以参考知乎和Bilibili等非常符合设计规范的APP。但是如何给toolbar增加阴影，Google后发现一般给出的答案是在toolbar下加一层阴影图片，一个view。。。。但后来我发现了一个组件叫做AppbarLayout。
AppBarLayout 是继承LinerLayout实现的一个ViewGroup容器组件，它是为了Material Design设计的App Bar，支持手势滑动操作。当我们将AppbarLayout和Toolbar配合使用的时候，就会得到我们想要的阴影效果。
```XML
<android.support.design.widget.AppBarLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">
        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="?attr/colorPrimary"
            android:minHeight="56dp"
            app:titleTextColor="@color/white">
        </android.support.v7.widget.Toolbar>
</android.support.design.widget.AppBarLayout>
```
而这个时候我们发现，我们无法设置Toolbar的文字颜色。我们可以使用设置主题的方式来设置。
```XML
<style name="CC.ToolbarWhiteTheme" parent="Widget.Design.CollapsingToolbar">
        <item name="android:textColorSecondary">@color/white</item>
</style>
```
我们可以自定义Toolbar的布局。以知乎中的一个界面为例。
知乎的一个界面
我们可以看到知乎的Toolbar从左至右依次是title、share 的 icon和三个小圆点。
话说那些用惯iOS程序的人看到有些Android特色的设计几乎都是一脸懵逼。。。。。包括公司的iOS开发，我在投屏演示功能的时候他们几乎都不能理解toolbar里的三个小圆点、FloatingActionBar和下拉刷新时候不动的界面但是出现的小圈。。。。他们觉得好蠢但我是发自内心的觉得比iOS的风格好看好吗。。。。。果粉勿喷。
title的使用很简单，因为toolbar本身就带那么一个属性，我们可以直接设置toolbar.setTitle = “……”来实现。分享按钮同样简单，即在布局中加入这个icon并添加点击事件，一般有baseactivity的可以在里面抽一个函数出来方便之后使用。
关于三个点，我们可以在res中建立一个package叫做menu，与layout、style等平级，然后实现父类的onCreateOptionsMenu函数，并将menu映射进去。
代码如下：
```Java
override fun onCreateOptionsMenu(menu: Menu): Boolean {
  menuInflater.inflate(R.menu.aaaaaa, menu)
  return super.onCreateOptionsMenu(menu)
}
```
顺便说一句，我用的是Kotlin，我们公司可能是比较早将Android项目全部使用Kotlin的公司了，Kotlin的本质与Java类似，但是语法上很多高级函数和他的扩展性让人欲罢不能，有坑，等有空可以总结一下。
今天大概写到这里，我想去陪女朋友了，这几天有事请假没有上班，刚好可以陪自己的女朋友，我一个人在上海工作，日常不在她身边，真的辛苦她了。代码重要，女朋友更重要。
