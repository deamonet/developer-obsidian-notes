The main method is not called when deploying the application to a non-embedded application server. The simplest way to start a thread is to do it from the beans constructor. Also a good idea to clean up the thread when the context is closed, for example:

```java
@Component
class EventSubscriber implements DisposableBean, Runnable {

    private Thread thread;
    private volatile boolean someCondition;

    EventSubscriber(){
        this.thread = new Thread(this);
        this.thread.start();
    }

    @Override
    public void run(){
        while(someCondition){
            doStuff();
        }
    }

    @Override
    public void destroy(){
        someCondition = false;
    }

}
```

[Share](https://stackoverflow.com/a/39738524 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/39738524/edit)

Follow

[edited May 9, 2018 at 23:42](https://stackoverflow.com/posts/39738524/revisions "show all edits to this post")