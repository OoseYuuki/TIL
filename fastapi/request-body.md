# Request
クライアント（ブラウザなど）からAPIにデータを送信する必要があるとき、データを**リクエストボディ（request body）**として送る。<br />
**リクエスト**ボディはクライアントによってAPIへ送られる。**レスポンス**ボディはAPIがクライアントに送るデータのことを指す。<br />
`Pydantic`による型定義を行い、リクエスボディのModelを作成するのが一般的？<br />



## `Pydantic`の`BaseModel`をインポート
以下実装例。<br />

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    return item

```

## データモデルの作成
`BaseModel`を継承したクラスとしてデータモデルを宣言する。<br />

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    return item

```

クエリパラメータの宣言と同様、モデル属性がデフォルト値を持つ時、必須な属性ではなくなる。それ以外は必須になる。<br />
オプショナルな属性にしたい場合は`None`を使用する。<br />


例えば、上記モデルは以下のようなJSON「`オブジェクト`」（もしくはPythonの`dict`）を宣言している。<br />

```JSON

{
    "name": "Foo",
    "description": "An optional description",
    "price": 45.2,
    "tax": 3.5
}

```
`description`と`tax`はオプショナル（デフォルト値は`None`）なので、以下のJSON「`オブジェクト`」も有効。<br />

```JSON
{
    "name": "Foo",
    "price": 45.2
}
```

## パラメータとして宣言
パスオペレーションに加えるために、パスパラメータやクエリパラメータと同様に宣言。<br />

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None


app = FastAPI()


@app.post("/items/")
async def create_item(item: Item):
    return item

```

作成したモデル`Item`で型を宣言する。<br />


## まとめ
Pythonの型宣言で、FastAPIは以下のことを行う。<br />


- リクエストボディをJSONとして読み取る。
- 適切な型に変換する。（必要な場合）
- データを検証する。
    - データが無効な場合、明確なエラーが返され、どこが不正なデータだったかを示す。
- 受け取ったデータをパラメータ`item`に変換する。
    - 関数内で`Item`型であると宣言したので、全ての属性とその型に対するエディタサポート（補完など）を全て使用できる。
- モデルのJSONスキーマ定義を生成し、好きな場所で使用することができる。
- これらのスキーマは、生成されたOpenAPIスキーマの一部となり、自動ドキュメントのSwaggerUIに使用される。


## SwaggerUI
以下のようにSchemasの内容揉みれるようになっている。<br />

[![Image from Gyazo](https://i.gyazo.com/ab00fccf8dd9d6fa9740957b81e1196d.png)](https://gyazo.com/ab00fccf8dd9d6fa9740957b81e1196d)<br />


それらが使われるパスオペレーションのそれぞれのAPIドキュメントにも表示される。<br />

[![Image from Gyazo](https://i.gyazo.com/ddadcee466f66924178ddfb6519e96b4.png)](https://gyazo.com/ddadcee466f66924178ddfb6519e96b4)<br />


## リクエストボディ + パスパラメータ
パスパラメータとリクエストボディを同時に宣言可能。<br />

FastAPIはパスパラメータである関数パラメータは**パスから受け取り**、Pydanticモデルによって宣言された関数パラメータは**リクエストボディから受け取る**ということを認識する。<br />

```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None


app = FastAPI()


@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item):
    return {"item_id": item_id, **item.dict()}

```

## リクエストボディ + パスパラメータ + クエリパラメータ
ボディとパスとクエリについても同時に宣言できる。<br />
FastAPIはそれぞれを認識し、適切な場所からデータを取得する。<br />


```python
from typing import Union

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None


app = FastAPI()


@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item, q: Union[str, None] = None):
    result = {"item_id": item_id, **item.dict()}
    if q:
        result.update({"q": q})
    return result

```

関数パラメータは以下のように認識される。<br />

- パラメータが**パス**で宣言されている場合は、優先的にパスパラメータとして扱われる。
- パラメータが**単数型**（`int`、`float`、`str`、`bool`など）の場合はクエリパラメータとして解釈される。
- パラメータが**Pydanticモデル型**で宣言された場合、リクエストボディとして解釈される。　


## 参考文献
[公式ドキュメント](https://fastapi.tiangolo.com/ja/tutorial/body/)
