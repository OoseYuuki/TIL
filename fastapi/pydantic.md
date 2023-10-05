# Pydantic
## 概要
Pythonの型アノテーションを利用して実行時における型ヒントを提供したり、データのバリデーション時のエラー設定を提供してくれるライブラリ。<br />

### モデル定義

以下のようにBaseModelを継承して定義する。<br />

```python

from datetime import datetime
from typing import List
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str   # (変数):(型)として、型を宣言する
    friendIds: List[int] = []  # "=" を利用してデフォルト値を定義することもできます
    created_at: datetime


external_data={
    'id': '1',
    'name' :'太郎さん',
    'created_at': '2019-11-03 03:34',
    'friendIds': [1,3]
}
user = User(**external_data)

print(user.id)

#> 1 # 入力された値が string 型でも、Int型に自動変換してくれます

print(repr(user.created_at))

#> datetime.datetime(2019, 11, 3, 3, 34)

print(user.friendIds)

#> [1,3]

print(user.dict())

"""
{
   'id': 1,
   'name': '太郎さん', 
   'created_at': datetime.datetime(2019, 11, 3, 3, 34),
   'friendIds': [1, 3]
}

"""
```
## `Field`について
データモデルを定義する際に使用される、より詳細なフィールド構成オプションを提供するためのユーティリティ関数。<br />
`Field`を使用することで、フィールドに関する様々な設定やバリデーションルールを指定することが可能。<br />

1. フィールドのデフォルト値を指定できる。(default)
2. フィールドに対するバリデーションルールを設定できる。（最小値・最大値・正規表現など）
3. フィールドの説明やドキュメンテーションを提供できる。
4. フィールドのエイリアスを指定できる。
5. フィールドの名前を変更できる。


以下実装例。<br />

```python

from pydantic import BaseModel, Field

class Person(BaseModel):
    name: str = Field(..., description="The person's name")
    age: int = Field(..., ge=0, le=150, description="The person's age")

# デフォルト値、バリデーションルール、説明を含むフィールドを持つPersonクラスを定義

```

`name`は`The person's name`という説明を提供し、`age`では、最小値(`ge`)0、最大値（`le`）150とする制限を提供できる。<br />


## `Optional`による任意値の設定
必須項目としてではなく、任意項目として設定したい場合に`Optional`を使用する。<br />
以下のように実装すれば`age`が省略可能なフィールドとして定義することができる。<br />

```python

from pydantic import BaseModel
from typing import Optional

class Person(BaseModel):
    name: str
    age: Optional[int]  # ageが省略可能なフィールドとして定義

# サンプルデータを作成
person1_data = {"name": "Alice", "age": 30}
person2_data = {"name": "Bob"}

# Personクラスにデータをパース
person1 = Person(**person1_data)
person2 = Person(**person2_data)

# データの出力
print(person1)  # 出力: Person name='Alice' age=30
print(person2)  # 出力: Person name='Bob' age=None

```


## 参考資料
[]()
