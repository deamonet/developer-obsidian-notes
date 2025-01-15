---
tags:
  - du
---
```shell
du -hs * | sort -hr | head

70M     chapter04
30M     chapter02
580K    chapter03
48K     chapter06
48K     chapter01
36K     chapter05
16K     README.md
8.0K    chapter07
0       requirements.txt

# head是显示前10个

du --help
-h, --human-readable  print sizes in human readable format (e.g., 1K 234M 2G)
-s, --summarize       display only a total for each argument

sort --help
-h, --human-numeric-sort    compare human readable numbers (e.g., 2K 1G)
-r, --reverse               reverse the result of comparisons

# 从大到小排序,可以用--max-depth指定文件夹深度
du --max-depth 1 -h | sort -hr

# 指定目录:
du /tmp -hs * | sort -hr

```
#du 