# Responseに関するメモ　

## Responseについて
リクエストとは逆で、レスポンスとはAPIがクライアントに送るデータのことを指す。<br />
FastAPIの`Response`クラスは、HTTPレスポンスを作成するためのユーティリティ。<br />
通常、FastAPIアプリケーション内でHTTPレスポンスをカスタマイズする必要がある場合に使用される。<br />

以下、`Response`を使用したFastAPIアプリケーションの実装例。<br />


```python
from fastapi import FastAPI, Response

app = FastAPI()

@app.get("/custom_response/")
def custom_response():
    content = "This is a custom response."
    
    # Responseオブジェクトを作成し、HTTPステータスコードとレスポンスボディを設定 response = Response(content=content, status_code=200)
    
    # レスポンスヘッダーを設定
    response.headers["X-Custom-Header"] = "Header Value"
    
    return response

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1", port=8000)

```

`custom_response`エンドポイントはHTTPステータスコード200とカスタムのレスポンスボディをもつレスポンスを返す。<br />
`Response`クラスを使用することで、HTTPステータスコード、ヘッダー、レスポンスボディなど、レスポンスのさまざまな側面をカスタマイズできる。<br />


## 参考文献
[公式ドキュメント](https://fastapi.tiangolo.com/ja/advanced/response-directly/)<br />

