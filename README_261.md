# 目次 #



# 配布物について #
現状の配布物には，以下のものが含まれている．
  * u-boot\_hack.bin / u-boot\_hack\_mark4.bin
  * KoboRoot.tgz
    * jpeg-turbo-1.4.0
      * http://libjpeg-turbo.virtualgl.org/
    * zlib-1.2.8
      * http://www.zlib.net/
    * dosfstools-3.0.27
      * https://github.com/dosfstools/dosfstools
      * http://daniel-baumann.ch/software/dosfstools/
    * freetype-2.5.5
      * http://www.freetype.org/
    * unrarsrc-5.2.3
      * http://www.rarlab.com/
    * libpng-1.6.16
      * http://www.libpng.org/pub/png/libpng.html
    * expat-2.1.0
      * http://expat.sourceforge.net/
    * sqlite-3.8.7.4
      * http://www.sqlite.org/
    * qt-5.2.1
      * http://qt-project.org/
      * https://github.com/kobolabs/
    * libxml2-2.9.2
      * http://xmlsoft.org/
  * KoboRoot\_hack.tgz
    * dropbear-2014.66
      * https://matt.ucc.asn.au/dropbear/dropbear.html
    * openssh-6.3p1
      * http://www.openssh.org/
    * gdb-7.7
      * http://www.gnu.org/software/gdb/
    * i2c-tools-3.1.0
      * http://www.lm-sensors.org/wiki/I2CTools
    * strace-4.8
      * http://sourceforge.net/projects/strace/
    * libuuid-1.0.2
      * http://sourceforge.net/projects/libuuid/
    * parted-2.4
      * http://www.gnu.org/software/parted/
    * busybox-1.22.1
      * http://www.busybox.net/
  * README.wiki (本ファイル)

# 差分の簡単な説明 #
## u-boot\_hack.bin / u-boot\_hack\_mark4.bin ##
この u-boot\_hack.bin を内蔵 SD に書き込むことで，外部 SD 起動の条件が変更され容易に実現可能になる．
|旧条件|外部 SD スロットにカードが装着されいる && ホームキーが押されている && 電源スイッチがスライドされたままである．|
|:--------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------|
|新条件|外部 SD スロットにカードが装着されいる．|

外部 SD 起動は uImage を外部 SD からロードし，かつ root=/dev/mmcblk1p1 で起動するため，カーネルや rootfs の差し替えが簡単にできハックに大変有用であるが，本バイナリは u-boot の書き換え方法を十分に理解できている方々向けのものである(文鎮化しても動じない，等)．

なお，個体依存で外部 SD 起動ができそうにないことがあるようだ(緑 LED 点灯したままフリーズ)．

シリアルをつないで様子を見てみると，外部 SD からのデータリードに DMA を起動させても完了フラグが変化しないっぽい(割り込みではなくて，フラグをポーリングで見張っている)．

microSD を交換(内部 ↔ 外部)しても同じ所で停止するので，u-boot 内の sd ドライバの作りが悪いのではないかな，と．

```
MMC read: dev # 0, block # 1023, count 1 partition # 0 ...
1 blocks read: OK

MMC read: dev # 0, block # 1024, count 1 partition # 0 ...
1 blocks read: OK

**************************
**    Boot from SD      **
**************************

ram p=70000000,size=268435456

MMC read: dev # 1, block # 1023, count 1 partition # 0 ...
```

ふと気がついたことがある．

これまでの開発機は i.MX507 で，こっちでは普通に外部 SD 起動できていた．で，今回こういう現象になるのは i.MX508 だということだ(表示部が壊れた読書機)．確認した個体数が i.MX507 品 2 台，i.MX508 品 1 台だけなのでまだ憶測の域を脱しないのだが，eratta を疑ってみるのがよさそうな気がする．

新たに調達した touch は i.MX508 だったが，こいつだと外部 SD 起動できた(当然 microSDHC は同じモノを使用)．なんじゃこりゃ? 純粋な個体差ってことか?

## jpeg-turbo-1.4.0 (KoboRoot.tgz) ##
http://libjpeg-turbo.virtualgl.org/ で配布されているものを
```
-mthumb -march=armv7-a -mtune=cortex-a8 -mfpu=neon -O3
```
を付加して clang + llvm でコンパイルしただけのものである．

1.3.1 では，CVE-2013-6629/6630 が修正されている．

1.4.0 から jchuff.c で builtin\_clz() を使うようになったため，ルックアップテーブルを使わなくなった．bss 領域が減ってウマー．x86 系ではテーブル参照の方が高速だそうで，ARM 系しか使わないのだそうだ．

### 性能評価結果 ###
使用データには http://ja.wikipedia.org/wiki/レナ_(画像データ) に貼ってある l\_hires.jpg(1084 x 2318, 24bpp) を使用し，伸長処理には djpeg を
```
#!/bin/sh
for i in `seq 1 100`
do
  djpeg $1 > /dev/null
done
```
と 100 回繰り返し実行するシェルスクリプトを用いて time コマンドで計測した．スクリプトファイル，評価データはすべて /tmp 以下に置き SD へのアクセスが行われないよう配慮した．

標準 (たぶん jpeg-turbo-1.2.90)
```
# time ./test.sh ./l_hires.jpg
real    0m 15.81s
user    0m 14.50s
sys     0m 0.69s
```

jpeg-turbo-1.3.0 (linaro gcc 版)
```
# time ./test.sh ./l_hires.jpg
real    0m 15.84s
user    0m 14.51s
sys     0m 0.79s
```

そんなわけで，以前のような劇的な改善が見られるようなことはなくなった．大変喜ばしいことである(いじり甲斐がなくなったけど)．

今回から clang + llvm でコンパイルしたのだが，linaro gcc に比べてわずかに早くなる(cbz のモノクロ jpeg を試行回数 500 回で計測)．特筆すべきはオブジェクトのサイズが結構小さくなるところ．
```
linaro gcc 版
# time ./test.sh P00007.jpg
real    0m 26.41s
user    0m 22.51s
sys     0m 2.90s

clang + llvm 版
# time ./test.sh P00007.jpg
real    0m 25.72s
user    0m 22.66s
sys     0m 2.76s

   text	   data	    bss	    dec	    hex	filename
 218527	   1360	  65552	 285439	  45aff	libjpeg.so.62.1.0 (linaro gcc 版)
 166095	   1348	  65544	 232987	  38e1b	libjpeg.so.62.1.0 (clang + llvm 版)
```

## zlib-1.2.8 (KoboRoot.tgz) ##
ベースにするバージョンを 1.2.8 に変更した以外に，以下の改良を加えている．
  * adler32 の neon 実装を g2cd プロジェクト http://sourceforge.net/projects/g2cd/ から拝借した
  * inffast の neon 実装を https://bugzilla.mozilla.org/show_bug.cgi?id=462796 から拝借した
  * 拝借する上でコンパイルが通る程度に上述の実装や Makefile.in 等あちこち手を入れた
  * inffast の 16バイト未満のコピールーチンを http://sourceware.org/ml/libc-ports/2009-07/msg00003.html のコードを参考に書き換えた
  * zutil.h にある ZSWAP32() の実装を gcc の組み込み関数 `__builtin_bswap32()` に置き換え
以上改良を加えた上で，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -ftree-vectorize
```
を付加してコンパイルした．

`__builtin_bswap32()` の簡単な解説については後述する(unrarsrc の方がわかりやすいコードを出してきたから)．が，圧縮側にしか有効じゃないっぽいから意味ないかも．

### 性能評価結果 ###
気が向いたらやる．

## dosfstools-3.0.27 (KoboRoot.tgz) ##
ベースにするバージョンを 3.0.27 に変更し，
```
-mthumb -march=armv7-a -mtune=cortex-a8 -mfpu=neon -ftree-vectorize -O3
```
を付加して clang + llvm でコンパイルしただけのものである．

/etc/init.d/rcS に dosfsck -a -w /dev/mmcblk0p3 を実行するように書かれているが，ChangeLog を確認すると 3.0.11 あたりに https://bugzilla.redhat.com/show_bug.cgi?id=660154 と不安になるような記載があるため念の為に入れることにした．

なお KoboReader/build/script/dosfstools.sh の記載に倣い，dosfsck と mkdosfs だけを /bin 以下に配置している(~~mkdosfs は recoveryfs に入れないと意味ないかも~~ recoveryfs では mkfs.vfat(busybox) を使っていた(ぉぃぉぃ)．どういう使い分けをしているんだ?)．

3.0.16 への更新の過程で，FAT のボリュームラベルは小文字を使ってはいけないことを知った．そうすると，recoveryfs の /etc/init.d/rcS に書いてある
```
mkfs.vfat -n KOBOeReader /dev/mmcblk0p3 && sync && sync
```
は仕様外の使い方だということに，やれやれ．

ちょっとした都合で開発機の MicroSDHC を SanDisk の SDSDQUA-032G-U46A から Transcend の Class6 品に変えたところ，書籍の追加(sqliteファイルの更新)が我慢できないくらい遅くなった．単純な Class の差とは思えないくらい，である．I/O スケジューラーの設定は cfq になっているので一概には言えないが，やはりランダムアクセス重視じゃないとダメじゃないかと．

dosfstools-3.0.18 から実行ファイル名が変更になったので、(ちょっと悩んだけど)追従するようにした。

dosfstools-3.0.20 から 32GB 以上の FAT パーティションではクラスタサイズが変更(増加)になった。しかし FAT フォーマットするのは recoveryfs からなので、そちらのバイナリ置き換えをしないといけないし、そもそも mmcblk0p`[`012`]` の分だけパーティションサイズが 32GB よりも小さいから意味が無い、残念。32GB でも持て余しているのに 64GB なんてありえないし。

dosfstools-3.0.26 で「頻繁な電源断によって生成された "odd" ファイルを修復する」ということがうたわれている．なんでも....
  * ls -l で見ることができない
  * ls -i では「不正なファイル名」と表示される
  * diropen や dirread では見えるが stat できない
  * ファイルアクセスができず，削除もできない
ファイルが，それにあたるらしい．「修復」といっても，正常な状態に戻るわけじゃないだろう．

## freetype-2.5.5 (KoboRoot.tgz) ##
ベースにするバージョンを 2.5.5 に変更した以外に，以下の改良を加えている．
  * FT\_CONFIG\_OPTION\_SUBPIXEL\_RENDERING の有効化
以上改良を加えた上で，
```
-mthumb -march=armv7-a -mtune=cortex-a8 -mfpu=neon -O3
```
を付加して clang + llvm でコンパイルしたものである．

サブピクセルレンダリングの有効化は http://www.freetype.org/patents.html の Other Patent Issues に記載されているモノである(だからデフォルトのコンフィグレーションでは無効になっている)．

とは言えそれだけのハズなのに，なぜかサイズが半分程度になるようである(clang + llvm でちょっと太ってしまった)．
```
-rwxrwxr-x    1 root     root        433368 Mar 14 20:18 libfreetype.so.6.11.2
-rwxr-xr-x    1 root     root        753016 Jun  8 12:47 libfreetype.so.6.6.2
```
某所では Monotype のレンダラが入っているっポイ，という情報があり，その分が根こそぎサイズ減につながっているのかもしれない．と言っても半分はないだろう，と思うけれども．

2.4.12 から Adobe の cff フォントのラスタライズ技術が取り込まれている(http://blogs.adobe.com/CCJKType/2013/05/adobe-contributes-cff-rasterizer-to-freetype-jp.html )．付属ドキュメントに倣い，デフォルトで有効になるようコードを修正して使ってみたところ，以前に比べてルビが読みやすくなった(2台並べて見比べた)．

また動作上使われないであろう freetype 組み込みの zlib 実装を，外部 shared libaray を使うようにコンフィグレーションした(サイズを小さくするつもりが `-mtree-vectorize` を付けたので相殺された)．

2.5.0 からデフォルトが Adobe のエンジンになったため、ローカル修正をやめた。また png のグリフ埋め込みがサポートされたので、これを有効にした(内蔵フォントをそのまま使うだけなら意味は無いだろうけど)。

2.5.1 から http://mimosa-pudica.net/unix-font.html に記載のパッチを適用をやめてみた．いちおう新旧ライブラリを入れた touch を 2 台ならべてルビの目視比較をしてみたりしたが，特に差は感じられなかった．Adobe エンジンの効果かなと勝手に予想．

2.5.3 で CVE-2014-2240 が修正された．HarfBuzz レンダラを入れてみたいところだが，コンパイルに手間取ってい ~~るので精進中~~ たが，なんとなくコンパイルできるようになった(入れるなら hack 側にだけど)．HarfBuzz はレンダラではなくてレイアウトエンジンである記載を見つけた．ということは，入れても意味なさそうな気がしてきた．もうちょっと素性を調べてみよう．

2.5.4 で CVE-2014-2240 が修正された．

### 性能評価結果 ###
ft2demos に含まれている ftbench で性能評価ができることがわかったので試しにやってみた．

標準 (libfreetype.so.6.6.2, kobo-update-3.5.0 版)
```
# /tmp/ftbench -t 5 -p -s 14 -b a -f 0008 /tmp/arial.ttf
Load                      : 73.191 us/op
```

freetype-2.5.3 (clang + llvm 版)
```
# /tmp/ftbench -t 5 -p -s 14 -b a -f 0008 /tmp/arial.ttf
Load                      : 58.648 us/op
```

初期の頃(いつだったっけ?)に比べて標準配布のライブラリが 2 倍近く速くなっているので，こちらも置き換えの意味が無くなってきた．

## unrarsrc-5.2.3 (KoboRoot.tgz) ##
ベースにするバージョンを 5.20(release) に変更した以外に，以下の改良を加えている．
  * shar256.cpp をコンパイラの最適化がききやすいように代入文をループ実行に書き換え
以上改良を加えた上で，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -ftree-vectorize
```
を付加してコンパイルしたものである．

5.10 からはエンディアン変換の実装を builtin 関数に置き換えてもらった．

## libpng-1.6.16 (KoboRoot.tgz) ##
ベースにするバージョンを 1.6.16 に変更し，さらに
  * png.h にあるエンディアン変換のマクロを gcc の builtin 関数に置き換え
以上改良を加えた上で，
```
-mthumb -march=armv7-a -mtune=cortex-a8 -mfpu=neon -ftree-vecorize
```
を付加してコンパイルしたものである．

1.6.4 は neon enable でコンパイルするとエラーになる．通報したら程なく 1.6.5 が出たがパッケージングミスなのか直ってなくて，さらに 1.6.6 が出た．

1.6.7 からは neon の実装が C で書かれるようになり，thumb 出力できるようになった．

Ubuntu のバージョンを上げたら clang + llvm の具合が悪くなった．なので今回は gcc でコンパイルしているのだが clang + llvm よりも小さくなった．インライン展開が減ったからか?
```
   text   data    bss     dec    hex filename
 178688    980      4  179672  2bdd8 libpng16.so.16.4.0  (linaro gcc 版)
 148010    884      4  148898  245a2 libpng16.so.16.7.0  (clang + llvm 版)
 128486    968      4  129458  1f9b2 libpng16.so.16.13.0 (linaro gcc 版)
```

1.6.8 で CVE-2013-6954 が修正された．

1.6.10 で CVE-2014-0333 が修正された．

1.6.15 でメモリの不正アクセスに関する脆弱性が修正されているとのこと．

## expat-2.1.0 (KoboRoot.tgz) ##
ベースにするバージョンを 2.1.0 に変更し，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -ftree-vectorize
```
を付加してコンパイルしただけのものである．

前バージョンである 2.0.1 から実に 5 年近く経っていて，更新されたことを知らない人が多いんじゃないかと思う．こちらも多数の CVE が報告されているため，入れ替えた方が無難である(Changes ななめ読みしただけでも 5 つほど)．

でも，どこからも参照されていないような感じがしてならない．

## sqlite-3.8.7.4 (KoboRoot.tgz) ##
ベースにするバージョンを 3.8.7.4 に変更(qt-5.2.1 に入っている sqlite は 3.7.17)した以外に，以下の改良を加えている．
  * sqlite3Get4byte()，sqlite3Put4byte()，BYTESWAP32() の実装を gcc の組み込み関数 `__builtin_bswap32()` に置き換え
以上改良を加えた上で，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -ftree-vectorize
```
を付加し，さらに
```
SQLITE_THREADSAFE SQLITE_ENABLE_FTS3 SQLITE_ENABLE_RTREE SQLITE_OMIT_LOAD_EXTENSION SQLITE_DEFAULT_WORKER_THREADS=1
```
を define してコンパイルしたものである．前の 3 つは sqlite3 を普通に作ると付与されるもの，LOAD\_EXTENTIONをOMITするのは qt-5.2.1 に組み込まれるときに付与されるもの，WORKER\_THREADSを1にするのは独自に付与したものである．

実は qt-5.2.1 に組み込まれるときには，さらに `SQLITE_OMIT_COMPLETE` が define されるのだが(src/3rdparty/sqlite.pri)，コマンドラインインターフェースの sqlite3 コマンドをリンクするときに undef でエラーになってしまうので，これは外しておいた(コマンド本体は収めていないけど)．

3.8.4.1 以降，気持ち動作が軽くなったような気がする(例によってプラセボ効果だろうけど)．

3.8.7 から worker thread を生成できるようになったので試しに使ってみたが，効果を体感できなかった．う～む....

## qt-5.2.1 (KoboRoot.tgz) ##
ベースにするバージョンは 5.2.1 のまま，今後あるかもしれない sqlite の更新を容易にするために `-system-sqlite` を付与してコンフィグレーションし
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -ftree-vectorize
```
を付加してコンパイルしたものである．

今回は sqldriver プラグイン部分だけを収めている，あしからず．

## libxml2-2.9.2 (KoboRoot.tgz) ##
ベースにするバージョンを 2.9.2 に変更し，
```
-mthumb -march=armv7-a -mtune=cortex-a8 -mfpu=neon -ftree-vectorize
```
を付加してコンパイルしただけのものである．

2.9.2 で CVE-2014-3660 と CVE-2014-0191 が修正された．

## dropbear-2014.66 (KoboRoot\_hack.tgz) ##
これは kobo 上で動作するアプリケーションを開発する方向けのモノである．

ひょんなことから Eclipse RSE を使う機会に恵まれたのだが，結構面白かったので kobo でも使えるようにするために用意してみた．内容的には以下の改良を加えている．

  * エンディアン変換マクロを gcc の組み込み関数 `__builtin_bswap32()` に置き換え
以上改良を加えた上で，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -ftree-vectorize
```
を付加してコンパイルしただけのものである．

なお dropbear-2013.59 以降では CVE-2013-4421/4434 が修正されている．

dropbear-2013.62 以降では，楕円曲線暗号のサポートとホスト鍵の自動生成機能が付加された．

以下に当方が ssh 接続するまでに行った手順を記述する．好みに応じて適宜変更して使用してほしい．

  1. ホスト鍵を生成するディレクトリを作成する
```
# mkdir /etc/dropbear
```
  1. inetd 経由で起動するよう仕込みをする(たぶん inetd の仕込みはできてるよね?)
```
# vi /etc/inetd.conf
22      stream  tcp     nowait  root    /bin/dropbearmulti dropbear -i -B -R
# kill -HUP (inetd の PID)
```
以上で root のパスワードなしのまま ssh 接続できると思われる(ttssh2 などでチェックされたい)．初回接続時に該当暗号アルゴリズムのホスト鍵がなかった場合には自動生成処理が走るため接続に失敗するかもしれないが，次からは普通に接続できるようになる．

さらに後述の openssh(に含まれる sftp-server) と組み合わせることで Eclipse RSE のだいたいの機能が使用可能になる．

## openssh-6.3p1 (KoboRoot\_hack.tgz) ##
これは kobo 上で動作するアプリケーションを開発する方向けのモノである．

ひょんなことから Eclipse RSE を使う機会に恵まれたのだが，結構面白かったので kobo でも使えるようにするために用意してみた．前出の dropbear と合わせることで Eclipse RSE の，だいたいの機能は使えるようになる．

dropbear には sftp が含まれていないため，sftp-server だけをこちらから流用した．内容的には
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -ftree-vectorize
```
を付加してコンパイルしただけのものである．

dropbear を経由して起動されるので，特に追加設定の必要はない．

openssh-6.4 が出ているが，ssh に対するセキュリティフィックスだけなので今回は見送っている．

## gdb-7.7 (KoboRoot\_hack.tgz) ##
これは kobo 上で動作するアプリケーションを開発する方向けのモノである．

ホストからのリモートデバッグができるよう，gdbserver を，また今回からは gdb 本体も入れてみた．存分にデバッグして欲しい(そんな人いる?)．

リモートデバッグについては nickel の PID に attach しておき，devkitPro の arm-none-eabi-gdb から手動接続し，info registers, info float，info threads の出力とか，^C による実行の中断，continue による実行の再開とかもやってみた．
```
(gdb) info registers
r0             0xfffffdfe       4294966782
r1             0x10eb370        17740656
r2             0x10eb588        17741192
r3             0x10eb7a0        17741728
r4             0x7ed09808       2127599624
r5             0x7ed0992c       2127599916
r6             0x10eb588        17741192
r7             0x8e     142
r8             0x10eb370        17740656
r9             0x7ed09808       2127599624
r10            0xe      14
r11            0x415    1045
r12            0x0      0
sp             0x7ed097d8       0x7ed097d8
lr             0x30aa8fe4       816484324
pc             0x30a936b0       0x30a936b0
cpsr           0x80000010       -2147483632
(gdb) info threads
Cannot access memory at address 0x7c3
  Id   Target Id         Frame
  6    Thread 924        0x30a936b0 in ?? ()
  5    Thread 923        0x3081f6b0 in ?? ()
  4    Thread 844        0x30a936b0 in ?? ()
  3    Thread 843        0x30a936b0 in ?? ()
  2    Thread 839        0x30a936b0 in ?? ()
* 1    Thread 766        0x30a936b0 in ?? ()
Cannot access memory at address 0x7c3
```
ちゃんと言うことをきくようなので，恐らく Eclipse CDT や Qt Creator からでも使えると思う．あれ，0x3000\_0000 番台で止まってる，shared library は 0x4000\_0000 からじゃなかったっけな....

## i2c-tools-3.1.0 (KoboRoot\_hack.tgz) ##
これは kobo 上で動作するアプリケーションを開発する方向けのモノである．

ベースにするバージョンを 3.1.0 に変更し，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -ftree-vectorize
```
を付加してコンパイルしただけのものである．i2cset/i2cget は入っているのに i2cdetect/i2cdump が入っていないので，入れてみようかと(完全にハード解析用....)．

デバイスファイルを見ると，
```
# ls -al /dev/i2c-*
crw-rw----    1 root     root       89,   0 Apr 26 07:54 /dev/i2c-0
crw-rw----    1 root     root       89,   1 Apr 26 07:54 /dev/i2c-1
crw-rw----    1 root     root       89,   2 Apr 26 07:54 /dev/i2c-2
```
と I2C バスは 3 つあり，i2cdetect を使ってみると
  * I2CBUS 0 のスレーブアドレス 0x50 で UU (たぶん Neonode zForce の MSP430)
  * I2CBUS 1 のスレーブアドレス 0x48 でなにやら検出 (たぶん LM75A)
  * I2CBUS 2 のスレーブアドレス 0x43 で UU (たぶん MSP430)

ということになった．

ここで LM75A(温度センサー) のデバイスシートを確認すると，i2c スレーブアドレスは 0b1001XXX ということでとり得る値の範囲は 0x48 から 0x4f ということになる．なので I2CBUS 1 に接続されている 0x48 のデバイスは LM75A ではなかろうかと思われる．

試しに i2cdump を使ってみると....
```
# i2cdump -y 1 0x48 w
     0,8  1,9  2,a  3,b  4,c  5,d  6,e  7,f
00: 4018 ff00 004b 0050 ffff ffff ffff ffff
08: 4018 ff00 004b 0050 ffff ffff ffff ffff
    :
```
と，出てくる．レジスタ 0 が 0x1840 で下位 5bit が無効だから値としては 0b00011000010 = 194 となり，0.125 を乗じて 24.25 度と検出しているようである．
レジスタ 2 と 3 がデフォルト値なので，これはもう確定でしょ．

あと /proc/interrupts を確認してみると imx-i2c がバスの数だけ(irq 62，63，64)あり，62 と 64 は画面をタッチするたびに値がカウントアップされる．このあたりが MSP430 との通信用ではなかろうかと思われる．

IC2BUS 0 と 2 で UU が検出されるアドレスがあるところを見ると，実は i.MX がスレーブ動作しているのでは? などと思われ．しかし u-boot のソースコードを読んでいると，0x43 に対して i2c\_read/i2c\_write していることがわかる．はて，マスターとスレーブを切り替えて動いているということだろうか，仕様的には可能だけどそういう使い方をあまり見かけない....

レジスタマップメモ

|レジスタアドレス|中身|R/W|レジスタアドレス|中身|R/W|
|:-----------------------|:-----|:--|:-----------------------|:-----|:--|
|0x00|プロセッサバージョン|R only(?)|0xa1|フロントライト?|W only(?)|
|0x04|年月日時分秒|R only(?)|0xa2|フロントライト?|W only(?)|
|0x05|年月日時分秒|W only(?)|0xa3|フロントライト?|W only(?)|
|0x08|? |R only(?)|0xa4|フロントライト?|W only(?)|
|0x10|年|W only(?)|0xa5|フロントライト?|W only(?)|
|0x11|月|W only(?)|0xa6|フロントライト?|W only(?)|
|0x12|日|W only(?)|0xa7|フロントライト?|W only(?)|
|0x13|時|W only(?)|  | | |
|0x14|分|W only(?)|  | | |
|0x15|秒|W only(?)|  | | |
|0x16|? |W only(?)|  | | |
|0x20|年月|R only(?)|  | | |
|0x21|日時|R only(?)|  | | |
|0x23|分秒|R only(?)|  | | |
|0x41|? |R only(?)|  | | |
|0x42|? |R only(?)|  | | |
|0x60|? |R only(?)|  | | |

zForce の Linux ドライバを発見した( http://permalink.gmane.org/gmane.linux.kernel.input/31490 )．本家には反映されてないみたいなので，現状はユーザーランドから i2c ドライバ経由でハンドリングしているのだろう．あれ，dmesg に zforce の文字列が( https://github.com/androidzaurus/kobo-touch-rakuten/blob/master/dmesg.txt )．

なんだ，github にあるカーネルコードには zForce ドライバが入っているじゃないか．ほいで，

```
CONFIG_TOUCHSCREEN_ZFORCE=y
```

だから，カーネル組み込みになってる，フムフム．

Neonode の zForce は MSP430 コミなのか( http://www.neonode.com/neonode-announces-collaboration-with-texas-instruments-enabling-next-generation-touch-screen-devices/ )．それで 2 コ載ってるわけなのね．

## strace-4.8 (KoboRoot\_hack.tgz) ##
これは kobo 上で動作するアプリケーションを開発する方向けのモノである．

mobileread のフォーラムなどで strace を使って nickel の挙動などを調べている人がいたので，入れてみる気になった．例えば http://www.mobileread.com/forums/showthread.php?t=200706 とか．

特に特別なことをしておらず，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -ftree-vectorize
```
を付加してコンパイルしただけのものである．

nickel に attach して open をトレースすると，
```
# strace -e open -p 763
Process 763 attached
open("/sys/devices/platform/pmic_battery.1/power_supply/mc13892_bat/capacity", O_RDONLY|O_LARGEFILE|O_CLOEXEC) = 17
open("/sys/devices/platform/pmic_battery.1/power_supply/mc13892_bat/status", O_RDONLY|O_LARGEFILE|O_CLOEXEC) = 17
open("/sys/devices/platform/pmic_battery.1/power_supply/mc13892_bat/status", O_RDONLY|O_LARGEFILE|O_CLOEXEC) = 17
    :
```
という感じになった．

## libuuid-1.0.2 (KoboRoot\_hack.tgz) ##
これは kobo 上で動作するアプリケーションを開発する方向けのモノである．

特に特別なことをしておらず，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -ftree-vectorize
```
を付加してコンパイルしただけのものである．

後述の parted-2.4 のために導入した．単独では意味を成さない．

## parted-2.4 (KoboRoot\_hack.tgz) ##
これは kobo 上で動作するアプリケーションを開発する方向けのモノである．

特に特別なことをしておらず，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -ftree-vectorize
```
を付加してコンパイルしただけのものである．

PC のハードディスク交換時に gparted を使っている．kobo の FAT32 領域(/dev/mmcblk0p3) のリサイズにも使えるのではないかと思い立って試してみたところ，何事も無く使えてしまったので含めてみた．

内蔵 microSD(HC) が 4GB の個体の場合，手順を工夫すれば殻割りせず，かつ既存のデータを保持したまま領域を増やすことができる．

以下を参考にされたい．http://www.mobileread.com/forums/showthread.php?t=230158

ざっくり手順は以下のとおり．
```
# killall nickel
# umount /dev/mmcblk0p3
# parted
GNU Parted 2.4
Using /dev/mmcblk0
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print
print
Model: SD 00000 (sd/mmc)
Disk /dev/mmcblk0: 8166MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type     File system  Flags
 1      9961kB  278MB   268MB   primary  ext4
 2      278MB   547MB   268MB   primary  ext4
 3      547MB   1978MB  1431MB  primary  fat32

(parted) resize 3
resize 3
WARNING: you are attempting to use parted to operate on (resize) a file system.
parted's file system manipulation code is not as robust as what you'll find in
dedicated, file-system-specific packages like e2fsprogs.  We recommend
you use parted only to manipulate partition tables, whenever possible.
Support for performing most operations on most types of file systems
will be removed in an upcoming release.

Start?  [547MB]? 

End?  [1978MB]? -1
-1
moving data... 88%      (time left 00:00)
(parted) print
print
Model: SD 00000 (sd/mmc)
Disk /dev/mmcblk0: 8166MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number  Start   End     Size    Type     File system  Flags
 1      9961kB  278MB   268MB   primary  ext4
 2      278MB   547MB   268MB   primary  ext4
 3      547MB   8166MB  7619MB  primary  fat32

(parted) quit
quit
# reboot
```

最新のリリースは 3.1 なのだが，3.0 以降はファイルシステム依存の実装を排除したとのこと(気持ちはよくわかる)で古いバージョンを採用した．

## busybox-1.22.1 (KoboRoot\_hack.tgz) ##
ベースにするバージョンを 1.22.1 に変更し，特にコンパイルフラグを付加せずにただコンパイルしただけのものである．

最初から入っているものはいくつかの機能が削れられていたので，いちおう自動で生成できるフルフルのコンフィグレーションで作ってみた．

また，下手にあれこれコンパイルフラグを付けるとかえってサイズが大きくなってしまう傾向にあったので，何も付けないことにした．

# 開発環境について #
開発ホストには Ubuntu 14.04 LTS を使用している(より正確には VirtualBox 上で Ubuntu を動作させている)．

開発ツールチェインには，https://github.com/kobolabs/Kobo-Reader の toolchain/gcc-linaro-arm-linux-gnueabihf-4.8-2013.04-20130417\_linux.tar.bz2 を使用している．

clang + llvm 3.4.1 も併用している． ~~Ubuntu 14.04 にしてから具合が悪く，調査中．~~ コツがつかめてきた．さらに 3.5.0 も使い始めた．

Q8200 から E3-1225v2 にして Qt のコンフィグレーション、コンパイルが苦にならなくなった、快適。

# DOING #
  * 外部 SD 起動機能を使ってフルバックアップ，文鎮復旧を実現するための準備を始める．
    * とりあえず ddrescue を hack 側に含めて動かしてみる．
    * やりかたがまとまったらメモを書く．

# TODO #
  * できるものから clang + llvm でコンパイルしてみる．
  * byte swap のコードを gcc の組み込み関数に置換する(`__builtin_bswap32()` とかね)．
  * unrar-5.21 release が出ているので試してみる．
  * sqlite-3.8.8.3 が出ていたので試してみる．
  * libpng-1.6.17rc1 が出ていたので試してみる．
  * busybox-1.23.1 で stable になったので試してみる．
  * dropbear-2015.67 が出ていたので試してみる．

# PENDING #
  * sqlite-3.7.17 から導入された mmap を試してみる．
  * i2ctools-3.1.1 が出ていたので様子を見てみる．
  * openssl ちょっと改良．
    * 1.0.2 がキタ．
    * エンディアン変換のコードをちょっと書き換えてみた．パフォーマンス測定をお楽しみに．
    * 差が感じられないので保留．
```
vld1.64  {d16-d17},[ip]
vrev64.8 q8,q8
vmov     r1,r0,d16
vmov     r3,r2,d17
bl       _armv4_AES_decrypt
vmov     d16,r1,r0
vmov     d17,r3,r2
vrev64.8 q8,q8
vst1.64  {d16-d17},[ip]
```

# おまけ #

## リカバリの簡略化 ##
通常のリカバリ操作では，一旦工場出荷状態にして再起動してから最新のファームウエアに更新する必要がある．やや敷居が高いが，これを簡略化する方法について記載する．
  * recoveryfs 内の /upgrade/fs.tgz を用意する．
  * リカバリ動作で初期状態にしたいバージョンの KoboRoot.tgz を用意する．
  * 両方のファイルを gunzip して tar ファイルにする．
  * 2 つの tar ファイルを結合し，新たな fs.tar を作成する．
```
% tar --concatenate --file fs.tar KoboRoot.tar
```
  * gzip 圧縮して新たな fs.tgz を作成する．
  * できた fs.tgz を recoveryfs の /upgrade 以下に上書きする．
  * ついでに recoveryfs 内の /upgrade に KoboRoot.tgz を用意した中にある upgrade の中身(u-boot.bin と uImage)をコピーしておく．
これで通常のリカバリ操作だけで，最初から任意のバージョンで動作させられる．

手元の fs.tgz が約 110 MB，kobo-update-2.4.0.zip の KoboRoot.tgz が 68MB ということで，結合すると約 180MB くらいになった．

recoveryfs のパーティションサイズが約 256MB なのでアップデートに含まれる KoboRoot.tgz が今後肥大化していくと，このテが使えなくなるかもしれない．両方の tar ファイルを fs 上にほどいて tar ファイルを作りなおすと小さくできると思うが，uid や gid，パーミッションあたりに気を遣う必要がある(と思う)．

## ダメ文字はなぜ治らないのか ##
rakuten の怠慢ではないかと。

http://download.kobobooks.com/magento/userguides/downloads/KoboTouch/kobotouch_userguide_jp.pdf

の 51 ページ「EPUB、PDF、DRM について」を参照すると、読みたい本を電子ブックリーダーに移動する方法がいくつか書いてあって、その中に

  * microUSB ケーブルで PC に接続して、ドラッグ ＆ ドロップで本を移動する

とある。他のブックストアで購入したモノを入れることを許容しているのだから、rakuten として使用できるファイル名に制限を加えることは事実上不可能なわけで、ダメ文字問題は仕様通りの使い方で再現可能である。

そんなわけで、電凸したら http://kobo.faq.rakuten.ne.jp/app/answers/detail/a_id/19356/kw/ を盾にしてきた、やれやれ。いちいちこんなところ見ないよ、紛らわしいからマニュアルからドラッグ & ドロップの記載を削除してよね。

以下、蛇足。

たまたま kobo にファイル転送しつつ本棚を自動作成する ruby スクリプトを公開しているサイトを発見した．ここの「備考」の記載を一部引用すると....
```
ファイル名に日本語などが使われる場合は、 (Koboの仕様っぽい)不具合の回避のために Base64 っぽいリネームが行なわれます。 
```
とある．作者さんに問い合わせたところ、転送ファイルは epub と pdf だけに制限しているとのことで、cbz などには作用しないことがわかった。残念。でも、これを改良するのが現実的な解ってところではなかろうか、転送めんどくさいけど。

## auto\_vacuum を有効にしてみる ##
sqlite には auto\_vacuum 機能がある．

簡単に試したいところであるが，残念ながら
```
Therefore, auto-vacuuming must be turned on before any tables are created. It is not possible to enable or disable auto-vacuum after a table has been created.
```
ということでやや敷居が高い．当方が試した手順をちょっと書いておくので参考にされたい．

  * ダンプする (処理系が UTF-8 を通す状態で実行すること)
```
% echo '.dump' | sqlite3 (some where)/.kobo/KoboReader.sqlite > KoboReader.txt
```
  * 前述の英文にある通り KoboReader.txt の CREATE TABLE よりも前に PRAGMA を追加
```
% vi KoboReader.txt
PRAGMA auto_vacuum=1;
```
  * db ファイルを復元する
```
% sqlite3 KoboReader.sqlite < KoboReader.txt
```
  * 既存のファイルと置き換える(別途バックアップはとっておくこと)
```
 % mv KoboReader.sqlite (some where)/.kobo/KoboReader.sqlite
```

## ChainLP で mozjpeg ##
mozjpeg はファイルサイズをより小さくするというプロジェクトなので，kobo に内蔵するのは意味が無いだろうとスルーしていた．

ふと思い立って ChainLP で使う方法がないかと調べてみたところ，詳細設定の「画像」タブにある「画像形式」に「External」を指定することで外部コマンドを起動できることがわかった．それじゃぁ，と mozjpeg の cjpeg を使えるように Visual Stduio をウニャウニャし，さらにゴニョゴニョすることで無事に使えるようになった．

手元の自炊ファイルを処理させると，えらいサイズが小さくなる．Windows 内蔵のエンコーダーがバカなのか，IJG の実装がいいのか，mozjpeg が優秀なのか，サッパリわからない．そんなわけで mozjpeg だけでなく IJG の cjpeg も用意し，比較．

|使用エンコーダー(設定)|198ページものの漫画|285ページものの小説|
|:-------------------------------|:--------------------------|:--------------------------|
|ChainLP JPEG (デフォルト)|32,570,656 bytes|35,975,913 bytes|
|IJG jpeg-9a (デフォルト)|29,344,535 bytes|  |
|mozjpeg-2.1 (デフォルト)|23,296,104 bytes|26,331,081 bytes|
|mozjpeg-2.1 (force grayscale)|23,017,331 bytes|  |

結果だけを見ると mozjpeg が優秀，という事になりそうだ．妥当な比較なのかやや怪しいけれども．画像1枚の伸長時間も比較してみた．

|使用エンコーダー(設定)|ファイルサイズ|伸長時間(djpeg 100回ループ)|
|:-------------------------------|:--------------------|:----------------------------------|
|IJG jpeg-9a(デフォルト)|162,889byte|4.96sec|
|IJG jpeg-9a(progressive)|150,758byte|10.29sec|
|mozjpeg-2.1 (デフォルト)|130,735byte|7.51sec|
|mozjpeg-2.1 (progressive)|129,478byte|4.71sec|

肝心の出力画像だが，見比べた範囲では差を感じなかったので mozjpeg を常用しよう．

要望があれば cjpeg の実行バイナリを kobohack-j に入れられるか検討するのだけれども，欲しい人いないよね?

## jpeg と png どっちがいいのか ##
kobo で cbz や cbr を見るときの画像フォーマットは jpeg と png どっちがいいのか．その昔 16 階調グレースケール png 1 択というのをどこぞで読んだ気がするが，所有スキャナの出力は jpeg か pdf なので jpeg にしていた．

さて，ホントのところは如何に.... mozjpeg で頑張ってみたので，調子に乗ってもうちょっと頑張ってみた．libpng-1.6.13 を ImageMagick 経由で，以下のように変換を行った．

```
% convert -depth 4 -colorspace gray -format png -geometry 758x1024 hogehoge.jpg hogehoge.png
```

|元画像 jpeg (1106 x 1600)|445,419byte||
|:---------------------------|:----------|:|
|mozjpeg-2.1 (758 x 1024, 8bpp, grayscale, progressive)|129,478byte|4.71sec (djpeg 100回ループ)|
|libpng-1.6.13 (708 x 1024, 4bpp, grayscale)|155,478byte|10.99sec (timepng 100回ループ)|

ふむ，どうやら jpeg の方がいいらしい(timepng 使って良かったのだろうか)．

## 隠し(?)サポートフォーマット ##
ひょんなことから kobo は **xps フォーマットをサポート** しているっぽいことに気づいた．試したらちゃんと閲覧できたのだけれど，残念ながら自分的には使い道がない．価値を見い出す人がいるかもしれないので念のため触れておく．

xps については以下を参照されたい．

  * http://msdn.microsoft.com/ja-jp/magazine/ee156489.aspx
  * http://ja.wikipedia.org/wiki/XML_Paper_Specification

Windows マシンであれば，仮想プリンタ **Microsoft XPS Document Writer** を使うことで xps ファイルを簡単に生成できる．どこまで真面目にサポートされているかが未知数なので，期待しないほうが良いのかもしれないけど．

.... ん? Mobipocket ってのもサポートしているっぽいんだけど，これって....

## 愛称一覧 ##
|愛称|製品名|
|:-----|:--------|
|trilogy|kobo Touch|
|pixie|kobo mini|
|kraken|kobo glo|
|dragon|kobo Aura HD|
|phoenix|kobo Aura 6"|
|dahlia|kobo Aura H2O|
|alyssum|???|

# おわりに #
本文書の最新版は http://code.google.com/p/kobohack-j/wiki/README_261 で公開されているので，近況についてはこちらを参照されたい．