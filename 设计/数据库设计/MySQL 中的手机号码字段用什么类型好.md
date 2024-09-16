  

   [iugo](https://www.v2ex.com/member/iugo) · 2014-10-21 16:13:25 +08:00 · 51818 次点击

这是一个创建于 3029 天前的主题，其中的信息可能已经有所发展或是发生改变。

需要根据手机号码查询订单, 看到别人在用 char(11) 来存储手机号码.  
  
字符串的效率不是应该比数字差吗? 为什么不用 int 来储存?  
  
不过手机号码的数字就像姓名一样含义应该是字符串的, 只不过长成个数字的外貌.  
  
对于手机号这种数据来说, 是否 char 是最好的选择?

 [字符串](https://www.v2ex.com/tag/%E5%AD%97%E7%AC%A6%E4%B8%B2) [char](https://www.v2ex.com/tag/char) [字段](https://www.v2ex.com/tag/%E5%AD%97%E6%AE%B5)

15 条回复  **•**  2014-10-21 21:08:30 +08:00

![wy315700](media/wy315700.png)

    1

**[wy315700](https://www.v2ex.com/member/wy315700)**   

   2014-10-21 16:23:26 +08:00    ![❤️](media/❤️.png) 1

你存INT以后 还怎么按照开头或者尾号查询

![learnshare](media/learnshare.png)

    2

**[learnshare](https://www.v2ex.com/member/learnshare)**   

   2014-10-21 16:25:33 +08:00

它就是个字符串，不是数字啊

![HunterPan](media/HunterPan.png)

    3

**[HunterPan](https://www.v2ex.com/member/HunterPan)**   

   2014-10-21 16:26:03 +08:00

必须字符串

![staticor](media/staticor.png)

    4

**[staticor](https://www.v2ex.com/member/staticor)**   

   2014-10-21 16:58:58 +08:00

字符串吧 前x位可作区域码 虽然可能用不到

![holyghost](media/holyghost.png)

    5

**[holyghost](https://www.v2ex.com/member/holyghost)**   

   2014-10-21 17:03:29 +08:00

因为int存不下11位呀

![caixiexin](media/caixiexin.png)

    6

**[caixiexin](https://www.v2ex.com/member/caixiexin)**   

   2014-10-21 17:15:49 +08:00

用int的话，万一对这串手机号有特殊操作咋办？比如正则匹配，取号头啥的

![iugo](media/iugo.png)

    7

**[iugo](https://www.v2ex.com/member/iugo)**   

OP

   2014-10-21 17:46:10 +08:00

@[wy315700](https://www.v2ex.com/member/wy315700)  
@[learnshare](https://www.v2ex.com/member/learnshare) 谢谢.  
@[HunterPan](https://www.v2ex.com/member/HunterPan) 谢谢.  
@[staticor](https://www.v2ex.com/member/staticor) 谢谢,  
@[holyghost](https://www.v2ex.com/member/holyghost)  
@[caixiexin](https://www.v2ex.com/member/caixiexin)  
  
谢谢, 手机号码的本质是字符串, 虽然是纯数字组成的.  
还能实现将来可能的区位查询.  
是的, INT 无法存下手机号, 还要 BIGINT 才行.  
  
结论: 看来 char 储存手机号是一种符合逻辑, 常用的方法.

![iugo](media/iugo.png)

    8

**[iugo](https://www.v2ex.com/member/iugo)**   

OP

   2014-10-21 17:47:29 +08:00

@[wy315700](https://www.v2ex.com/member/wy315700) 谢谢.  
@[holyghost](https://www.v2ex.com/member/holyghost) 谢谢.  
@[caixiexin](https://www.v2ex.com/member/caixiexin) 谢谢.  
不好意思, 本来上帖要修改为一并感谢的, 没写好.

![jasontse](media/jasontse.jpg)

    9

**[jasontse](https://www.v2ex.com/member/jasontse)**   

   2014-10-21 17:55:12 +08:00 via iPad

都存一遍，爱怎么查效率都高。

![zhujinliang](media/zhujinliang.png)

    10

**[zhujinliang](https://www.v2ex.com/member/zhujinliang)**   

   2014-10-21 18:10:17 +08:00 via iPhone    ![❤️](media/❤️.png) 1

话说身份证号为什么会有个x，就是怕程序员存成整数型

![learnshare](media/learnshare.png)

    11

**[learnshare](https://www.v2ex.com/member/learnshare)**   

   2014-10-21 18:13:02 +08:00

@[zhujinliang](https://www.v2ex.com/member/zhujinliang) 整数型也太长了...

![caixiexin](media/caixiexin.png)

    12

**[caixiexin](https://www.v2ex.com/member/caixiexin)**   

   2014-10-21 18:41:05 +08:00

另外，如果确定是11位的可以用char，要是长度，格式不确定，比如带+86格式的手机号什么的，用varchar，长度设长一点，拓展性更好。

![gDD](media/gDD.png)

    13

**[gDD](https://www.v2ex.com/member/gDD)**   

   2014-10-21 18:48:13 +08:00 via iPhone

@[zhujinliang](https://www.v2ex.com/member/zhujinliang) 看不出来时开玩笑，最后一位是校验位，用X是逼不得已的…

![wy315700](media/wy315700.png)

    14

**[wy315700](https://www.v2ex.com/member/wy315700)**   

   2014-10-21 19:42:53 +08:00

@[zhujinliang](https://www.v2ex.com/member/zhujinliang)  
  
1。那个是罗马字母X，也就是10，不是因为字母x  
2。为什么会出现X，是因为这一位是前面17位运算以后除以11得出来的校验值。

![oott123](media/oott123.png)

    15

**[oott123](https://www.v2ex.com/member/oott123)**   

   2014-10-21 21:08:30 +08:00 via Android

@[gDD](https://www.v2ex.com/member/gDD) 也不算迫不得已吧，毕竟对9取余就行了…