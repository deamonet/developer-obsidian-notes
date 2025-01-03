尽管某些书籍上总是说避免使用全局变量，但是在实际的需求不断变化中，往往定义一个全局变量是最可靠的方法，但是又必须要避免变量名覆盖。

Python 中 global 关键字可以定义一个变量为全局变量，但是这个仅限于在一个模块（py文件）中调用全局变量：

我们知道Python使用变量的时候是可以直接使用的，x=[] ,y=2,z="123",而不需要先定义（var x; var y=2;var z='222'），这样的话，在函数内部就无法操作外部的变量了，因为它总会认为你是在定义一个新变量并且赋值，不过 global 就可以解决这个问题。

global 的基础用法 ：

```python
x = 6
def func():
    global x #定义外部的x
    x = 1
 
func()
print (x)
#输出1

```

这个时候，即使你在另外一个py文件 再次使用 global x 也是无法访问到的，因为在这个py模块中并没有一个叫做x的变量，于是就会报错 未定义。

### 那么我们怎么办？

借鉴global 关键字的思路，既然在一个文件里面可以生效的话，那么我们就专门为全局变量定义一个“全局变量管理模块”就好了

新建一个py文件，文件名是：gol.py，具体代码：
```python
 
def _init():#初始化
    global _global_dict
    _global_dict = {}
 
 
def set_value(key,value):
    """ 定义一个全局变量 """
    _global_dict[key] = value
 
 
def get_value(key,defValue=None):
　　""" 获得一个全局变量,不存在则返回默认值 """
    try:
        return _global_dict[key]
    except KeyError:

```
相信如果你看懂了就应该知道思路了，利用global的单独文件全局性，从而可以定义在一个文件中的全局变量，然后这个单个文件的全局变量可以保存多个文件的共同全局变量

操作的时候，以Key对Value 的方法操作，我相信大家都懂。
 
```python
import gol
 
gol._init()#先必须在主模块初始化（只在Main模块需要一次即可）
 
 
#定义跨模块全局变量
gol.set_value('uuid',uuid)
gol.set_value('token',token)
```

然后其他的任何文件只需要导入即可使用：
```python
import gol
 
#不需要再初始化了
ROOT = gol.get_value('uuid')
CODE = gol.get_value('token')
```
就这样就可以实现跨文件的全局变量使用；

并且还有一个简单但是强大的全局变量管理器，你可以自己添油加醋，实现一些比如全局变量禁止直接修改，禁止修改某些只可读的全局变量等等。

|                                                                                                                                                                                                                                                 |     |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| 1.作者：[Syw](http://www.cnblogs.com/syw20170419/)  <br>2.出处：[http://www.cnblogs.com/syw20170419/](http://www.cnblogs.com/syw20170419/)  <br>3.本文版权归作者和博客园共有，欢迎转载，但未经作者同意必须保留此段声明，且在文章页面明显位置给出原文连接，否则保留追究法律责任的权利。  <br>4.如果文中有什么错误，欢迎指出。以免更多的人被误导。 |     |