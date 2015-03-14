
# mozjpeg の cjpeg を Visual Studio でビルドする #
ChainLP で外部起動する jpeg エンコーダーに mozjpeg の cjpeg を使うための，mozjpeg のビルド方法についてメモしておく．

## 用意するもの ##
  * mozjpeg
    * https://github.com/mozilla/mozjpeg
  * Visual Studio (Express で十分)
    * http://www.microsoft.com/ja-jp/dev
  * cmake
    * http://www.cmake.org/
  * nasm (WITH\_SIMD でビルドする場合のみ)
    * http://www.nasm.us/

mozjpegはソースコードを任意のフォルダに展開しておく．また Visual Studio と cmake はインストールし使える状態にしておく．

## ビルドする ##
cmake を使って Visual Studio のソリューションファイルを生成し，それを使って Visual Studio でビルドする，という流れになる．

  * cmake-gui を起動し **Where is the source code:** には mozjpeg のソースコードが展開されているフォルダを，**Where to build the binaries:** には実際にビルドを実行するフォルダを指定する．
    * ビルドを実行するフォルダは，ソースコードが展開されているフォルダとは別にした方がよい．
    * ビルドを実行するフォルダは，あらかじめ作っておく必要はない(後で「作る?」と聞かれる)．
  * Configure ボタンを押下する．
    * ビルドを実行するフォルダが未生成の場合は，ここで「作る?」と聞かれる．
  * ビルドに使用するツール，実行環境を指定するダイアログが出るので自分の環境に合ったモノを指定し Finish ボタンを押下する．
    * わたしの場合は **Visual Studio 12 2013 Win64** と **Use default native compilers** を指定した．
    * ビルド時に指定できる各種パラメータが表示される．
  * **WITH\_SIMD** がデフォルトでチェックされているので **NASM** に nasm の実行ファイルをフルパスで記載する．
    * **WITH\_SIMD** のチェックを外すなら **NASM** はそのままでも良い
  * 再度 Configure ボタンを押下する．
  * 各種パラメータ表示のバックグラウンドが赤くなっていないことを確認し，Generate ボタンを押下する．
    * ビルドを実行するフォルダに各種ファイル・フォルダが生成される．
    * Visual Studio で使用するソリューションファイル **libmozjpeg.sln** もここで生成される．
    * cmake の使用はこれで終了．
  * Visual Studio を起動し，libmozjpeg.sln を読み込ませる．
  * **cjpeg-static** プロジェクトをビルドする．
    * Debug ビルドでなく，Release や MinSize ビルドにする．
    * cjpeg-static.exe ができるので，好きに使う．

# ChainLP での使い方 #
cjpeg とかで -arithmetic が使え ~~ないのだけど，まだ arithmetic coding は一般的ではない?~~ るようになったので，ちょっと考察してみた(要: mozjpeg-3.0 以降)．

v2.1 と v3.0 の出力サイズを比較してみたところ，v3.0 の方がサイズが大きくなる傾向が見られた．なので v2.1 の cjpeg でエンコードし，v3.0 の jpegtran を使って arithmetic 化するということに．

以下のようなバッチファイルを作成し，ClainLP の External 設定で指定すればよいだろう．
```
@echo off
.\cjpeg-static_21.exe -grayscale -optimize -progressive -outfile %1.tmp %1
.\jpegtran-static_30.exe -arithmetic -outfile %2 %1.tmp
del %1.tmp
```