# SQLAlchemyに関するメモ

## データフェッチの書き方
### SQLAlchemy2.0以前の記述方法

```python
session.query(User).all()
```

### SQLAlchemy2.0以降の記述方法
```python
sesson.execute(select(User)).scalars().all()

or

session.scalars(select(User)).all()
```
案件によって異なるがSQLAlchemyのバージョンに沿った書き方をした方が良さげ。<br />


### `scalars()`と`scalar()`の違い
`scalars()`はクエリから**複数のスカラー値**を取得し、`scalar()`は**単一のスカラー値**を取得するための関数となる。<br />
#### `scalars()`の実装例
```python
from fastapi import FastAPI
from sqlalchemy.orm import Session
from sqlalchemy import select
from app.models import User
from app.database import get_session

app = FastAPI()

@app.get("/get_user_count")
def get_user_count():
    with get_session() as session:
        query = select(User).count()
        user_count = session.execute(query).scalars().first()
        return {"user_count": user_count}

```

#### `scalar()`の使用例

`scalars()`と違い、複数のスカラー値ではなく、クエリから最初のスカラー値を直接取得する。<br />

```python
from fastapi import FastAPI
from sqlalchemy.orm import Session
from sqlalchemy import select
from app.models import User
from app.database import get_session

app = FastAPI()

@app.get("/get_first_user_name")
def get_first_user_name():
    with get_session() as session:
        query = select(User).first()
        user_name = session.execute(query).scalar()
        return {"first_user_name": user_name}

```


### 表結合（`outerjoin`や`innerjoin`について）

#### `innerjoin()`(内部結合)

内部結合は、2つのテーブルから一致する行のみを選択する結合タイプ。<br />
つまり、結合条件を満たす行のみが結果に含まれる。<br />
内部結合を使用すると、一致するデータのみが結果に含まれ、一致しないデータは排除される。<br />

以下、2つのテーブルを内部結合する実装例。<br />

```python
from sqlalchemy.orm import Session
from app.models import User, Order

def get_user_orders(session: Session):
    # 内部結合を使用してUserとOrderテーブルを結合
    query = session.query(User, Order).join(Order, User.id == Order.user_id)
    results = query.all()
    return results

```

#### `outerjoin()`（外部結合）

外部結合は、2つのテーブルから一致する業だけでなく、一致しない行も含む結合タイプ。<br />
一方のテーブルにデータがない場合も、結合条件を満たす行が含まれ、結合条件を満たさない場合は`NULL`値が含まれる。<br />

以下、2つのテーブルを外部結合する実装例。<br />


```python
from sqlalchemy.orm import Session
from app.models import User, Order

def get_user_orders_with_outer_join(session: Session):
    # 外部結合を使用してUserとOrderテーブルを結合
    query = session.query(User, Order).outerjoin(Order, User.id == Order.user_id)
    results = query.all()
    return results


```

まとめると、内部結合は一致するデータのみを取得し、外部結合は一致しないデータも取得する結合タイプ。<br />


## 参考資料
- [[Python] SQLAlchemyを頑張って高速化](https://qiita.com/yukiB/items/d6a70da802cb5731dc01)
- [公式ドキュメント](https://www.sqlalchemy.org/)
- [2.0 Migration - ORM Usage](https://docs.sqlalchemy.org/en/20/changelog/migration_20.html#migration-20-query-usage)
