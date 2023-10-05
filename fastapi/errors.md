# エラーハンドリング

## HTTPExceptionを利用するケース
以下実装例。<br />

```python
from fastapi import FastAPI, HTTPException
from fastapi.responses import JSONResponse
from fastapi.responses import PlainTextResponse
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()

items = {"1": "test_item"}

@app.get("/testapi/http_exception/items/{item_id}")
async def get_http_exception(item_id: str):
    # 指定したキーが存在しない場合
    if item_id not in items:
        raise HTTPException(status_code=404, detail="item_not_found")
    return {"item": items[item_id]}

```

エラーが発生するとStatusCode`404`を返す。<br />

## 独自で定義したExceptionを利用するケース
Exceptionを継承し、独自定義したエラーを返却する。<br />
パスパラメータとして指定された値をレスポンスボディに埋め込んだりしたい場合に有効。<br />

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

items = {"1": "test_item"}


class CustomNotFoundException(Exception):
    def __init__(self, item_id: str):
        self.item_id = item_id


@app.exception_handler(CustomNotFoundException)
async def CustomExceptionHandler(request: Request, exception: CustomNotFoundException):
    return JSONResponse(status_code=404, content={"detail": f"item_id_{exception.item_id} is_not_found"})


@app.get("/testapi/custom_exception/items/{item_id}")
async def get_custom_exception(item_id: str):
    # 指定したキーが存在しない場合
    if item_id not in items:
        raise CustomNotFoundException(item_id=item_id)
    return {"item": items[item_id]}

```

## デフォルトExceptionハンドラーをオーバーライドするケース
デフォルトのExceptionハンドラーをオーバーライドさせる。<br />
レスポンスボディをJSONではなく、text形式で返却させたい場合に有効。<br />

```python
from fastapi import FastAPI, HTTPException
from fastapi.responses import PlainTextResponse
from starlette.exceptions import HTTPException as StarletteHTTPException

app = FastAPI()

items = {"1": "test_item"}


@app.exception_handler(StarletteHTTPException)
async def http_exception_handler(request, exc):
    return PlainTextResponse(str(exc.detail), status_code=exc.status_code)


@app.get("/testapi/override_http_exception/items/{item_id}")
async def get_override_http_exception(item_id: str):
    if item_id not in items:
        raise HTTPException(
            status_code=404, detail=f"item_id_{item_id}_is_not_found")
    return {"item": items[item_id]}

```

## 参考文献
[Qiita記事](https://qiita.com/KWS_0901/items/46a2c5b661ee23e61652)<br />


[公式ドキュメント](https://fastapi.tiangolo.com/tutorial/handling-errors/)<br />

