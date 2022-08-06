
## 主要功能

- 高效的App版本更新库，提供两个下载引擎：HttpURLConnection(推荐) 和系统的 DownloadManager
- 支持断点续传，节省流量（避免网络抖动时重新下载）
- 支持设置是否使用本地下载好的缓存文件
- 通知栏显式下载进度，且支持根据下载状态设置通知策略
- 统一处理应用没有安装权限的交互逻辑
- 内置通用的更新弹窗、支持强制更新（支持自定义更新弹窗）

<img src="/images/img.png" width="540px">

## 开发过程

参考过 github 上其他的一些更新库，主要是参考了系统的 DownloadManager 源码。

- 系统的 DownloadManager 更新通知栏进度之前，会进行速度采样，不能 Buffer 满了就通知，避免短时间创建大量通知对象
- 重定向需要考虑 301,302,303,307 308，其中 301,302,303 在 HttpURLConnection 定义了常量，307，308 没有定义，需要自己定义常量
- 重定向需要考虑最大次数，避免极端情况的死循环
- 需要考虑下载续传的 Response code
- 由于各厂商 Android 定制化，最好能够统一处理应用没有安装权限的交互逻辑


## 经过测试的机型

| 厂商        | 机型    |  系统版本  |
| --------   | -----:   | :----: |
| 小米        | Redmi4     |   Android 6.0.1    |
| 华为        | 荣耀7i      |   Android 6.0.1  |
| 华为        | 荣耀V8      |   Android 7.0  |
| 小米        | Note4X      |   Android 7.0  |
| vivo        | Y85A      |   Android 8.1.0  |
| 华为        | 荣耀V10      |   Android 9  |
| 华为        | Mate20      |   Android 9  |
| vivo        | x50      |   Android 10  |
| 荣耀        | Magic3 至臻版      |   Android 11  |
| 小米        | 小米 11 Ultra      |   Android 12  |

## 如何使用

### 开始下载
```
val request = DownloadRequest(applicationContext, url, mode)
    .setNotificationTitle(resources.getString(R.string.app_name))
    .setNotificationContent(getString(R.string.downloader_notifier_description))
    .setIgnoreLocal(ignoreLocalFile)
    .setNeedInstall(needInstall)
    .setNotificationVisibility(notifierVisibility)
    .setNotificationSmallIcon(R.mipmap.ic_launcher)
    .setShowNotificationDisableTip(notifierDisableTip)
    .registerListener(this)
    .startDownload()
```

### 监听的回调

```
override fun onDownloadStart() {
}

override fun onProgressUpdate(percent: Int) {
}

override fun onDownloadComplete(uri: Uri) {
}

override fun onDownloadFailed(e: Throwable) {
}
```

### 移除监听

```
override fun onDestroy() {
    super.onDestroy()
    request?.unregisterListener(this)
}
```

> 注意：下载完成、下载失败都会自动移除监听器


## TODOs

- [x] 判断 DownloadManager 是否可用，如果可用，优先使用 DownloadManager，不可用，则使用常规的下载方式
- [x] 下载成功后，启动安装界面前，处理安装未知应用的权限。
- [x] 判断文件是否已经下载，不仅要判断下载状态，还需要判断文件是否存在
- [x] 断点续传
- [x] 通知栏进度条
- [x] 国际化
- [x] 进度信息，要进行速度采样，避免短时间创建大量通知对象
- [x] 如果禁用通知栏，可以设置是否提示
- [x] 添加 FileProvider 其他 path
- [x] 检查回调内存产生的内存泄露
- [x] 相同 url 重复触发下载操作
- [x] HTTPS 证书，不要信任所有证书，根据系统信任的证书即可
- [x] 强制更新
- [x] 通知栏的点击处理
- [ ] 支持 ETAG
- [ ] 支持 MD5 校验
- [ ] 支持多文件同时下载，打造成文件下载器
- [ ] 多文件下载时，通知栏能够聚合
- [ ] 网络重新连接，能够自动继续下载
- [ ] 根据网络类型指定下载策略



