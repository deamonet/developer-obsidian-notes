# idea右边找不到maven窗口(Idea_最右侧常用栏中没有Maven选项)

发布于2021-09-26 10:55:17阅读 2.1K0

方案一： 首先idea自带了maven控件，不像Eclipse还需要下载控件，如果你以前有maven在右边，出于某种原因，消失找不到 了，你可以试试我写的方法。

方法1.你点击一下你idea界面最左下角的那个小框，maven应该从里面找到

方法2.点击菜单栏View->Tool Windows->Maven projects

方法3.点击菜单栏Help->Find Action(Ctrl+Shift+A),输入Maven projects

https://blog.csdn.net/huajuanaini/article/details/51793336?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-2.control

方案二： 右侧边栏没有出现maven, 还有一种可能就是pom.xml文件没有识别, idea觉得这个项目就不是个maven项目，导致idea无法加载依赖包。因此上述三种方法都没有用， 解决办法: 右键pom.xml文件, 点击" add as maven project "

1.鼠标左键选中工程，使用快捷键Shift + Ctrl + A，然后输入maven，选中如图所示的Add Maven Projects选项

![](media/9a86eea2a9ba79591a9140ab2b09bb86.png);

2.在弹出框中选中该工程的pom文件，点击ok即可

![](media/4eedd804fc29ee6379f34b7d03b68434.png);

3.右侧伸缩栏中的maven选项即会出现

![](media/f7506bf7a9abdecc6976cbd137b07d8e.png);

本文参与 [腾讯云自媒体分享计划](https://cloud.tencent.com/developer/support-plan) ，欢迎热爱写作的你一起参与！

本文分享自作者个人站点/博客：https://my.oschina.net/botkenni复制

如有侵权，请联系 cloudcommunity@tencent.com 删除。

[IDE](https://cloud.tencent.com/developer/tag/10278?entry=article)[Maven](https://cloud.tencent.com/developer/tag/10300?entry=article)[编程算法](https://cloud.tencent.com/developer/tag/10663?entry=article)[XML](https://cloud.tencent.com/developer/tag/10203?entry=article)
