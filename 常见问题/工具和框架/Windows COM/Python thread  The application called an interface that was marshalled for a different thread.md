

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 5 years, 11 months ago

Modified [5 years, 11 months ago](https://stackoverflow.com/questions/41192015/python-thread-the-application-called-an-interface-that-was-marshalled-for-a-dif?lastactivity "2016-12-19 13:49:35Z")

Viewed 3k times

1

[](https://stackoverflow.com/posts/41192015/timeline)

The following code works without errors:

```python
 self.sim.create_pnl_results(gui_values, dfs)
```

This code gives me an error:

```python
thread = Thread(target=self.sim.create_pnl_results, args=(gui_values, dfs))
thread.start()

in _ApplyTypes_ self._oleobj_.InvokeTypes(dispid, 0, wFlags, retType, argTypes, *args),
com_error: (-2147417842, 'The application called an interface that was marshalled for a different thread.', None, None)
```

It seems that the application calls a com object somewhere further down the line and because I would like to put a gui (QT) on top I need to make it a separate thread. How can I avoid the above error message? I have read that it is connected to a windows problem. Any suggestions how to resolve it in python would be appreciated. I have no control on the com objects that are called from the application and I don't see why it would make a difference if I call them from the main thread of a new thread.

additional information about how the com object is called:

```python
def process_ts(*args):
    ts_id, i, dfn , ts_queue = args
    pythoncom.CoInitialize()
    ts = win32com.client.Dispatch(pythoncom.CoGetInterfaceAndReleaseStream(ts_id, pythoncom.IID_IDispatch))
    ts_queue.put( tuple([i , ACRF._com_to_ts(ts, i, dfn)]) )
    pythoncom.CoUninitialize ()
```

-   [python](https://stackoverflow.com/questions/tagged/python "show questions tagged 'python'")
-   [multithreading](https://stackoverflow.com/questions/tagged/multithreading "show questions tagged 'multithreading'")
-   [win32com](https://stackoverflow.com/questions/tagged/win32com)

[Share](https://stackoverflow.com/q/41192015 "Short permalink to this question")

[Improve this question](https://stackoverflow.com/posts/41192015/edit)

Follow

[edited Dec 17, 2016 at 16:45](https://stackoverflow.com/posts/41192015/revisions "show all edits to this post")

asked Dec 16, 2016 at 20:29

[

![Nickpick's user avatar](media/Nickpick's_user_avatar.jpg)

](https://stackoverflow.com/users/5181181/nickpick)

[Nickpick](https://stackoverflow.com/users/5181181/nickpick)

5,8401515 gold badges6060 silver badges114114 bronze badges

[Add a comment](https://stackoverflow.com/questions/41192015/python-thread-the-application-called-an-interface-that-was-marshalled-for-a-dif# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 1 Answer

Sorted by:

                                              Highest score (default)                                                                   Trending (recent votes count more)                                                                   Date modified (newest first)                                                                   Date created (oldest first)                              

3

[](https://stackoverflow.com/posts/41224042/timeline)

Many (if not most) GUI applications run into problems if you try to update/redraw the UI from another thread. Just imagine what would happen if you had one thread trying to write "blue" 100x to a file, and another thread trying to write "red" 100x. You'll see something like:

```python
redblueredblruede...
```

(granted with Python you have the GIL so you might not get precisely that same effect, but using multiprocessing you could probably make it happen).

The exact same thing would happen to your graphical application if one thread is trying to turn a region blue, while the other is trying to turn it red. Because that's undesirable, there are guards put in place to stop it from happening by throwing exceptions like the one you encountered.

What you have happening here is something like this:

```python
----(UI Thread)-,----------------->
                 \
                  `---(new thread)-----(affect the UI)-X kaboom!
```

What you _want_ to have happen is something like this:

```python
----(UI Thread)-,--------------------------------------,---(affect the UI)-->
                 \                                    /
                  `---(new thread)-----(pass result)-`
```

The exact mechanics of it will change from framework to framework. In .NET you've got an `if (thing.InvokeRequired){ thing.BeginInvoke(data); }`

On Windows, COM objects are protected the same sort of way. If you create an COM object on a thread it doesn't like you trying to access it on a different thread. So if you're creating it on a different thread and you still need to interact with it you're going to have to communicate across your threads.

If your code isn't blocking, or it doesn't take very long to run (i.e. < 250ms) then running the calls on the main thread should be just fine. Typically UI frameworks allow you to register a callback to be executed after `N` amount of time, and then it will just be executed on the main thread whenever it can be.