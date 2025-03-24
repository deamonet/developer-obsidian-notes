当 join 两张表的字段名相同时，可以改成 USING(字段名)


`USING`是在MySQL中使用`join`時可以使用，當兩個要`join`的資料表中，用在`on`的欄位名稱相同時，就可以用`using`代替，如：  
在A表中有欄位C1，在B表中也有欄位C1，一般在join A和B兩個Table時，會寫成：  

[?](http://bioankeyang.blogspot.com/2012/03/mysqlusing.html#)

1

`select` `a.XX, b.yy` `from` `A a` `inner` `join` `B b` `on` `a.C1 = b.C1`

但當`join`用的欄位名稱相同時，如上列的`C1`，就可以寫成：  

[?](http://bioankeyang.blogspot.com/2012/03/mysqlusing.html#)

1

`select` `a.XX, b.yy form A a` `inner` `join` `B b using(C1)`