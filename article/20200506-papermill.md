# 2020/05/06 - Papermill で実行環境を記録して評価実験の再現性を上げる

機械学習で学習・評価実験を走らせた時に最も重要なことの一つはログの保存です。
ログを保存しておくことで、評価結果をあとから参照できるだけでなく、評価実験時の実行環境をあとから参照して実験の再現や精度改善に役立てることができます。

今回は、手作業による作業ログの記録の問題をあげて、その問題点を改善する Jupyter Notebook と Papermill を紹介します。

## 手作業による作業ログ

実行環境を実行を記録する最もナイーブな方法は、実行コマンドをその実行ディレクトリに作業ログとして手作業で保存します。

例えば、次のようにコマンドを実行した時は

```sh
$ python train.py --num_epochs 10
```

ログファイルに次のように書いておきます。

```
# 作業ログファイル

学習を実行

$ python train.py --num_epochs 10
```

これには多くの問題があります。すぐに思いつくだけでも

- ログファイルとの差分発生: コマンドライン引数を変更して実行したのに、ログファイルの修正を忘れると実験を再現できなくなる
- 作業環境の記録: 何かの問題で学習スクリプトや設定ファイルを変更した際、その記録を取るのも手作業であり、ミスが発生しやすい

があります。

## Jupyter Notebook と Papermill

このような問題に対処するためには実行ログを記録しておくことができる Jupyter Notebook を利用すると便利です。

Jupyter Notebook は便利なのですが、デフォルトでは外部からパラメータを指定することが難しいです。
そこで、この Jupyter Notebook をコマンドラインからパラメータ指定して実行しやすくするためのツールが [Papermill](https://github.com/nteract/papermill) です。

Papermill については利用方法は簡単で、READMEを読めばすぐに使い始めることができます。
基本的な使い方は、あらかじめ Jupyter Notebook でコマンドラインから注入するパラメータが書かれたセルを `parameters` タグで指定しておきます。そして、Papermill 実行時にパラメータを与えれば、`parameters` を指定したセルの次に `injected-parameters` が指定された新しいセルを追加してパラメータを上書きして実行してくれます。
もちろん、上書きされた情報も notebook 内に残るのであとから参照することができますので、「ログファイルとの差分発生」のような、作業ログファイルと実行コマンドの差が発生してしまう問題に対処できます。

また、notebook 内で git の情報を出力しておくことで「作業環境の記録」も行うことができます。
例えば、notebook の先頭で次のようなコマンドを打っておけば、現在の実行環境の情報や利用しているライブラリのバージョンを記録することができ、あとから作業環境を確認するのが便利になります。

```sh
! git log -1
! git status
! git diff
! pip list
```

## コードのテスト

以上のように Jupyter Notebook を Papermill で実行することで多くのメリットがあります。
一方で、notebook 内でコードを記述するとテストするのが難しくなるのも事実です。
そのような場合には、テストが必要なコードは別のPythonスクリプトとしてまとめておき、Jupyter notebook 内でテストを実行するようにします。

例えば、 `trainer.py` という学習を行うためのPythonスクリプトを作成しておいて、notebook内でそのテストコード `trainer_test.py` を実行することで実現できます。

```sh
! pytest -v trainer_test.py
```

こうすることで notebook 実行時にテストが通っていたことも後から確認できますね。

## まとめ

Jupyter Notebook と Papermill を使うことで、半年後に実験結果を見ることになったとしても、その実験を再現できる可能性を上げることができます。