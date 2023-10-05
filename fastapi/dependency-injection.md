# DI（`Dependency Injection`、依存性注入）

## FastAPI標準のDIと、その課題
FastAPIは標準でDIシステムを持っており、まずはそれを使うことが基本である。<br />
[公式ドキュメント](https://fastapi.tiangolo.com/tutorial/dependencies/)の説明が丁寧なので、あとでこれを読む。<br />

以下のような課題があるらしい。<br />

### 課題①：テストごとの別のモックを注入することがしづらい
[Stack Overflow](https://stackoverflow.com/questions/65259085/best-way-to-override-fastapi-dependencies-for-testing-with-a-different-dependenc)に同じ悩みが掲載されている。<br />


エンドポイントの定義例。<br />

```python
def common_parameters() -> dict[str, str]:
    return {"message": "Hello world"}


@app.get("/hoge")
def get_hoge(params: dict[str, str] = Depends(common_parameters)):
    return { "result": params["message"] }

```

以下の実装例は、`common_parameters`を複数種類のモックで差し替えてテストする例である。<br />
コメントしている箇所のような課題が生まれるので、`dependency_overrides`は都度復元すべきだと感じる。<br />


```python

def mock_parameters_1():
    return {"message": "See you"}


def mock_parameters_2():
    return {"message": "Bye bye"}    


client = TestClient(app)

def test_mock_1():
    app.dependency_overrides[common_parameters] = mock_parameters_1
    response = client.get("/hoge")
    assert response.json() == { "result": "See you" }
    

def test_mock_2():
    app.dependency_overrides[common_parameters] = mock_parameters_2
    response = client.get("/hoge")
    assert response.json() == { "result": "Bye bye" }


def test_normal():
    response = client.get("/hoge")
    # このテストでは dependency_overrides を変更していないが、上で破壊されているので失敗する
    assert response.json() == { "result": "Hello world" }

```
...よくわからないので、また今度記事を読み込んでまとめる。<br />

## `Depends`とは
FastAPIにおいて、Dependency Injectionをサポートする関数
- `Depends()`には関数を渡す
- 返り値は関数の実行結果
以下実装例。<br />

```python
# import などを省略

async def common_parameters(q: Optional[str] = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return commons


@app.get("/users/")
async def read_users(commons: dict = Depends(common_parameters)):
    return commons

```

## 参考文献
- [FastAPIでDIする（Dependency Injectorを使う）](https://zenn.dev/shimat/articles/4be773f427c502)
