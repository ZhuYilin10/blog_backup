---
title: 记录 Android 7.0 适配踩过的一个坑
date: 2018-02-25 16:51:08
tags: Android
---
还是上次适配 Android O 的 Adaptive Icons 时候留下的一个大坑。。。。。。
测试的同事确认没什么问题可以发布新版本了，开开心心上线后发现我们自己的手机一更新就会崩溃，而测试的同事却没有问题，赶紧撤下上线了的版本，经检查发现原来同事的手机还运行着 Android 4.4，而我们的手机已经是 Android 8.0。
在我们适配 Android 8.0 的时候将 compileSdkVersion 和 targetSdkVersion 的版本都升级到了 26，由此带来的问题就是没有注意到 Android 7.0 以上的新特性，具体 Android N 的全新特性可以查看[Android 7.0 行为变更](https://developer.android.com/about/versions/nougat/android-7.0-changes.html)。
在 Android 7.0 以前，在下载新版本 apk 后，使用 Intent 安装 apk 文件的代码大致如下：
```Java
val intent = Intent(Intent.ACTION_VIEW)
intent.flags = Intent.FLAG_ACTIVITY_NEW_TASK
val file = File(Environment.getExternalStorageDirectory() + "/download/" + "app.apk")
val uri = Uri.fromFile(file)
intent.setDataAndType(uri, "application/vnd.android.package-archive")
```
但在 Android N 上，为了更好的控制权限和注重安全隐私，Google 进行了全新的规定：
```
对于面向 Android 7.0 的应用，Android 框架执行的 StrictMode API 政策禁止在您的应用外部公开 file:// URI。如果一项包含文件 URI 的 intent 离开您的应用，则应用出现故障，并出现 FileUriExposedException 异常。
要在应用间共享文件，您应发送一项 content:// URI，并授予 URI 临时访问权限。进行此授权的最简单方式是使用 FileProvider 类。如需了解有关权限和共享文件的详细信息，请参阅共享文件。
```
当然本身这项适配工作并没有难度，这里也只是记录一下适配的方法。
首先是第一步，在清单文件中进行申明：
```
<manifest>
    ...
    <application>
        ...
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="com.xxxx.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            ...
        </provider>
        ...
    </application>
</manifest>
```
接下来我们要创建resource xml file:
```
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <files-path name="my_docs" path="docs/"/>
</paths>
```
这里需要解释一下，在paths节点内部支持以下几个子节点：
**<root-path/> 代表设备的根目录new File("/");**
**<files-path/> 代表context.getFilesDir()**
**<cache-path/> 代表context.getCacheDir()**
**<external-path/> 代表Environment.getExternalStorageDirectory()**
**<external-files-path>代表context.getExternalFilesDirs()**
**<external-cache-path>代表getExternalCacheDirs()**
我们可以根据位置的不同选择对应的节点。
当然创建了文件之后我们需要在清单文件中注明。
```
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="com.xxxx.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
    android:name="android.support.FILE_PROVIDER_PATHS"
    android:resource="@xml/provider_paths" />
</provider>
```
这个时候基本的配置已经完成了，我们还需要在代码中做一个版本的判断。回到我们 install apk 的方法。
Uri 不能单纯的通过 Uri.fromFile() 函数来完成，
```
val uri = if (SDK_INT >= N) {
  intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)
  FileProvider.getUriForFile(this, "com.xxxx.fileprovider", file)
  } else {
    Uri.fromFile(file)
  }
```
完成之后提交合并打包，然后再一次尝试应用内升级，一步成功～
不得不说，以后需要多阅读 Android 的官方文档，尤其是涉及新版本适配的时候，很多坑完全是可以避免的。
