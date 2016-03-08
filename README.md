aapt-build
==========
this work is unfinished.

看到有fork，所以觉得应该把README写清楚，以下是当初在Window下的尝试过程：

一、准备
1、获取源码。android开放源码存放在git库中： https://android.googlesource.com/
选择项目  platform/frameworks/base 
选择 TAG  android-4.4.4_r2.0.1（最新的tag）
寻找aapt  tools/aapt/
2、搭建编译环境
当前开发环境为 Windows + Eclipse(in ADT), 要支持C++编译，还需要安装插件和编译器。
>在Eclipse中安装CDT(C/C++ Development Toolkit)插件；
>安装MinGW.

二、解决依赖问题
     在Eclipse中用向导新建C++工程，导入源码。在CDT语法分析之后，报告出一些错误。出现大批语法错误和多处引用头文件找不到：#include \<utils/\*\*\*.h> (其中\*\*\*泛指头文件名)。可见aapt源码要依赖其他的工具或者库，不能独立编译。那么，去找这个utils吧。
     还是去源码库，aapt/目录下没有，遍历父目录tools/，也没有。再上一层，到了根目录。这时有个子目录映入眼帘：include/ ，应该是它了，进去看看，没有！那么，搜索吧，搜索...结果还是没有，整个源码集中都找不到，那这个utils/是何方神圣？是个什么库吗？google之。
     搜索发现，在早期版本的源码中是存在utils/这个目录的，是从哪个TAG开始丢掉了呢？从4.4.4往前一个个翻出来看吧。没有，没有，...，有了！版本： 4.0.1_r1 。选定这个TAG，获取到 tools/aapt/ 和 include/utils/ 。
     工程清理，重新导入源码。嗯，现在utils又出现大批语法错误和多处引用头文件找不到。其中有#include \<cutils/\*\*\*.h>
，去找cutils/ 。在platform/frameworks/base中找了好久，最终发现是在另外一个项目中：platform/system/core/./include/cutils/ .嗯，好了，找到，添加进去，检查，坏了，cutils也要报错。针对报错的文件，进行以下处理：
>没有被引用到的，删掉；
>宏定义错误的，修改宏定义；
>类型、常量、方法找不到的，注释掉；
解决一大堆编译红叉之后，变得稍微有些条理。
cutils(.h & .cpp)依赖的头文件缺少:
sys/cdefs.h
sys/socket.h
sys/ioctl.h
sys/mount.h
sys/wait.h
netinet/in.h
machine/cpu-features.h

utils(.h & .cpp)依赖的头文件缺少:
linux/ioctl.h
linux/limits.h
netinet/in.h
android/looper.h
android/configuration.h
sys/epoll.h
sys/poll.h
sys/uio.h
zlib.h
system/graphics.h
private/utils/Static.h

aapt源码依赖的头文件缺少:
png.h
zipfile/zipfile.h
expat.h
host/pseudolocalize.h
zlib.h

接下来，就把这些头文件一个个找出来补充上。
先找 private/utils/Static.h，因为刚才在找别的文件时，好像见过它。果然，就在utils同级目录中发现private，把它添加到工程中，一个搞定。
然后 system/graphics.h，在system/core/./include/中。
然后 zipfile/zipfile.h，在system/core/./include/中。
然后 android/looper.h ，在frameworks/base/native/include/中。

还有linux/,netinet/等，这些是Linux平台的底层库，参考了 http://winuxxan.blog.51cto.com/2779763/502340 ，看来还是要在Linux平台来做交叉编译了，路还很长啊。


【To Be Continued】

