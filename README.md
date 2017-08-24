# iOS Objc 方法调用记录插件: iTracer v1.3 

[![Join the chat at https://gitter.im/waruqi/tboox](https://badges.gitter.im/waruqi/tboox.svg)](https://gitter.im/waruqi/tboox?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge) [![donate](http://tboox.org/static/img/donate.svg)](http://tboox.org/donation/)

如果你想逆向 某些app的调用流程 或者 系统app的一些功能的 私有 framework class api 调用流程， 可以试试此工具

只需要 配置需要挂接的 类名和app名， 就可以实时追踪 相关功能的 调用流程。 支持批量 hook n多个类名

#### 特性

* 批量跟踪ios下指定class对象的所有调用流程
* 支持ios for armv6,armv7,arm64 以及mac for x86, x64
* 自动探测参数类型，并且打印所有参数的详细信息

#### 更新内容

* 增加对arm64的支持，刚调通稳定性有待测试。
   arm64进程注入没时间做了，暂时用了substrate的hookprocess， 所以大家需要先装下libsubstrate.dylib
   armv7的版本是完全不依赖substrate的。
 
* arm64的版本对参数的信息打印稍微做了些增强。

注：此项目已不再维护，仅供参考。

1. 配置需要挂接的class

    ```
    修改itrace.xml配置文件，增加需要hook的类名：
    <?xml version="1.0" encoding="utf-8"?>
    <itrace>
      <class>
        <SSDevice/>
        <SSDownload/>
        <SSDownloadManager/>
        <SSDownloadQueue/>
        <CPDistributedMessagingCenter/>
        <CPDistributedNotificationCenter/>
        <NSString args="0"/>
      </class>
    </itrace>
    ```

    注： 尽量不要去hook， 频繁调用的class， 比如 UIView NSString， 否则会很卡，操作就不方便了。
    注： 如果挂接某个class， 中途打印参数信息挂了， 可以在对应的类名后面 加上 args="0" 属性， 来禁止打印参数信息， 这样会稳定点。 
         如果要让所有类都不打印参数信息， 可以直接设置： <class args="0">


2. 安装文件

    将整个itracer目录下的所有文件用手机助手工具，上传到ios系统上的 /tmp 下面：
    ```
    /tmp/itracer
    /tmp/itrace.dylib
    /tmp/itrace.xml
    ```

3. 进行trace

    进入itracer所在目录：
    ```
      cd /tmp
      ```

    修改执行权限：
    ```
      chmod 777 ./itracer
    ```

    运行程序: 

    ```
      ./itracer springboard (spingboard 为需要挂接的进程名， 支持简单的模糊匹配)
    ```

4. 使用substrate进行注入`itrace.dylib`来trace

    在ios arm64的新设备上，使用`itracer`注入`itrace.dylib`已经失效，最近一直没怎么去维护，如果要在在arm64上注入进行trace，可以借用substrate，将`itrace.dylib`作为substrate插件进行注入
    再配置下`itrace.plist`指定需要注入到那个进程中就行了，具体可以看下substrate的插件相关文档。

    放置`itrace.dylib`和`itrace.plist`到substrate插件目录`/Library/MobileSubstrate/DynamicLibraries`，然后重启需要trace的进程即可, `itrace.xml`的配置文件路径变为`/var/root/itrace/itrace.xml`。
    
    ios arm64设备的`itrace.dylib` 编译时使用 `xmake f -p iphoneos -a arm64`命令。

5. 查看 trace log， 注： log 的实际输出在： 控制台-设备log 中：

    ```
    Jan 21 11:12:58 unknown SpringBoard[5706] <Warning>: [itrace]: [3edc9d98]: [SSDownloadQueue downloads]
    Jan 21 11:12:58 unknown SpringBoard[5706] <Warning>: [itrace]: [3edc9d98]: [SSDownloadManager downloads]
    Jan 21 11:12:58 unknown SpringBoard[5706] <Warning>: [itrace]: [3edc9d98]: [SSDownloadManager _copyDownloads]
    Jan 21 11:12:58 unknown SpringBoard[5706] <Warning>: [itrace]: [3edc9d98]: [SSDownloadQueue _sendDownloadStatusChangedAtIndex:]: 0
    Jan 21 11:12:58 unknown SpringBoard[5706] <Warning>: [itrace]: [3edc9d98]: [SSDownloadQueue _messageObserversWithFunction:context:]: 0x334c5d51: 0x2fe89de0
    Jan 21 11:12:58 unknown SpringBoard[5706] <Warning>: [itrace]: [3edc9d98]: [SSDownloadQueue downloads]
    Jan 21 11:12:58 unknown SpringBoard[5706] <Warning>: [itrace]: [3edc9d98]: [SSDownloadManager downloads]
    Jan 21 11:12:58 unknown SpringBoard[5706] <Warning>: [itrace]: [3edc9d98]: [SSDownloadManager _copyDownloads]
    Jan 21 11:12:58 unknown SpringBoard[5706] <Warning>: [itrace]: [3edc9d98]: [SSDownload cachedApplicationIdentifier]
    Jan 21 11:12:58 unknown SpringBoard[5706] <Warning>: [itrace]: [3edc9d98]: [SSDownload status]
    Jan 21 11:12:58 unknown SpringBoard[5706] <Warning>: [itrace]: [3edc9d98]: [SSDownload cachedApplicationIdentifier]
    Jan 21 11:12:58 unknown SpringBoard[5706] <Warning>: [itrace]: [3edc9d98]: [CPDistributedNotificationCenter postNotificationName:userInfo:]: SBApplicationNotificationStateChanged: {
          SBApplicationStateDisplayIDKey = "com.apple.AppStore";
          SBApplicationStateKey = 2;
          SBApplicationStateProcessIDKey = 5868;
          SBMostElevatedStateForProcessID = 2;
      }
    Jan 21 11:12:58 unknown SpringBoard[5706] <Warning>: [itrace]: [3edc9d98]: [CPDistributedNotificationCenter postNotificationName:userInfo:toBundleIdentifier:]: SBApplicationNotificationStateChanged: {
          SBApplicationStateDisplayIDKey = "com.apple.AppStore";
          SBApplicationStateKey = 2;
          SBApplicationStateProcessIDKey = 5868;
          SBMostElevatedStateForProcessID = 2;
      }: null
    Jan 21 11:12:59 unknown SpringBoard[5706] <Warning>: [itrace]: [105d7000]: [SSDownloadManager _handleMessage:fromServerConnection:]: 0xe6920b0: 0xe007040
    Jan 21 11:12:59 unknown SpringBoard[5706] <Warning>: [itrace]: [105d7000]: [SSDownloadManager _handleDownloadStatesChanged:]: 0xe6920b0
    Jan 21 11:12:59 unknown SpringBoard[5706] <Warning>: [itrace]: [105d7000]: [SSDownloadManager _copyDownloads]
    Jan 21 11:12:59 unknown SpringBoard[5706] <Warning>: [itrace]: [105d7000]: [SSDownload persistentIdentifier]
    Jan 21 11:12:59 unknown SpringBoard[5706] <Warning>: [itrace]: [105d7000]: [SSDownload _addCachedPropertyValues:]: {
          I = SSDownloadPhaseDownloading;
      }
    Jan 21 11:12:59 unknown SpringBoard[5706] <Warning>: [itrace]: [105d7000]: [SSDownload _applyPhase:toStatus:]: SSDownloadPhaseDownloading: <SSDownloadStatus: 0xe6b8e80>
    Jan 21 11:12:59 unknown SpringBoard[5706] <Warning>: [itrace]: [105d7000]: [SSDownloadQueue downloadManager:downloadStatesDidChange:]: <SSDownloadManager: 0x41ea60>: (
          "<SSDownload: 0xe6bd970>: -4085275246093726486"
      )
    ```

####如何编译

1. 编译ios版本：

```bash
xmake f -p iphoneos
xmake 
```
```bash
xmake f -p iphoneos -a arm64
xmake 
```

2. 编译macosx版本：

```bash
xmake f -p macosx
xmake 
```

更详细的xmake使用，请参考：[xmake文档](http://xmake.io/#/zh/)

依赖库介绍：[tbox](https://github.com/waruqi/tbox/wiki)

#### 联系方式

* 邮箱：[waruqi@gmail.com](mailto:waruqi@gmail.com)
* 主页：[TBOOX开源工程](http://www.tboox.org/cn)
* 社区：[TBOOX开源社区](http://www.tboox.org/forum)
* QQ群：343118190(TBOOX开源工程), 260215194 (ihacker ios逆向分析)
* 微信公众号：tboox-os

