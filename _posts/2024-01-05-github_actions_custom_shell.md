---
title: GitHub actionsでインラインPythonスクリプトを直接実行する
tags: tech 開発
layout: post
---

# GitHub actionsでワークフローに書かれたpythonスクリプトを直接実行できます
GitHub actionsでCIを構築していた時に、カレントディレクトリにあるPythonファイルに書かれた定数を読み込んでくる必要があったんですが、わざわざそのためにpythonファイル作るのも馬鹿だなあと思って調べていたら、インラインでPythonスクリプトを実行する方法があったのでメモっておきます。

# まずサンプルコード
ステップのみ記載します。

```
      - name: output version
        id: output_version
        shell: python
        run: |
          import os, sys
          sys.path.append(os.getcwd())
          import constants
          with open(os.environ["GITHUB_OUTPUT"], mode = "a") as f:
            f.write("version="+constants.APP_VERSION)
```

# shellを指定すればよい
ステップ内のshellパラメーターで、runで実行されるコマンドのシェルを指定できます。具体的には、runで指定されたコマンドがファイルに保存され、シェルの第一引数に渡されます。

行ってしまえば何でも指定できるわけで、未検証ですがperlなども動くって書いてありました。好きな言語のインラインスクリプトを書くことができるはずです。

Pythonのコードでは、まずカレントディレクトリをパッケージの検索パスに追加しています。こうしないと、リポジトリ内のモジュールがimportできません。

```
          import os, sys
          sys.path.append(os.getcwd())
```

その後、リポジトリ内にあるconstantsというモジュールのAPP_VERSIONという定数をステップの出力に出力しています。

# pipでインストールしたモジュールを使う
もちろん、事前にsetup-pythonを実行しておけば、任意のバージョンのPythonとpipでインストールしたモジュールが使えます。

```
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          architecture: x86
          python-version: 3.8
          cache: pip

      - name: Install requirements
        run: |
          python -m pip install requests

      - name: send requests by python
        shell: python
        run: |
            import requests
            requests.get("https://hogehoge.com")
            ...
```

# 最後に
GitHub actions、今回はじめて使いましたが、かなり洗練されているなあと思いました。ここまで設定が書きやすくて自由度が高いCIプラットフォームが無料で使えるとは、良い時代になった物です。
