# fastapi-cache

![pypi](https://img.shields.io/pypi/v/fastapi-cache2.svg?style=flat)
![license](https://img.shields.io/github/license/long2ice/fastapi-cache)
![workflows](https://github.com/long2ice/fastapi-cache/workflows/pypi/badge.svg)
![workflows](https://github.com/long2ice/fastapi-cache/workflows/ci/badge.svg)

## Introduction

`fastapi-cache` is a tool to cache fastapi response and function result, with backends support `redis` and `memcache`.

## Features

- Support `redis` and `memcache` and `in-memory` backends.
- Easily integration with `fastapi`.
- Support http cache like `ETag` and `Cache-Control`.

## Requirements

- `asyncio` environment.
- `redis` if use `RedisBackend`.
- `memcache` if use `MemcacheBackend`.

## Install

```shell
> pip install fastapi-cache2
```

or

```shell
> pip install fastapi-cache2[redis]
```

or

```shell
> pip install fastapi-cache2[memcache]
```

## Usage

### Quick Start

```python
import aioredis
from fastapi import FastAPI
from starlette.requests import Request
from starlette.responses import Response

from fastapi_cache import FastAPICache
from fastapi_cache.backends.redis import RedisBackend
from fastapi_cache.decorator import cache

app = FastAPI()


@cache()
async def get_cache():
    return 1


@app.get("/")
@cache(expire=60)
async def index(request: Request, response: Response):
    return dict(hello="world")


@app.on_event("startup")
async def startup():
    redis = await aioredis.create_redis_pool("redis://localhost", encoding="utf8")
    FastAPICache.init(RedisBackend(redis), prefix="fastapi-cache")

```

### Use `cache` decorator

If you want cache `fastapi` response transparently, you can use `cache` as decorator between router decorator and view function and must pass `request` as param of view function.

And if you want use `ETag` and `Cache-Control` features, you must pass `response` param also.

You can also use `cache` as decorator like other cache tools to cache common function result.

### Custom coder

By default use `JsonCoder`, you can write custom coder to encode and decode cache result, just need inherit `fastapi_cache.coder.Coder`.

```python
@app.get("/")
@cache(expire=60,coder=JsonCoder)
async def index(request: Request, response: Response):
    return dict(hello="world")
```

### Custom key builder

```python
def my_key_builder(
    func,
    namespace: Optional[str] = "",
    request: Request = None,
    response: Response = None,
    *args,
    **kwargs,
):
    prefix = FastAPICache.get_prefix()
    cache_key = f"{prefix}:{namespace}:{func.__module__}:{func.__name__}:{args}:{kwargs}"
    return cache_key

@app.get("/")
@cache(expire=60,coder=JsonCoder,key_builder=my_key_builder)
async def index(request: Request, response: Response):
    return dict(hello="world")
```

### InMemoryBackend

`InMemoryBackend` only support in single node instead of distributed environment.

## License

This project is licensed under the [Apache-2.0](https://github.com/long2ice/fastapi-cache/blob/master/LICENSE) License.