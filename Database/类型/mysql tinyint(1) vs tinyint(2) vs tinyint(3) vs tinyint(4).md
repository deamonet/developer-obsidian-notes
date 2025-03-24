[mysql tinyint(1) vs tinyint(2) vs tinyint(3) vs tinyint(4) - Stack Overflow](https://stackoverflow.com/questions/13120904/mysql-tinyint1-vs-tinyint2-vs-tinyint3-vs-tinyint4)

What is the difference between:

-   TinyINT(1)
-   TinyINT(2)
-   TinyINT(3)
-   TinyINT(4)

TinyINT(M) always has a range from -128..+127 signed or 0..255 unsigned. M is the display width.

M indicates the maximum display width for integer types. The maximum display width is 255. Display width is unrelated to the range of values a type can contain, as described in Section 11.2, “Numeric Types”. For floating-point and fixed-point types, M is the total number of digits that can be stored.

from [http://dev.mysql.com/doc/refman/5.5/en/numeric-type-overview.html](http://dev.mysql.com/doc/refman/5.5/en/numeric-type-overview.html)

From <[https://stackoverflow.com/questions/13120904/mysql-tinyint1-vs-tinyint2-vs-tinyint3-vs-tinyint4?answertab=trending#tab-top](https://stackoverflow.com/questions/13120904/mysql-tinyint1-vs-tinyint2-vs-tinyint3-vs-tinyint4?answertab=trending#tab-top)>

[agui1989](https://segmentfault.com/u/agui1989)  [发布于 2018-03-20](https://segmentfault.com/q/1010000013845123/a-1020000013845778)

int类型的（包括tinyint,smallint...)后面括号内的数字，一般情况下是不需要专门设置的，默认的就好了。

因为它只与显示有关，和占用的空间无关。

而只有一种情况下，我们需要用到：

当数字的长度小于指定位数时，用0补齐。这时需要结合zerofill使用

比如 tinyint(2) zerofill

如果是3，则显示为 03

如果是122，则显示为 122

如果你不使用zerofill，而括号内的数字随便写，效果是一样的。

[Yujiaao](https://segmentfault.com/u/yujiaao) 更新于 2018-03-20

mysql 中int(1)和tinyint(1)中的1只是指定显示长度，并不表示存储长度.

tinyint可以存储1字节, 即unsigned 0~255(signed -127~127)。显示大小不受此限制 (所有整数类型相同),即使设为1,也可以存入和取出大于10的数。括号里的数字,即显示大小对整数来说主要有两个目的, 一是做为编码文档;将 tinyint (1) 放到表定义中会告诉人们只有数字 0 - 9 应该输入, 表没有被设计具有其他值。

另一个目的是, 你可以与属性ZEROFILL联合使用。ZEROFILL将填充小于显示大小的数字,在他们面前补上零。例如 TINYINT (3) 与 ZEROFILL, 插入值4, 你会得到 004。

参见:

[https://lists.mysql.com/mysql...](https://link.segmentfault.com/?enc=cclFzIRj44X4c63z7SjWdA%3D%3D.FHJ6ddgI3NHCB8QiLxV8njOdpo4VlzN9f%2FrFh57D8XSb4eXKyTGj07r5aC90qXlX)

[http://www.cnblogs.com/xiaoch...](https://link.segmentfault.com/?enc=TrZolPK9%2BLEpbafKQkisYw%3D%3D.tbFW8%2BdkR4ZCt4JdYU3t2CY08JEKHNTcvB8AJAoVSplAAa%2FmZEGlTorcaSu%2FJAu%2FdBQa9hq9lmi9j1A9Esxjl3op0AzAEOvRCgQA1O4mJWg%3D)

From <[https://segmentfault.com/q/1010000013845123](https://segmentfault.com/q/1010000013845123)>