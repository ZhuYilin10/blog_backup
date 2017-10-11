---
title: 解决太坑的 svg 资源向下兼容性问题记录
date: 2017-10-11 23:06:04
tags: Android
---
终于合并了上次做的 Adaptive Icon 的分支，但发现服务器跑不过去，一看是由于没有升级 gradle 等对应配置文件的问题。
当升级过后，发现了一个很大的问题。我们早早使用了 SVG 作为图片资源文件，取代了体积较大的 .png 文件。一直以来的做法是
```XML
<ImageView
 android:layout_width="wrap_content"
 android:layout_height="wrap_content"          
 android:src="@drawable/xxxxxxxx" />
```
但现在发现我们的用法是错误的。经过踩坑简单记录一下正确的方法，可以解决向下兼容性问题。
首先，需要在 build.gradle 配置文件中添加
```
defaultConfig {
        vectorDrawables.useSupportLibrary = true
      }
```
然后引入
```
compile 'com.android.support:appcompat-v7:26.1.0'
```
第三步就是将 ImageView 属性设置为如下：
```XML
<ImageView
 android:layout_width="wrap_content"
 android:layout_height="wrap_content"          
 app:srcCompat="@drawable/xxxxxxxx" />
```
到此为低版本加载 SVG 的方法。当然 Android 5.0 以上不存在这个问题。
另外提一句，这一切是在 Activity 父类为 AppCompatActivity 的情况下进行，如果 Activity 继承自 Activity，那么应该使用 android.support.v7.widget.AppCompatImageView, 因为当使用 AppCompatActivity 时会自动将 ImageView 包装为 AppCompatImageView。
另一个问题就是 Textview 显示 SVG 图片的问题，这是使用比较多的地方，我们在使用 android:drawableRight 属性的时候会倒入 SVG 图像，但是这样做的结果就是低版本的奔溃。事实上，AppCompatTextView 是没有对 CompoundDrawable 进行适配的，所以我们的解决方法是代码手工写，判断系统版本如果小于5.0，就用 ContextCompat.getDrawable 获取到 Drawable 实例，再 setCompoundDrawablesWithIntrinsicBounds。建议自己定义一个 TextView，免去手写烦恼，为了节省时间，我用了一个开源库， [VectorCompatTextView](https://github.com/woxingxiao/VectorCompatTextView)，谢谢作者分享这个库。
当然我说的 ContextCompat.getDrawable 获取到 Drawable 实例然后赋值的方法是最好用的方法，当个别 XML 解决不了的时候, 如 ImageView 想设置 background 但没有那么一个叫做 backgroundCompat 属性的时候建议这样写。
```
imageView.background = ContextCompat.getDrawable(context, R.drawable.xxxxxxxx)
```
当然今天只是简单记录一下，这几天有一些东西需要记录，包括多渠道打包配置文件，还有 Linux 安装体验，包括重装系统后恢复 blog 的记录。
