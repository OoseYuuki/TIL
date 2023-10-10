# 列挙型（`enum`）の基本的な使い方

## 列挙型の特徴
- 複数の定数（列挙子）を一つの型で統一して管理できる
- 一つの列挙子に複数の表現が定義できる
- 列挙子に共通野処理を定義することができる

定数をまとめて管理できる。ただし、定数に変更があるたびにコードの書き換えが必要になる。<br />

## `enum`モジュールとは
列挙型クラスをサポートする。<br />

### `enum.Enum`を使用した実装例
例えば、血液型を表すクラスがあるとする。<br />
血液型は, B, O, ABの4種類から成り立っているので、一般的なクラスで表現すると以下のようになる。<br />

```python
class BloodType:
    Def __init__(self, type: str):
        self. type = type
```

ただ、上記の例だとA, B, O, AB以外の文字列が入ることを許可してしまう。<br />
なので、`__init__`内で探知する。<br />


```python

class BloodType:
    Def __init__(self, type: str):
        if type not in (‘A’, ‘B’, ‘O’, ‘AB’):
            raise ValueError(‘妥当な血液型ではありません’)
        self. type = type
```

これを列挙型クラスで表現すると以下のようになる。<br />


```python
import enum


class BloodType(enum.Enum):
    A = 'A'
    B = 'B'
    O = 'O'
    AB = 'AB'
```

## `enum`モジュールの基本的な使い方
```python
import enum


class BloodType(enum.Enum):
       A = 'A型'
       B = 'B型'
       O = 'O型'
       AB = 'AB型'
```


```python
>>> import app.example
>>>
>>>
>>> app.example.BloodType.O
<BloodType.O: 'O型'>
>>> app.example.BloodType.O.value
'O型'
>>> app.example.BloodType.O.name
'O'

```


