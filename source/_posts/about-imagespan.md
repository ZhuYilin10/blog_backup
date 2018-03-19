---
title: 使用原生取代 WebView —— ImageSpan
date: 2018-03-19 19:42:09
tags: Android
---
好累的两周。
经历了上海的房子被二房东收回去，自己说没得住就没得住，经历了辞职，经历了苏州找房，现在还在上海短租着上着班，漂泊不定，整个人会很沮丧，想想真的很难受。这个时候感觉自己应该干点什么事情，比如把之前想总结的 ImageSpan 的用法给写了。
ImageSpan 是图文混排中最重要的一部分，TextView 之所以能做到图文混排也就是因为 ImageSpan 的存在。
首先看一下最简单的本地图片的加载。
```Java
TextView textView = findViewById(R.id.image_span);

SpannableString spannableString = new SpannableString("0000000ddddddddd");
Drawable drawable = ContextCompat.getDrawable(this, R.drawable.ic_launcher_foreground);
drawable.setBounds(0, 0, drawable.getIntrinsicWidth(), drawable.getIntrinsicHeight());
spannableString.setSpan(new ImageSpan(drawable), 4, 5, Spanned.SPAN_INCLUSIVE_EXCLUSIVE);

textView.setText(spannableString);
```
![本地图片加载](https://ws1.sinaimg.cn/large/af31670dgy1fpieygy22nj20ii0y4jso.jpg)
当然我们可以通过拼接的方式，多个 SpannableString 组合成为一个 SpannableBuilder 来加载多张图片，但是值得注意的是， drawable 必须要 setBounds，Drawable的setBounds方法有四个参数，setBounds(int left, int top, int right, int bottom),这个四参数指的是drawable将在被绘制在canvas的哪个矩形区域内。这里的单位是 px 不是 dp。建议自己写一个 DisplayUtils 或者使用 Kotlin 提供的 anko 库等方式来转换。

但是我们使用本地图片的时候终究少，更多时候我们会使用网络图片，如何加载网络图片就成了一个新的问题，我在开发的时候选择的方案是第三方图片加载框架的配合使用。这里要注意的是，加载图片应该是在子线程完成，然后主线程刷新的。
首先我们需要创建一个 BitmapDrawable 的对象
```Java
class URLDrawable : BitmapDrawable() {
    var drawable: Drawable? = null

    override fun draw(canvas: Canvas) {
        if (drawable != null) {
            drawable!!.draw(canvas)
        }
    }
}
```
然后我们就要开始写这个 AsyncTask 了。这里提一下 AsyncTask，它是 Android 提供的一个助手类，它对 Thread 和 Handler 进行了封装，方便我们使用。Android 之所以提供 AsyncTask 这个类，就是为了方便我们在后台线程中执行操作，然后将结果发送给主线程，从而在主线程中进行UI更新等操作。
直接放代码：
```Java
class ImageGetterAsyncTask(private val context: Context, private val urlDrawable: URLDrawable, val url: String) : AsyncTask<TextView, Void, Bitmap?>() {
    private var textView: TextView? = null

    override fun doInBackground(vararg params: TextView): Bitmap? {
        textView = params[0]
        return try {
            Picasso.get().load(url).get()
        } catch (e: Exception) {
            null
        }
    }

    override fun onPostExecute(bitmap: Bitmap?) {
        val bitmapDrawable = BitmapDrawable(context.resources, bitmap)
        bitmapDrawable.setBounds(0, 0, 100, 100)
        urlDrawable.setBounds(0, 0, 100, 100)
        urlDrawable.drawable = bitmapDrawable
        urlDrawable.invalidateSelf()
        textView!!.invalidate()
    }
}
```
这里一定要把 textview 传进来，因为这样我们才能实现刷新 UI 的目的。
我选用的是 Picasso 来加载图片，当然还可以选择其他的框架，只要稍作修改就可以了。
最后是 URLImageParser
```Java
class URLImageParser(private val textView: TextView, private val context: Context) {

    fun getDrawable(url: String): Drawable {
        val urlDrawable = URLDrawable()
        ImageGetterAsyncTask(context, urlDrawable, url).execute(textView)
        return urlDrawable
    }
}
```
至于使用，则很简单，因为我们已经拿到了一个 drawable 的对象了。
```Java
val drawableFromNet = URLImageParser(textView, this).getDrawable("https://www.baidu.com/img/bd_logo1.png")
drawableFromNet.setBounds(0, 0, 100, 100)
spannableString.setSpan(ImageSpan(drawableFromNet), 4, 5, Spanned.SPAN_INCLUSIVE_EXCLUSIVE)
```
![网络图片加载](https://ws1.sinaimg.cn/large/af31670dgy1fpiezr5gfrj20ii0y4gmu.jpg)
最后还有一种是特殊的情况，就是加载 database 64 图片。
```Java
val stream = ByteArrayInputStream(Base64.decode(imageDataBytes.toByteArray(), Base64.DEFAULT))
val bitmap = BitmapFactory.decodeStream(stream)
val bitmapDrawable = BitmapDrawable(env.resources, bitmap)
bitmapDrawable.setBounds(0, 0, widthInt, heightInt)
spannableString.setSpan(TransYImageSpan(bitmapDrawable, env.px2dip(exVerticalAlign.toInt())), 0, spannableString.length, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE)
builder.append(spannableString)
```
代码已上传 Github [ImageSpan](https://github.com/ZhuYilin10/ImageSpan)
