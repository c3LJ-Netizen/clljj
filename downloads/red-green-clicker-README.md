# 红绿快点 · Android

检测你选定屏幕位置从红色变成绿色，立即向另一个指定位置发出点击。

**[手机直接下载 APK](https://github.com/c3LJ-Netizen/clljj/raw/refs/heads/main/downloads/red-green-clicker-1.0.apk)**

[完整源码 ZIP](https://github.com/c3LJ-Netizen/clljj/raw/refs/heads/main/downloads/red-green-clicker-source.zip)

Android 8.0 及以上，无需 root。安装包为自行签名的个人工具，不经过应用商店。没有联网权限，不保存或上传屏幕。iPhone 无法安装 APK。

## 第一次使用

1. 下载并安装 APK，打开“红绿快点”。如果浏览器提示，请允许该浏览器安装这个文件。
2. 点“① 开启无障碍点击”，阅读说明后，在系统里找到“红绿快点”并开启。Android 13 及以上部分手机需先到系统应用信息的右上角菜单，选择“允许受限设置”，再开启无障碍。入口名称以手机系统为准。
3. 返回应用，点“② 开启屏幕采集”。同意系统授权；若出现范围选择，请选**整个屏幕**。Android 14 及以上优先使用系统的全屏授权配置。
4. 切到目标应用。通过悬浮条分别选择**监测点**和**点击点**。监测点放在色块内部，避开文字、边缘和阴影。两个点可以相同。
5. 把悬浮条拖到空白处，不能盖住任何选点。按“开始”。必须先看到红色，之后变绿才点。默认每次开始只点一次。
6. “暂停”停止颜色检测；“停止”、通知里的“停止采集”、以及“收起”会关闭屏幕采集。再次采集需要重新授权。

选点时手指松开即确认，返回键取消。选点窗口消失后再开始识别，避免十字标记影响取色。屏幕旋转或分辨率变化会停止采集，请重新授权并选点。

## 速度与防误触

- 默认极速：1 帧绿色即触发，点击请求无额外等待，手势按压时间 1 ms；系统实际执行不保证 1 ms。
- MediaProjection + ImageReader 连续接收屏幕帧，使用 acquireLatestImage 丢弃积压旧帧，独立线程仅读取监测点周围 3×3 像素，不生成 Bitmap、不压缩图像。也可改为中心 1 像素。
- 采集期间在内存保留最近一帧，开始时也会检查静止画面的当前颜色；暂停时不进行颜色判断。同一帧不会重复计为多帧确认。
- 连续 2 / 3 帧绿色确认可减少误触，但会增加帧等待时间。改设置后点“保存设置”。
- 可开启循环：每次点击完成后等 300 ms，然后必须重新看到红色，再变绿才点。300 ms 只用于重复点击的防抖，不影响首次触发速度。
- 采用 HSV 色相判定红色与绿色，过滤低饱和和过暗像素。适用于清晰红/绿色块；照片、渐变、闪烁、相似背景可能产生误判。先用测试页与实际目标验证。
- 内置测试页：点“设置测试坐标”→移开悬浮条→悬浮条点“开始”→测试页点“3 秒后变绿”。命中后显示从应用提交变色到收到点击的时间，包含渲染、采集和手势调度，手动点击也会被统计，不能当作实验室测量。

**没有承诺固定毫秒延迟。** 采集帧率、屏幕刷新率、系统负载、厂商调度及目标应用会影响反应时间。界面显示的“识别计算 μs”只计算本次读取像素和判色开销，不是变色到点击的总延迟。

## 限制

- 不支持 iOS。浏览器网页无法跨应用点击，所以这里提供原生 Android APK。
- 受 FLAG_SECURE 等系统保护的画面可能是黑屏，无法取色。系统锁屏、其他应用开始投屏或系统回收服务时，采集可能终止；需重新授权。
- 开始时已经是绿色不会触发。色块在开始前已闪红再变绿，也不会追溯触发。
- 坐标对应当前整个屏幕；旋转、应用布局改变、分屏或缩放后要重新选点。
- 这是指定坐标的颜色触发器，不会理解页面含义。请先在内置测试页确认坐标，再用于自己的目标页面。

## 源码构建

源代码无第三方依赖。需要 Python 3、JDK 11+、Android platform 30+、Android Build Tools 35。

设置 `JAVA_HOME`、`ANDROID_JAR`、`ANDROID_BUILD_TOOLS` 三个环境变量，运行 `python build.py`。可用 `COLOR_TAP_BUILD` 指定构建目录。脚本运行纯 Java 逻辑测试，再编译、转换 DEX、对齐、签名和验证 APK。

默认签名密钥在构建目录生成，仅用于个人构建；不会包含在源码压缩包或仓库。正式维护请自行保管签名密钥；不同密钥构建的 APK 不能相互覆盖安装。

最低 Android 8 / API 26，目标 API 35。构建与签名校验、477 项颜色/状态断言已通过；尚未在用户手机或运行中的安卓设备上验证端到端延迟。

系统接口参考：[MediaProjection](https://developer.android.com/media/grow/media-projection)、[AccessibilityService.dispatchGesture](https://developer.android.com/reference/android/accessibilityservice/AccessibilityService#dispatchGesture(android.accessibilityservice.GestureDescription,%20android.accessibilityservice.AccessibilityService.GestureResultCallback,%20android.os.Handler))。
