---
title: 为 WebView 添加超时
date: 2018-12-19 16:55:45
tags: Android RxJava
---
在手机程序开发中，网页成为一个非常重要的部分，在一些复杂的效果显示上，网页有着远超原生的便捷性，所以现在很多页面都会使用 WebView 进行加载，但是在复杂的页面显示时由于资源过大或者别的原因，可能会出现超时。但是在我希望对 WebView 进行一个 timeout 参数设置的时候，发现 WebView 本身没有提供相应的设置。
一开始，我选择了和网上大多开发者一样的方法进行超时的设置，即在 WebViewClient 中的 onPageStarted 时启动一个计时器，在到时间后获取页面加载进度，判断是否已经加载完成，如果没有完成则进行错误的处理。
代码如下：
```Java
lateinit var disposable: Disposable
override fun onPageStarted(view: WebView, url: String?, favicon: Bitmap?) {
                disposable = Observable.timer(10L, TimeUnit.SECONDS).observeOn(AndroidSchedulers.mainThread()).subscribe {
                    if (view.progress < 100) {
                        view.pauseTimers()
                        view.stopLoading()
                        loadSubject!!.onError(Throwable("页面超时"))
                    }
                }
                super.onPageStarted(view, url, favicon)
            }
```
但后来发现 RxJava 自身提供了一个 timeout 的函数进行操作。
```
对原始Observable的一个镜像，如果过了一个指定的时长仍没有发射数据，它会发一个错误通知
如果原始Observable过了指定的一段时长没有发射任何数据，Timeout操作符会以一个onError通知终止这个Observable。
```
这样我们在 webview.load() 的时候对网页添加超时，代码如下：
```Kotlin
fun load(): Observable<String> {
        return webView.load().timeout(10L, SECONDS).onErrorResumeNext { throwable: Throwable ->
            if (throwable is TimeoutException) {
                return@onErrorResumeNext Observable.error(Throwable("页面加载失败，请稍后重试"))
            }
            Observable.error(throwable)
        }
    }
```
很简洁，很好懂。
