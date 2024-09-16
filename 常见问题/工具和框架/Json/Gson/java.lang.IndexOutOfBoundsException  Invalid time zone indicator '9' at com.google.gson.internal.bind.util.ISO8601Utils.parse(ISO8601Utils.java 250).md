gson 不能处理太长的unix-timestamp，离了个大普。

解决办法是，重新写一个date的反序列化逻辑，或者直接用jackson.


参考：
[java - Convert String date to Object yields Invalid time zone indicator '0' - Stack Overflow](https://stackoverflow.com/questions/41958062/convert-string-date-to-object-yields-invalid-time-zone-indicator-0)

[Gson报错Invalid time zone indicator ‘ ‘_超级侠哥的博客-CSDN博客](https://blog.csdn.net/znb769525443/article/details/109751708)
[Failed to parse date "1534467411000":Invalid time zone indicator '0'_Think_Bigger的博客-CSDN博客](https://blog.csdn.net/uniquewonderq/article/details/103037400)

[Invalid time zone indicator ' ' and the solution given didn't work · Issue #2002 · google/gson (github.com)](https://github.com/google/gson/issues/2002)

[Invalid time zone indicator ' ' · Issue #1297 · google/gson (github.com)](https://github.com/google/gson/issues/1297)
