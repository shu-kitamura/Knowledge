# ファイルをReadするときに指定の範囲を読む方法

## 概要

ファイルから指定の範囲のみを読み込むためにした調査を記載する

### 先頭から指定したバイト数だけread

先頭から5バイトをreadする
```python
file = open("test.txt")
txt = file.read(5)
file.close()
```

### 指定したバイトから指定したバイトまでread

3バイト目から6バイト目までread
```python
file = open("test.txt")
file.seek(2)
txt = file.read(4)
file.close()
```
