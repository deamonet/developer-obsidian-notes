队列是一种特殊的线性表，它只允许在表的前端（front）进行删除操作，而在表的后端（rear）进行插入操作。进行插入操作的端称为队尾，进行删除操作的端称为队头。队列中没有元素时，称为空队列。

在队列这种数据结构中，最先插入的元素将是最先被删除的元素；反之最后插入的元素将是最后被删除的元素，因此队列又称为“先进先出”（FIFO—first in first out）的线性表。

在java5中新增加了java.util.Queue接口，用以支持队列的常见操作。该接口扩展了java.util.Collection接口。

Queue继承了Collection接口

Queue使用时要尽量避免Collection的add()和remove()方法，add()和remove()方法在失败的时候会抛出异常。

要使用offer()来加入元素，使用poll()来获取并移出元素。

它们的优点是通过返回值可以判断成功与否， 如果要使用前端而不移出该元素，使用element()或者peek()方法。

值得注意的是LinkedList类实现了Queue接口，因此我们可以把LinkedList当成Queue来用。  
　　add         增加一个元索                          如果队列已满，则抛出一个IIIegaISlabEepeplian异常  
　　remove   移除并返回队列头部的元素    如果队列为空，则抛出一个NoSuchElementException异常  
　　element  返回队列头部的元素               如果队列为空，则抛出一个NoSuchElementException异常  
　　offer        添加一个元素并返回true        如果队列已满，则返回false  
　　poll         移除并返问队列头部的元素    如果队列为空，则返回null  
　　peek       返回队列头部的元素               如果队列为空，则返回null  
　　put          添加一个元素                          如果队列满，则阻塞  
　　take        移除并返回队列头部的元素    如果队列为空，则阻塞