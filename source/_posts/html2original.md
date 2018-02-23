---
title: 使用原生取代 WebView —— 图文混排 1
date: 2018-02-22 19:30:57
tags: Android
---
这篇总结博客想写很久了，一直没空动笔，很羞愧。今天终于有空坐下来，2018年的第一篇技术总结，写写2017年下半年花了很多心思做的使用原生取代 WebView，也就是原生实现图文混排及交互的功能。
我们公司是做英语作业平台的，所以我们的核心功能就是试卷，学生们通过我们的软件完成填空题，选择题，文本题等各种题型。在以往我们的试卷部分是通过 WebView 由前端开发的同事写好交互，我们通过调用 JS 代码来实现功能，但我一直对于这样的混合开发保持怀疑，毕竟对于手机，尤其是碎片化严重的 Android 来说，不同的系统版本不同的厂家甚至不同的用户习惯都会带来 WebView 性能差甚至加载出现问题的情况，所以我们的项目实现原生图文混排及交互的功能势在必行。
原生图文混排及交互，也就是富文本。iOS 的同事在拿到这个任务的时候选择了 iOS 著名的富文本库 [YYText](https://github.com/ibireme/YYText)，而负责 Android 端的我选择了使用原生 TextView 来实现，使用 SpannableString 以及 SpannableStringBuilder 来构造。
SpannableString 与 SpannableStringBuilder 是非常强大的，我们对于 TextView 所想实现的所有功能和样式几乎都可以通过它来完成。最简单的例子就是多种颜色的一段文字。
如图所示。
![多种颜色文字](https://ws1.sinaimg.cn/large/af31670dgy1fople7zmr8j20ma14u75o.jpg)
如果是针对整个 TextView 来设置颜色是无法达到这样的效果的，我们需要用 SpannableString 设置不同的 span 后拼接，这样可以达到这样的效果。
```java
        val textView = findViewById<TextView>(R.id.sample_text)
        val spannableStringBuilder = SpannableStringBuilder()

        val blueString = SpannableString("Hello")
        blueString.setSpan(ForegroundColorSpan(ContextCompat.getColor(this, R.color.colorPrimary)), 0, blueString.length, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE)
        spannableStringBuilder.append(blueString)

        val redString = SpannableString("Hello")
        redString.setSpan(ForegroundColorSpan(ContextCompat.getColor(this, R.color.colorAccent)), 0, redString.length, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE)
        spannableStringBuilder.append(redString)

        textView.text = spannableStringBuilder
```
这里需要提及的是 setSpan 这个方法。
```Java
        setSpan(Object what, int start, int end, int flags)
```
这里备注一下几个参数对应的含义。
第一个参数 Object what 指的是我们需要设置的 Span 的类型，如我们显示字体颜色的 ForegroundColorSpan；
第二个参数 int start 以及第三个参数 int end 指的是我们对于这段 spannableString 设置样式的起始位置；
而第三个参数也就是 flag 的取值如下
```
Spannable. SPAN_INCLUSIVE_EXCLUSIVE：前面包括，后面不包括，即在文本前插入新的文本会应用该样式，而在文本后插入新文本不会应用该样式
Spannable. SPAN_INCLUSIVE_INCLUSIVE：前面包括，后面包括，即在文本前插入新的文本会应用该样式，而在文本后插入新文本也会应用该样式
Spannable. SPAN_EXCLUSIVE_EXCLUSIVE：前面不包括，后面不包括
Spannable. SPAN_EXCLUSIVE_INCLUSIVE：前面不包括，后面包括
```
这样就可以基本实现通过设置 span 来达到不同的效果。
这是使用 span 的总结第一篇，为自己后面列个提纲
1. ImageSpan 包括 本地图片用法 网络图片用法 Data Base64图片加载方法
2. clickableSpan
3. 特殊化的自定义 Span
4. 如何在 TextView 指定位置插入 View
