---
title: 想利用 ppocrlabel 做圖像訓練的前置步驟
tags:
  - ppocrlabel
  - paddlepaddle
  - tensorFlowLite
abbrlink: 1718242579
date: 2024-06-26 17:42:08
---


原本公司想要用影像去辨識七段顯示器，剛好網路上看到有可以用的[連結](<https://github.com/renjithsasidharan/seven-segment-ocr>)，於是開始了前途多舛的時間，結果終於好了。不過很多步驟我來不及記起來，但至少這些注意事項已經夠多了！
<!-- more -->

## 開始

### 第一步

首先你去用個 miniConda 用個獨立的環境，以免被相依性搞到掛掉。

### 第二步

執行 `python3 -m pip install paddlepaddle -i https://mirror.baidu.com/pypi/simple`，我就照著步驟安裝

### 第三步

執行 `pip install PPOCRLabel`，就可以先安裝需要的相依性套件。如果跳出錯誤，就照他的安裝，例如：

- 可能會遇到 `gcc: error: /FIPython.h: File o directory non esistente` 的錯誤訊息，網路搜尋後可能是 `lmdb` 的問題，因此我執行：
`conda install -c conda-forge python-lmdb`

### 第四步

我反安裝：`pip uninstall PPOCRLabel`，因為我想要用 whl 安裝的方式。

### 第五步

```cmd
cd ./PPOCRLabel 
反正就是這個資料夾

python setup.py bdist_wheel
pip install dist/看這個資料夾裡出現的那個檔名

```

### 第六步

中間跳出缺少什麼相依，就安裝

### 第七步

這時一直跳出 `ImportError: numpy.core.multiarray failed to import`，但是我重新安裝 numpy 也沒用，這時我只好重新安裝 opencv-python，但是在安裝時看到下面的訊息：

```cmd
ERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
paddleocr 2.7.3 requires opencv-python<=4.6.0.66, but you have opencv-python 4.10.0.84 which is incompatible.
```

看來是安裝太新的版本了，於是我執行 `pip install opencv-python<=4.6.0.66`結果還是沒用。但是看起來還是不行，看起來是 numpy 太新，於是我嘗試：

```python
pip install numpy<2.0.0
```

可以了，終於前進到下一個錯誤了

### 第八步

```cmd
Traceback (most recent call last):
  File "F:\PaddleOCR-release-2.7.1\PaddleOCR-release-2.7.1\PPOCRLabel\PPOCRLabel.py", line 2840, in <module>
    sys.exit(main())
             ^^^^^^
  File "F:\PaddleOCR-release-2.7.1\PaddleOCR-release-2.7.1\PPOCRLabel\PPOCRLabel.py", line 2828, in main
    app, _win = get_main_app(sys.argv)
                ^^^^^^^^^^^^^^^^^^^^^^
  File "F:\PaddleOCR-release-2.7.1\PaddleOCR-release-2.7.1\PPOCRLabel\PPOCRLabel.py", line 2818, in get_main_app
    win = MainWindow(lang=args.lang,
          ^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "F:\PaddleOCR-release-2.7.1\PaddleOCR-release-2.7.1\PPOCRLabel\PPOCRLabel.py", line 104, in __init__
    self.table_ocr = PPStructure(use_pdserving=False,
                     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\tools\miniconda3\envs\lastTry\Lib\site-packages\paddleocr\paddleocr.py", line 762, in __init__
    super().__init__(params)
  File "F:\PaddleOCR-release-2.7.1\PaddleOCR-release-2.7.1\PPOCRLabel\..\ppstructure\predict_system.py", line 82, in __init__
    self.return_word_box = args.return_word_box
                           ^^^^^^^^^^^^^^^^^^^^
AttributeError: 'Namespace' object has no attribute 'return_word_box'
```

然後根據[這裡](<https://github.com/PaddlePaddle/PaddleOCR/issues/11166>)的說明，是 paddleocr 版本太新的緣故，試著安裝 2.6.0 版本，解決了

### 第九步

你可能會遇到 `Microsoft Visual C++ 14.0 is required. Get it with “Microsoft Visual C   Build Tools`，你需要執行：
`conda install libpython m2w64-toolchain -c msys2`。什麼去找 build-tools 的都沒用，[我覺得裡面講的也很有道理，可以去看看](<https://blog.csdn.net/qzzzxiaosheng/article/details/125119006>)。

### 第十步

在此資料夾裡執行 `python PPOCRLabel.py`，遇到問題：

```cmd
Traceback (most recent call last):
  File "<frozen runpy>", line 198, in _run_module_as_main
  File "<frozen runpy>", line 88, in _run_code
  File "C:\tools\miniconda3\envs\lastTry\Scripts\PPOCRLabel.exe\__main__.py", line 4, in <module>
  File "C:\tools\miniconda3\envs\lastTry\Lib\site-packages\PPOCRLabel\PPOCRLabel.py", line 40, in <module>
    from paddleocr import PaddleOCR, PPStructure
  File "F:\PaddleOCR-release-2.7.1\PaddleOCR-release-2.7.1\PPOCRLabel\..\paddleocr.py", line 575, in <module>
    class PaddleOCR(predict_system.TextSystem):
                    ^^^^^^^^^^^^^^
NameError: name 'predict_system' is not defined
```

最後在[這裡](<https://github.com/PaddlePaddle/PaddleOCR/issues/12057>)看到解法，其中底下說：

```text
从2.7.1开始paddleocr.py里面就缺了这句话from tools.infer import predict_system
```

加進去後，就好啦！
