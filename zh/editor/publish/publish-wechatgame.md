# 发布到微信小游戏
这篇文档将会覆盖：

- 微信小游戏的运行环境介绍
- 如何使用 Cocos Creator 3D发布微信小游戏
- 小游戏资源管理

## 微信小游戏的运行环境介绍

微信小游戏是微信小程序下的游戏产品平台，它不仅提供了强大的游戏能力，还和小程序一样，提供了大量的原生接口，比如支付，社交，文件系统，照片，NFC 等。相当于同时结合了 WEB 易于传播以及 Native 功能丰富的优势。

小游戏的运行环境是小程序环境的扩展，基本思路也是封装必要的 WEB 接口提供给用户，尽可能追求和 WEB 同样的开发体验。小游戏在小程序环境的基础上提供了 WebGL 接口的封装，使得渲染能力和性能有了大幅度提升。不过由于这些接口都是微信团队通过自研的原生实现封装的，所以并不可以等同为浏览器环境。

作为引擎方，为了尽可能简化开发者的工作量，我们为用户完成的主要工作包括：

- 引擎框架适配微信小游戏 API，纯游戏逻辑层面，用户不需要任何额外的修改
- Cocos Creator 3D 编辑器提供了快捷的打包流程，直接发布为微信小游戏，并自动唤起小游戏的开发者工具
- 自动加载远程资源，缓存资源以及缓存资源版本控制

除此之外，小游戏的游戏提交，审核和发布流程和小程序是没有区别的，都需要遵守微信团队的要求和标准流程，具体信息可以参考文末的链接。

## 使用 Cocos Creator 3D 发布微信小游戏

1. 在 [微信公众平台](https://mp.weixin.qq.com/debug/wxagame/dev/devtools/download.html) 下载微信开发者工具
2. 在编辑器菜单栏的 **偏好设置 -> [原生开发环境]()** 中设置微信开发者工具路径

    ![](./publish-wechatgame/preference.png)
3. 登陆微信公众平台，找到 appid

    ![](./publish-wechatgame/appid.jpeg)
4. 在 **构建发布** 面板的 **发布平台** 中选择 **微信小游戏**，填入小游戏 appid，然后点击 **构建**

    ![](./publish-wechatgame/build.png)
5. 点击 **运行** 打开微信开发者工具

    ![](./publish-wechatgame/tool.jpeg)
**注意**：微信开发者工具，如果之前在点击上没运行过，会出现：`Please ensure that the IDE has been properly installed` 的报错。需要手动打开一次微信开发者工具，然后才能在 Cocos Creator 3D 里直接点击 **运行** 调用。
6. 预览部署

按照这样的流程，项目的 build 目录下就会生成一个微信小游戏的发布包 **wechatgame** 文件夹(具体构建任务名为准），其中已经包含了微信小游戏环境的配置文件：`game.json` 和 `project.config.json`

![](./publish-wechatgame/package.jpeg)

## 构建选项介绍

参数名 | 可选 | 默认值 | 说明
- | - | - | -
appid | 必填 | 'wx6ac3f5090a6b99c5' | 微信小程序 appid，填写后将会写入在 `project.config.json` 内。
remoteServerAddress | 选填 | ' ' | 远程服务器地址，填写后获取资源将会从该路径上获取
subContext | 选填 | ' ' | 子域文件夹，相对于构建最终包体的路径，将会在构建之后拷贝到结果内
orientation | 必填 | 'landscape' | 设备方向，填写后将会写入在 `game.json` 内。
分包 | 可选 | true | 是否开启分包功能

平台相关的配置选项都是根据平台的支持来的，这里只提供了部分数据的界面修改，具体参数请参考[微信小游戏配置文档](https://developers.weixin.qq.com/minigame/dev/reference/configuration/app.html#%E9%85%8D%E7%BD%AE%E9%A1%B9)。

## 小游戏环境的资源管理

在小游戏环境中，资源管理是最特殊的部分，它和浏览器的不同在于下面四点：

1. 小游戏的包内体积不能够超过 4MB，包含所有代码和资源，额外的资源必须通过网络请求下载。
2. 对于从远程服务器下载的文件，小游戏环境没有浏览器的缓存以及过期更新机制。
3. 对于小游戏包内资源，小游戏环境内并不是按需加载的，而是一次性加载所有包内资源，然后再启动页面。
4. 不可以从远程服务器下载脚本文件。

这里引出了两个关键的问题，首页面加载速度和远程资源缓存及版本管理。对于首页面加载速度，我们建议用户只保存脚本文件在小游戏包内，其他资源都从远程服务器下载。而远程资源的下载、缓存和版本管理，其实在 Cocos Creator 3D 中，已经帮用户做好了。下面我就来解释一下这部分的逻辑。

在小游戏环境中，我们提供了一个 wxDownloader 对象，给它设置了 `REMOTE_SERVER_ROOT` 属性后，引擎下载资源的逻辑就变成：

1. 检查资源是否在小游戏包内
2. 不存在则查询本地缓存资源
3. 如果没有缓存就从远程服务器下载
4. 下载后保存到小游戏应用缓存内供再次访问时使用
5. 缓存空间有大小限制，如果超出限制则会保存失败，此时打印提示信息并使用资源下载时的临时文件作为资源

**注意**：需要额外注意的是，一旦缓存空间占满之后，所有需要下载的资源都无法进行保存，只能使用下载保存的临时文件，而微信会在小游戏退出之后自动清理所有临时文件，所以下次再次运行小游戏时，这些资源又会再度下载，然后一直循环往复此过程。另外，缓存空间超出限制导致文件保存失败的问题不会在微信开发者工具上出现，因为微信开发者工具没有限制缓存大小，所以测试缓存时需要真实微信环境进行测试。

同时，当开启引擎的 md5Cache 功能后，文件的 url 会随着文件内容的改变而改变，这样当游戏发布新版本后，旧版本的资源在缓存中就自然失效了，只能从服务器请求新的资源，也就达到了版本控制的效果。

具体来说，开发者需要做的是：

1. 构建时，在 **构建发布配置** 面板中勾选 md5Cache 功能。
2. 设置 **远程服务器地址**，然后点击 **构建**。
3. 构建完成后将微信小游戏发布包目录下的 res 文件夹完整的上传到服务器。
4. 删除本地发布包目录下的 res 文件夹。
5. 对于测试阶段来说，可能用户无法部署到正式服务器上，需要用本地服务器来测试，那么请在微信开发者工具中打开 **详情** 页面，勾选项目设置中的 **不检验安全域名、TLS 版本以及 HTTPS 证书** 选项。

![](./publish-wechatgame/detail.jpeg)

**注意**：如果缓存资源超过微信环境限制，用户需要手动清除资源，可以在微信小游戏下使用 `wx.downloader.cleanAllAssets()` 和 `wx.downloader.cleanOldAssets()` 接口来清除缓存。前者会清除缓存目录下的所有缓存资源，请慎重使用；而后者会清除缓存目录下目前应用中未使用到的缓存资源。

## 平台 SDK 接入

除了纯游戏内容以外，其实微信小游戏环境还提供了非常强大的原生 SDK 接口，其中最重要的就是用户、社交、支付等，这些接口都是仅存在于微信小游戏环境中的，等同于其他平台的第三方 SDK 接口。这类 SDK 接口的移植工作在现阶段还是需要开发者自己处理。下面列举一些微信小游戏所提供的强大 SDK 能力：

1. 用户接口：登陆，授权，用户信息等
2. 微信支付
3. 转发以及获得转发信息
4. 文件上传下载
5. 媒体：图片、录音、相机等
6. 其他：位置、设备信息、扫码、NFC、等等

## 接入微信小游戏的开放数据域

微信小游戏为了保护其社交关系链数据，增加了 **开放数据域** 的概念，这是一个单独的游戏执行环境。开放数据域中的资源、引擎、程序，都和主游戏完全隔离，开发者只有在开放数据域中才能访问微信提供的 `wx.getFriendCloudStorage()` 和 `wx.getGroupCloudStorage()` 两个 API，用于实现一些例如排行榜的功能。

详情请参考 [接入微信小游戏的开放数据域](../publish/publish-wechatgame-subdomain.md)。

## 微信小游戏已知问题：

我们对微信小游戏的适配工作还未完全结束，目前仍不支持以下组件：

- VideoPlayer
- WebView

用户如果有需要，目前可以先自己直接调用微信的 API 来使用。

另外目前我们对微信分包的适配还有一些问题，如果在分包内使用了其他的组件脚本会无法加载，这个问题我们将会在下个版本修复，如果你有使用到这个功能请关注一下官方的更新日志或者是论坛上联系我们，我们会给予一个临时的解决方案。

## 参考链接

- [微信小游戏开发文档](https://mp.weixin.qq.com/debug/wxagame/dev/index.html)
- [微信公众平台](https://mp.weixin.qq.com/)
- [小游戏 API 文档](https://developers.weixin.qq.com/minigame/dev/document/render/canvas/wx.createCanvas.html)
- [微信开发者工具下载](https://mp.weixin.qq.com/debug/wxagame/dev/devtools/download.html)
- [微信开发者工具文档](https://developers.weixin.qq.com/minigame/dev/devtools/devtools.html)
- [微信缓存空间溢出测试案例](https://github.com/cocos-creator/WeChatMiniGameTest)
