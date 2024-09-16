What's the correct way of handling a websocket error besides logging it?

Regarding `onError()`, the Endpoint [documentation](http://docs.oracle.com/javaee/7/api/javax/websocket/Endpoint.html#onError%28javax.websocket.Session,%20java.lang.Throwable%29) states that:

> Developers may implement this method when the web socket session creates some kind of error that is not modeled in the web socket protocol. This may for example be a notification that an incoming message is too big to handle, or that the incoming message could not be encoded.
> 
> There are a number of categories of exception that this method is (currently) defined to handle:
> 
> - connection problems, for example, a socket failure that occurs before the web socket connection can be formally closed. These are modeled as SessionExceptions
>     
> - runtime errors thrown by developer created message handlers calls.
>     
> - conversion errors encoding incoming messages before any message handler has been called. These are modeled as DecodeExceptions
>     

Are all of these types of exceptions fatal, causing the websocket to close?

Should the `onError()` method close the websocket (call `Session.close()`) if an error occurs?

So far, I assumed that it's my responsibility to cleanly close the session, informing the client about the close reason. This is why my `onError()` tried invoking `session.close()` if `session.isOpen()` returned true, but this caused tomcat (8.0.15) to throw a `NullPointerException`:

```java
...
Caused by: java.lang.NullPointerException
    at org.apache.tomcat.websocket.server.WsRemoteEndpointImplServer.onWritePossible(WsRemoteEndpointImplServer.java:96)
    at org.apache.tomcat.websocket.server.WsRemoteEndpointImplServer.doWrite(WsRemoteEndpointImplServer.java:81)
    at org.apache.tomcat.websocket.WsRemoteEndpointImplBase.writeMessagePart(WsRemoteEndpointImplBase.java:444)
    at org.apache.tomcat.websocket.WsRemoteEndpointImplBase.startMessage(WsRemoteEndpointImplBase.java:335)
    at org.apache.tomcat.websocket.WsRemoteEndpointImplBase.startMessageBlock(WsRemoteEndpointImplBase.java:264)
    at org.apache.tomcat.websocket.WsSession.sendCloseMessage(WsSession.java:536)
    at org.apache.tomcat.websocket.WsSession.doClose(WsSession.java:464)
    at org.apache.tomcat.websocket.WsSession.close(WsSession.java:441)
    at my.package.MyEndpoint.onWebSocketError(MyEndpoint.java:229)
    ... 18 more
```

Is this a tomcat bug, a misunderstanding on my part, or both?

**Edit:** It seems that the Java EE websocket example [dukeeetf2](https://java.net/projects/javaeetutorial/sources/svn/content/trunk/examples/web/websocket/dukeetf2/src/main/java/javaeetutorial/web/dukeetf2/ETFEndpoint.java?rev=1783) assumes that errors are fatal; and that there's no need to close the session. The errors are logged, and the session is removed:

```java
@OnError
public void error(Session session, Throwable t) {
    /* Remove this connection from the queue */
    queue.remove(session);
    logger.log(Level.INFO, t.toString());
    logger.log(Level.INFO, "Connection error.");
}
```

- [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'")
- [tomcat](https://stackoverflow.com/questions/tagged/tomcat "show questions tagged 'tomcat'")
- [websocket](https://stackoverflow.com/questions/tagged/websocket)
- [onerror](https://stackoverflow.com/questions/tagged/onerror "show questions tagged 'onerror'")

[Share](https://stackoverflow.com/q/27731490 "Short permalink to this question")

[Improve this question](https://stackoverflow.com/posts/27731490/edit)

Follow

[edited Oct 16, 2020 at 16:55](https://stackoverflow.com/posts/27731490/revisions "show all edits to this post")

asked Jan 1, 2015 at 14:08

[

![Malt's user avatar](https://i.stack.imgur.com/cocaf.jpg?s=64&g=1)

](https://stackoverflow.com/users/3199595/malt)

[Malt](https://stackoverflow.com/users/3199595/malt)

29.5k99 gold badges6868 silver badges110110 bronze badges

[Add a comment](https://stackoverflow.com/questions/27731490/how-to-handle-websocket-onerror# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 2 Answers

Sorted by:

8

[](https://stackoverflow.com/posts/27738683/timeline)

`@OnError` method invocation does not mean that Session will be closed; You can do whatever you want, it depends in the contract specified by your application.

stacktrace from tomcat implementation seems like a bug.

ad [dukeeetf2](https://java.net/projects/javaeetutorial/sources/svn/content/trunk/examples/web/websocket/dukeetf2/src/main/java/javaeetutorial/web/dukeetf2/ETFEndpoint.java?rev=1783) sample - seems like this code contains other assumptions - Endpoints does not throw an exception, so everything caught here is from underlying WebSocket framework implementation. That does not really mean that there is an "Connection Error"; I would maybe do close right away (if this is how I wan't my application to handle errors); this implementation could result in opened connections without any messages.

[Share](https://stackoverflow.com/a/27738683 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/27738683/edit)

Follow

answered Jan 2, 2015 at 7:09