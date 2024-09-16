https://stackoverflow.com/questions/71882419/fastapi-how-to-get-the-response-body-in-middleware


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