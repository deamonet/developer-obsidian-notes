# REST - PUT与POST

已经观察到许多人在设计系统时很难在**HTTP PUT和POST**方法之间进行选择。尽管如此，[RFC 2616](https://www.ietf.org/rfc/rfc2616.txt)在区分这两者方面已经非常明确了 - 但复杂的措辞却让我们很多人感到困惑。让我们尝试解决**何时使用PUT或POST**的难题。

让我们比较它们以便更好地理解。

PUT

POST

RFC-2616明确提到`PUT`对所包含实体的方法请求存储在提供的[Request-URI](http://restful.p2hp.com/home/resource-naming)下。如果Request-URI引用已存在的资源 - 将发生更新操作，否则如果Request-URI是有效资源URI（假设允许客户端确定资源标识符），则应该发生create操作。`PUT/questions/{question-id}`

该`POST`方法用于请求源服务器接受请求中包含的实体作为请求行中的Request-URI标识的资源的新下级。它本质上意味着`POST` request-URI应该是一个集合URI。`POST/questions`

`PUT`方法是幂等的。因此，如果您多次发送重试请求，那应该等同于单个请求修改。

`POST`不是幂等的。因此，如果您重试请求N次，您将最终拥有N个资源，其中N个不同的URI在服务器上创建。

`PUT`当您想要修改已经属于资源集合的单一资源时使用。PUT完全替换资源。如果请求更新资源的一部分，请使用PATCH。

`POST`要在资源集合下添加子资源时使用。

虽然`PUT`是幂等的，但我们不会缓存它的反应。

除非响应包含适当的Cache-Control或Expires头字段，否则对此方法的响应不可[缓存](http://restful.p2hp.com/learn/caching)。但是，303（请参阅其他）响应可用于指示用户代理检索可缓存资源。

通常，在实践中，始终使用PUT 进行UPDATE操作。

始终POST用于CREATE操作。

## PUT与POST：一个例子

假设我们正在设计网络应用程序。让我们列出几个URI及其目的，以便更好地了解何时使用`POST`以及何时使用`PUT`操作。

GET /device-management/devices ：获取所有设备  
**POST** /device-management/devices：创建新设备

GET /device-management/devices/{id} ：获取设备信息由“id”  
**PUT** /device-management/devices/{id} ：更新由“id”标识的设备信息  
DELETE /device-management/devices/{id} ：通过“id”删除设备

对其他资源也遵循类似的URI设计实践。