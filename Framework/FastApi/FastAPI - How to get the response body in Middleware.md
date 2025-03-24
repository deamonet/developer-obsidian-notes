Is there any way to get the response content in a middleware? The following code is a copy from [here](https://fastapi.tiangolo.com/tutorial/middleware/#other-middlewares).

```python
@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()

    response = await call_next(request)

    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

- [python](https://stackoverflow.com/questions/tagged/python "show questions tagged 'python'")
- [response](https://stackoverflow.com/questions/tagged/response "show questions tagged 'response'")
- [middleware](https://stackoverflow.com/questions/tagged/middleware)
- [fastapi](https://stackoverflow.com/questions/tagged/fastapi "show questions tagged 'fastapi'")

[Share](https://stackoverflow.com/q/71882419 "Short permalink to this question")

[Improve this question](https://stackoverflow.com/posts/71882419/edit)

Follow

[edited Apr 15, 2022 at 10:55](https://stackoverflow.com/posts/71882419/revisions "show all edits to this post")

[

![Chris's user avatar](https://www.gravatar.com/avatar/e745a24a67ba3366c2f6acd97072c532?s=64&d=identicon&r=PG)

](https://stackoverflow.com/users/17865804/chris)

[Chris](https://stackoverflow.com/users/17865804/chris)

24.5k77 gold badges6868 silver badges128128 bronze badges

asked Apr 15, 2022 at 9:38

[

![nickchen's user avatar](https://lh3.googleusercontent.com/a-/AOh14GgEv8ClCAoDJ_boD2D2vG_Wq0I742o2ZhtkI2cIDA=k-s64)

](https://stackoverflow.com/users/18339074/nickchen)

[nickchen](https://stackoverflow.com/users/18339074/nickchen)

12311 gold badge11 silver badge66 bronze badges

[Add a comment](https://stackoverflow.com/questions/71882419/fastapi-how-to-get-the-response-body-in-middleware# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 1 Answer

Sorted by:

29

[](https://stackoverflow.com/posts/71883126/timeline)

The `response` body is an iterator, which once it has been iterated through, it cannot be re-iterated again. Thus, you either have to save all the iterated data to a `list` (or `bytes` variable) and use that to return a custom [`Response`](https://fastapi.tiangolo.com/advanced/custom-response/#response), or initiate the iterator again. The options below demonstrate both approaches. In case you would like to get the `request` body inside the `middleware` as well, please have a look at [**this answer**](https://stackoverflow.com/a/73464007/17865804).

## Option 1

Save the data to a `list` and use [`iterate_in_threadpool`](https://github.com/encode/starlette/blob/88e9fc1411f6bd79131afa4a7d2f4dc576c8bf04/starlette/concurrency.py#L58) to initiate the iterator again, as described [here](https://github.com/encode/starlette/issues/874#issuecomment-1027743996) - which is what [`StreamingResponse`](https://fastapi.tiangolo.com/advanced/custom-response/#streamingresponse) uses, as shown [here](https://github.com/encode/starlette/blob/master/starlette/responses.py#L223).

```python
from starlette.concurrency import iterate_in_threadpool

@app.middleware("http")
async def some_middleware(request: Request, call_next):
    response = await call_next(request)
    response_body = [chunk async for chunk in response.body_iterator]
    response.body_iterator = iterate_in_threadpool(iter(response_body))
    print(f"response_body={response_body[0].decode()}")
    return response
```

**Note 1:** If your code uses `StreamingResponse`, `response_body[0]` would return only the first `chunk` of the `response`. To get the entire `response` body, you should join that list of bytes (chunks), as shown below (`.decode()` returns a string representation of the `bytes` object):

```python
print(f"response_body={(b''.join(response_body)).decode()}")
```

**Note 2:** If you have a `StreamingResponse` streaming a body that wouldn't fit into your server's RAM (for example, a response of 30GB), you may run into memory errors when iterating over the `response.body_iterator` (this applies to both options listed in this answer), **unless** you loop through `response.body_iterator` (as shown in Option 2), but instead of storing the chunks in an in-memory variable, you store it somewhere on the disk. However, you would then need to retrieve the entire response data from that disk location and load it into RAM, in order to send it back to the client (which could extend the delay in responding to the client even more)—in that case, you could load the contents into RAM in chunks and use `StreamingResponse`, similar to what has been demonstrated [here](https://stackoverflow.com/a/73843234/17865804), [here](https://stackoverflow.com/a/73672334/17865804), as well as [here](https://stackoverflow.com/a/73241648/17865804), [here](https://stackoverflow.com/a/74239367/17865804) and [here](https://stackoverflow.com/a/73770074/17865804) (in Option 1, you can just pass your iterator/generator function to `iterate_in_threadpool`). However, I would not suggest following that approach, but instead have such endpoints returning large streaming responses excluded from the middleware, as described in [this answer](https://stackoverflow.com/a/73464007/17865804).

## Option 2

The below demosntrates another approach, where the response body is stored in a `bytes` object (instead of a list, as shown above), and is used to return a custom [`Response`](https://fastapi.tiangolo.com/advanced/custom-response/#response) directly (along with the `status_code`, `headers` and `media_type` of the original response).

```python
@app.middleware("http")
async def some_middleware(request: Request, call_next):
    response = await call_next(request)
    response_body = b""
    async for chunk in response.body_iterator:
        response_body += chunk
    print(f"response_body={response_body.decode()}")
    return Response(content=response_body, status_code=response.status_code, 
        headers=dict(response.headers), media_type=response.media_type)
```

[Share](https://stackoverflow.com/a/71883126 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/71883126/edit)

Follow

[edited Jan 20, 2023 at 12:40](https://stackoverflow.com/posts/71883126/revisions "show all edits to this post")

answered Apr 15, 2022 at 10:53