# エラー処理でstacktraceを表示する

## 概要

try-exceptの例外処理でstacktraceをコンソールに表示する方法を記載する

## 使用環境

  *  windows 11
  *  Python 3.10.10

## 使用ライブラリ

  *  traceback

## 方法

except句で、以下のメソッドを使用することで表示できる

|メソッド名 |処理内容 |
| ---- | ---- |
| print_exc | stacktraceを標準エラーに出力する |

```python
import traceback

try:
    <try句の処理>
except:
    traceback.print_exc()
```
tracebackクラスには、他にもメソッドがあり、色々できそうに見えるが、
stacktraceの表示については上記の方法でできた。

### 使用例

```python
import traceback

def zerodiv():
    return 10/0

try:
    zerodiv()
except ZeroDivisionError:
    traceback.print_exc()
```
出力内容は以下

```
Traceback (most recent call last):
  File "C:\Users\shuse\OneDrive\デスクトップ\python\test_traceback.py", line 7, in <module>
    zerodiv()
  File "C:\Users\shuse\OneDrive\デスクトップ\python\test_traceback.py", line 4, in zerodiv
    return 10/0
ZeroDivisionError: division by zero
```

## 参考  

Pythonドキュメント : https://docs.python.org/ja/3/library/traceback.html