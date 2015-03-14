# 配布物について #
現状の配布物には，以下のものが含まれている．
  * u-boot\_hack.bin
  * KoboRoot.tgz
    * jpeg-turbo-1.3.0
    * zlib-1.2.8
    * libpng-1.2.50
    * dosfstools-3.0.17
    * libxml2-2.7.8
    * freetype-2.4.12
    * expat-2.1.0
    * busybox-1.20.2
    * unrarsrc-4.2.4
  * KoboRoot\_hack.tgz
    * dropbear-2013.58
    * openssh-6.2p1
    * gdb-7.6
    * i2c-tools-3.1.0
    * strace-4.7
  * README.wiki (本ファイル)

# 差分の簡単な説明 #
## u-boot\_hack.bin ##
この u-boot\_hack.bin を内蔵 SD に書き込むことで，外部 SD 起動の条件が変更され容易に実現可能になる．
|旧条件|外部 SD スロットにカードが装着されいる && ホームキーが押されている && 電源スイッチがスライドされたままである．|
|:--------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------|
|新条件|外部 SD スロットにカードが装着されいる．|

外部 SD 起動は uImage を外部 SD からロードし，かつ root=/dev/mmcblk1p1 で起動するため，カーネルや rootfs の差し替えが簡単にできハックに大変有用である．

なお本バイナリは u-boot の書き換え方法を十分に理解できている方々向けのものである(文鎮化しても動じない，等)．

## jpeg-turbo-1.3.0 (KoboRoot.tgz) ##
http://libjpeg-turbo.virtualgl.org/ で配布されているものを
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=softfp
```
を付加してコンパイルしただけのものである．

なお
ttp://ux.getuploader.com/KOBO/download/1/Kobo\_libjpeg-turbo\_updater\_20120824.zip
は当方が作成したものでなく，配布物内で使用するツールチェインを統一させたいため自分でコンパイルしなおしている(採用バージョンも異なる)．

## zlib-1.2.8 (KoboRoot.tgz) ##
ベースにするバージョンを 1.2.8 に変更した以外に，以下の改良を加えている．
  * adler32 の neon 実装を g2cd プロジェクト http://sourceforge.net/projects/g2cd/ から拝借した
  * inffast の neon 実装を https://bugzilla.mozilla.org/show_bug.cgi?id=462796 から拝借した
  * 拝借する上でコンパイルが通る程度に上述の実装や Makefile.in 等あちこち手を入れた
以上改良を加えた上で，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=softfp
```
を付加してコンパイルした．

### 性能評価結果 ###
使用データには /usr/local/Kobo/slideshow/ja/7.gz (伸長前 116,085byte，伸長後 2,179,072byte) を，伸長処理には zlib 付属の examples/gun を
```
#!/bin/sh
for i in `seq 1 1000`
do
    cat $1 | ./gun > /dev/null
done
```
と 1,000 回繰り返し実行するシェルスクリプトを用いて time コマンドで計測した．スクリプトファイル，評価データはすべて /tmp 以下に置き SD へのアクセスが行われないよう配慮した．

標準 (libz.so.1.2.4)
```
# time ./test.sh 7.gz
real    0m 35.07s
user    0m 15.83s
sys     0m 5.30s
```

neon 実装 (libz.so.1.2.7)
```
# time ./test.sh 7.gz
real    0m 29.75s
user    0m 16.44s
sys     0m 4.70s
```

neon 実装 + crc32 アセンブラ実装 (libz.so.1.2.7)
```
# time ./test.sh 7.gz
real    0m 39.06s
user    0m 26.02s
sys     0m 5.09s
```

neon 実装の場合，比較的圧縮率がよい本評価データでは約 15% の速度向上が見られた．

一方 crc32 のアセンブラ実装の場合は，予想に反して標準よりも遅くなった．これは crc32.c に実装されている BYFOUR が有効に作用しているのだろう，と邪推している．~~これを凌駕する neon 実装を探さなければ(あるのか?)．~~

そんなわけで，zlib は neon 実装だけ追加したものを採用している．

なお crc32 のアセンブラ実装は http://members.efn.org/~rick/work/u-boot/lib_arm/crc32.S から拝借し，crc32\_table は crc32.h を `-e 's/UL//g' -e 's/^ *0x/.long 0x/'` などとやってごまかした(!?)．

crc32 は逐次演算が基本なので，neon のような並列実行型での実装には不向きなアルゴリズムである(ということに気づいた)．したがってベクタペアワイズ加算の XOR 版のようなものがなければ(あっても?)，有効な代替実装ができないんじゃないかと．イマドキの Intel 系 CPU には crc32 ニーモニックがあるのがうらやましい．

もうちょっと高速化できないかと inffast の neon 実装で 16byte 未満のメモリコビーがバイト単位で実行されているのを見つけたので，http://sourceware.org/ml/libc-ports/2009-07/msg00003.html のコードを参考に書き換えてみた．

元々がオフセット付きのプレインデックス構文で書かれているため，unaligned access のコードが置き換えられずにいるのだが，とりあえず aligned access のやつはできて，性能評価結果は以下のようになった．1000 回実行で 500ms 差って....

neon 実装改 (現状の実装，libz.so.1.2.7)
```
# time ./test.sh 7.gz
real    0m 29.20s
user    0m 16.28s
sys     0m 4.84s
```
今回あれこれ調べたおかげで，結構多くの亜流が存在することがわかった．次回へ続く!?

## libpng-1.2.50 (KoboRoot.tgz) ##
ベースにするバージョンを 1.2.50 に変更し，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=softfp -ftree-vectorize
```
を付加してコンパイルしただけのものである．

標準で使用されている 1.2.43 を含む 1.2.50 よりも前のバージョンでは多数の CVE が公開されているため，入れ替えた方が無難である．

libpng の neon 実装は，例えば png\_read\_filter\_row() なら　https://bugzilla.mozilla.org/show_bug.cgi?id=462796 あたりで公開されていたりするが，イマイチ効果が確認できなかった(常用してないからあまりやる気もなかった)．しかし gcc のコンパイルオプションを変更するだけで，以下のように neon コードが吐き出されることがわかったため，これでよしとした．
```
    518:        e2811001        add     r1, r1, #1
    51c:        edd01b00        vldr    d17, [r0]
    520:        r1510005        cmp     r1, r5
    524:        ecfc0b02        vldmia  ip!, {d16}
    528:        f24108a0        vadd.i8 d16, d17, d16
    52c:        ece00b02        vstmia  r0!, {d16}
    530:        3afffff8        bcc     518 <png_read_filter_row+0xcc>
```
蛇足であるが zlib に対しても同じ事をやってみたところ，コードに変化が見られなかった．

ちょっとゴニョゴニョしてみたところ，zlib は -O3 でコンパイルされているのに対して libpng は -O2 であったことに気づき，-O2 -ftree-vectorize で zlib  をコンパイルしたところ，あちこちに neon コードが出てきた．

どっちが速いのか比べた方がよいのだが，まぁ気が向いた時にでも．

### 性能評価結果 ###
気が向いたらやる．

## dosfstools-3.0.17 (KoboRoot.tgz) ##
ベースにするバージョンを 3.0.17 に変更し，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=softfp -mthumb -mthumb-interwork
```
を付加してコンパイルしただけのものである(不可抗力(!?)で thumb バイナリになってしまった)．

/etc/init.d/rcS に dosfsck -a -w /dev/mmcblk0p3 を実行するように書かれているが，ChangeLog を確認すると 3.0.11 あたりに https://bugzilla.redhat.com/show_bug.cgi?id=660154 と不安になるような記載があるため念の為に入れることにした．

なお KoboReader/build/script/dosfstools.sh の記載に倣い，dosfsck と mkdosfs だけを /bin 以下に配置している(~~mkdosfs は recoveryfs に入れないと意味ないかも~~ recoveryfs では mkfs.vfat(busybox) を使っていた(ぉぃぉぃ)．どういう使い分けをしているんだ?)．

3.0.16 への更新の過程で，FAT のボリュームラベルは小文字を使ってはいけないことを知った．そうすると，recoveryfs の /etc/init.d/rcS に書いてある
```
mkfs.vfat -n KOBOeReader /dev/mmcblk0p3 && sync && sync
```
は仕様外の使い方だということに，やれやれ．~~ここを~~
```
mkdosfs -F 32 -r 32768 -s 64 -n KOBOEREADER /dev/mmcblk0p3 && sync && sync
```
~~とやることで，リカバリ後に良いことがあるかもしれない(microSD 増量ユーザー向けかな)．~~

ちょっとした都合で開発機の MicroSDHC を SanDisk の SDSDQUA-032G-U46A から Transcend の Class6 品に変えたところ，書籍の追加(sqliteファイルの更新)が我慢できないくらい遅くなった．単純な Class の差とは思えないくらい，である．なので，クラスタサイズを変えることは小手先の回避法であって，根本解決にならないと予想する．

I/O スケジューラーの設定は cfq になっているので一概には言えないが，やはりランダムアクセス重視じゃないとダメじゃないかと．

## libxml2-2.7.8 (KoboRoot.tgz) ##
ベースにするバージョンを 2.7.8 に変更した以外に，以下の修正を加えている．
  * https://mail.gnome.org/archives/xml/2010-November/msg00016.html
以上修正を加えた上で，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=softfp
```
を付加してコンパイルした．

http://xmlsoft.org/news.html の 2.7.7 のリリースノートには **libxml violates the zlib interface and crashes** などと書かれているので，2.7.8 に入れ替えておいた(2.8.0 以降では soname が変わってしまうので入れ替えられず)．

あちこちで double 型や float 型を使用する API があるっぽく，これらが VFP 命令を使うようになったため，少しは速度向上に寄与するかもしれない．

## freetype-2.4.12 (KoboRoot.tgz) ##
ベースにするバージョンを 2.4.12 に変更した以外に，以下の改良を加えている．
  * http://mimosa-pudica.net/unix-font.html なる良い文書を見つけたので，ここに記載のパッチを適用
  * http://sourceforge.net/projects/freetype/files/freetype2/2.4.12/ に書かれているよう，CFF フォントでは Adobe のエンジンが有効になるよう修正
以上改良を加えた上で，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=softfp -mtree-vectorize
```
を付加してコンパイルした．

改良の内容にサブピクセルレンダリングの有効化があるのだが，これは http://www.freetype.org/patents.html の Other Patent Issues に記載されているモノである(だからデフォルトのコンフィグレーションでは無効になっている)．

とは言えそれだけのハズなのに，なぜかサイズが半分以下になるようである．
```
-rwxrwxr-x    1 root     root        477356 Feb 20  2013 libfreetype.so.6.10.0
-rwxr-xr-x    1 root     root       1199080 Jul 17  2012 libfreetype.so.6.6.2
```
某所では Monotype のレンダラが入っているっポイ，という情報があり，その分が根こそぎサイズ減につながっているのかもしれない．と言っても半分はないだろう，と思うけれども．

2.4.12 から Adobe の cff フォントのラスタライズ技術が取り込まれている(http://blogs.adobe.com/CCJKType/2013/05/adobe-contributes-cff-rasterizer-to-freetype-jp.html )．付属ドキュメントに倣い，デフォルトで有効になるようコードを修正して使ってみたところ，以前に比べてルビが読みやすくなった(2台並べて見比べた)．

また動作上使われないであろう freetype 組み込みの zlib 実装を，外部 shared libaray を使うようにコンフィグレーションした(サイズを小さくするつもりが -mtree-vectorize を付けたので相殺された)．

また aokin フォントで「フォント設定共通化用テンプレートファイル」を見ると固まる，という話もあるようだが，そのような現象は見られないようである．その時のバックトレースが不完全(strip されてるから)ながら記録されているのだが，
```
#02 sp: 0x7efa7420 ip: 0x30b12ee8 /lib/libc-2.11.1.so: __fsetlocking+0x3e8 
#03 sp: 0x7efa7a50 ip: 0x30b1d49c /lib/libc-2.11.1.so: malloc_usable_size+0x770 
#04 sp: 0x7efa7a80 ip: 0x2fd98b58 /usr/lib/libfreetype.so.6.6.2: FT_Stream_OpenBzip2+0x73a5c 
#05 sp: 0x7efa7a88 ip: 0x2fd18a44 /usr/lib/libfreetype.so.6.6.2: itype_rdr_make_itype_outline+0x8d4 
#06 sp: 0x7efa7ce0 ip: 0x2fcc8240 /usr/lib/libfreetype.so.6.6.2: FT_Outline_Render+0x88 
#07 sp: 0x7efa7d30 ip: 0x2fcc8310 /usr/lib/libfreetype.so.6.6.2: FT_Outline_Get_Bitmap+0x44 
```
freetype 内の bzip2 モジュールで落ちているのか? という話がある．当方の見立てでは freetype 内に抱え込んでいる整数割り算ルーチンではないか，と考えている．調べた時のメモを無くしてしまったのだけど....

## expat-2.1.0 (KoboRoot.tgz) ##
ベースにするバージョンを 2.1.0 に変更し，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=softfp
```
を付加してコンパイルしただけのものである．

前バージョンである 2.0.1 から実に 5 年近く経っていて，更新されたことを知らない人が多いんじゃないかと思う．こちらも多数の CVE が報告されているため，入れ替えた方が無難である(Changes ななめ読みしただけでも 5 つほど)．

でも，どこからも参照されていないような感じがしてならない．

## busybox-1.20.2 (KoboRoot.tgz) ##
ベースにするバージョンを 1.20.2 に変更し，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=softfp -mthumb -mthumb-interwork
```
を付加してコンパイルしただけのものである．

configs/android\_defconfig を見てみたところ thumb オプションが付加されていたので，真似してみた．サイズ比較した結果は以下のとおりである．
```
-rwxr-xr-x    1 root     root        879928 May 31  2011 busybox (busybox-1.17.1 オリジナル)

-rwxr-xr-x    1 root     root        904292 Mar  9 10:07 busybox (busybox-1.20.2 ARM 版)
-rwxr-xr-x    1 root     root        666724 Sep  5 10:12 busybox (busybox-1.20.2 thumb 版)
```
thumb 版はサイズが小さい分だけファイル読み出しやメモリアロケーションにかかる時間のコストが小さくなることを期待している．しかし純粋な命令実行速度は ARM 版の方が早いハズなので，電源 ON 時のアプリ起動速度を bootchart で比較した．すると nickel 起動までの時間にさほど差がでなかったので，これで良しとする(.config をいじればもっと小さくできるのだが)．

また 1.21.0 も出ているのはもちろん把握しているが unstable であるため，stable 版である 1.20.2 を採用した．

## unrarsrc-4.2.4 (KoboRoot.tgz) ##
ベースにするバージョンを 4.2.4 に変更し，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=softfp -ftree-vectorize
```
を付加してコンパイルしただけのものである．

ただしソースコードを変更せずにコンパイルすると
```
# ./unrar
UNRAR 4.20 freeware      Copyright (c) 1993-2012 Alexander Roshal
```
と出てしまうため，4.24 と出るようにちょっとゴニョゴニョした．

## dropbear-2013.58 (KoboRoot\_hack.tgz) ##
これは kobo 上で動作するアプリケーションを開発する方向けのモノである．

ひょんなことから Eclipse RSE を使う機会に恵まれたのだが，結構面白かったので kobo でも使えるようにするために用意してみた．内容的には
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=softfp
```
を付加してコンパイルしただけのものである．

以下に当方が ssh 接続するまでに行った手順を記述する．好みに応じて適宜変更して使用してほしい．

  1. ホスト鍵を生成する
```
# mkdir /etc/dropbear
# cd /etc/dropbear
# dropbearkey -t dss -f dropbear_dss_host_key
# dropbearkey -t rsa -f dropbear_rsa_host_key
```
  1. inetd 経由で起動するよう仕込みをする(たぶん inetd の仕込みはできてるよね?)
```
# vi /etc/inetd.conf
22      stream  tcp     nowait  root    /bin/dropbearmulti dropbear -i -B
# kill -HUP (inetd の PID)
```
以上で root のパスワードなしのまま ssh 接続できると思われる(ttssh2 などでチェックされたい)．さらに後述の openssh(に含まれる sftp-server) と組み合わせることで Eclipse RSE のだいたいの機能が使用可能になる．

## openssh-6.2p1 (KoboRoot\_hack.tgz) ##
これは kobo 上で動作するアプリケーションを開発する方向けのモノである．

ひょんなことから Eclipse RSE を使う機会に恵まれたのだが，結構面白かったので kobo でも使えるようにするために用意してみた．前出の dropbear と合わせることで Eclipse RSE の，だいたいの機能は使えるようになる．

dropbear には sftp が含まれていないため，sftp-server だけをこちらから流用した．内容的には
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=softfp
```
を付加してコンパイルしただけのものである．

dropbear を経由して起動されるので，特に追加設定の必要はない．

## gdb-7.6 (KoboRoot\_hack.tgz) ##
これは kobo 上で動作するアプリケーションを開発する方向けのモノである．

ホストからのリモートデバッグができるよう，gdbserver を入れてみた．nickel の PID に attach しておき，devkitPro の arm-none-eabi-gdb から手動接続し，info registers, info float，info threads の出力とか，^C による実行の中断，continue による実行の再開とかもやってみた．
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
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=softfp
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
  * I2CBUS 0 のスレーブアドレス 0x50 で UU
  * I2CBUS 1 のスレーブアドレス 0x48 でなにやら検出
  * I2CBUS 2 のスレーブアドレス 0x43 で UU

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

あと /proc/interrupts を確認してみると imx-i2c がバスの数だけ(irq 62，63，64)あり，62 と 64 は画面をタッチするたびに値がカウントアップされる．このあたりが MSP430 との通信用ではなかろうかと思われる．IC2BUS 0 と 2 で UU が検出されるアドレスがあるところを見ると，実は i.MX がスレーブ動作しているのでは? などと思われ．

## strace-4.7 (KoboRoot\_hack.tgz) ##
これは kobo 上で動作するアプリケーションを開発する方向けのモノである．

mobileread のフォーラムなどで strace を使って nickel の挙動などを調べている人がいたので，入れてみる気になった．例えば http://www.mobileread.com/forums/showthread.php?t=200706 とか．

特に特別なことをしておらず，
```
-march=armv7-a -mtune=cortex-a8 -mfpu=neon -mfloat-abi=softfp
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

# 開発環境について #
開発ホストには Ubuntu 12.04 LTS を使用している(より正確には VirtualBox 上で Ubuntu を動作させている)．

開発ツールチェインには，フリースケール社が提供している i.MX5 開発用ソフトウエア(L2.6.35\_11.04.01\_ER\_source\_bundle.tar.gz)一式に含まれているツールチェイン(L2.6.35\_11.04.01\_ER\_source/pkgs/gcc-4.4.4-glibc-2.11.1-multilib-1.0-1.i386.rpm)を
使用している．

# DOING #
  * 現在情熱を傾けているネタは無い．

# TODO #
  * Qt まるごとコンパイル
    1. ツールチェインの違いなのか，普通にコンパイルすると C++ では ABI が非互換になるっぽいことがわかった．C++ では [r0](https://code.google.com/p/kobohack-j/source/detail?r=0) が this に相当するはずで，オリジナルの方は this を使ってないように見える．ちょっと厄介だなぁ，コンパイルオプションで何とかなったっけ?
    1. なんともならないようなので，Qt をまるごとコンパイルすることを考える．ついでに 4.8.4 に上げてみようかと．これができても KoboRoot.tgz にそのまま入れられないな．KoboRoot\_qt.tgz とかにするか．
    1. ~~うーん.... QtDBus だけがコンパイル通せない．他は大体できたんだけどなぁ．http://qt-project.org/forums/viewthread/20173 と同じようなことが起こっている．namespace あたりの問題か? C++ は苦手なんですけど．~~ pkg-config(PKG\_CONFIG\_PATH) をゴニョゴニョすることで良さそう．dbus 無効になってるっぽい，気づかなんだ....
    1. 楽天に電凸したらgithubにQtのパッチが追加登録された．一緒にtoolchainもついてきたのだけど，はてlinaroのhard-float版が出てきたのはなぜだろう．ほんとにこれ使ってるのか?
```
手元でコンパイルしたモノ
00005680 <non-virtual thunk to QSQLiteDriverPlugin::keys() const>:
    55e4:       e2411008        sub     r1, r1, #8
    55e8:       eaffffff        b       55ec <QSQLiteDriverPlugin::keys() const>

ここに該当する，オリジナルのバイナリ
    5ab8:       e2400008        sub     r0, r0, #8
    5abc:       eaffffff        b       5ac0 <qt_plugin_query_verification_data+0x24>
```
  * libzip の更新
    1. 0.10 でメモリリークもろもろ，0.10.1 で CVE が修正されているが soname が変わってしまっているので，そのまま置き換えられない．
    1. soname をだますだけでなんとかなるかと API 差分も確認したが，まるでダメ．
    1. 差分だけゴニョゴニョする気分になれ ~~るまで，保留~~ たので着手．
    1. CVE-2012-1162 は修正できた，CVE-2012-1163 は 0.10 以降のバグと思われ(つまり 0.9.3 は対象外)．
    1. メモリリークも zip\_close() だけならそれっぽいのを見つけられたが，そのまま修正して良いか注意深く見極める必要あり．

# PENDING #
  * sqlite 検討
    1. もっさり本棚を何とかしたいと思い，ちょっと分析を始める．
    1. 条件は「空の本棚を表示する時だけ」っぽいということがわかっているが，これが「アプリ実装」「QtSql」「SQLite」のどこに問題があるか見当をつけてみる．
    1. あれ，2.5.0 を入れたあたりから本棚もっさりが改善されたような．rcS の修正が効いてる? じゃ，検討止めるけど(Qt のコンパイルがメンドクサイ)．
```
echo -n 67108864 > /proc/sys/kernel/shmmax
```
  * uBoot の改善
    1. 色々なことをやる前に，まずは cache と interrupt 周りがどうなってるか調べる．とりあえず以下を有効にしたところ，電源 ON からカーネルスタートまでの時間が約 2.5sec -> 約 0.9sec と劇的な改善が．
```
#define CONFIG_ARCH_CPU_INIT
#define CONFIG_ARCH_MMU
```
    1. リリース用にコンパイルして気づいたのだが，github にあるコードのままだと同じものが作れない(たぶん外部 SD 起動できなくなる)．どうしよう，rakuten に現状のものを提供するよう電凸するか? でもリカバリはちゃんと動いたんだよな，コードの読み込みが足らないか? めんどくさいから L1 の I-cache だけ ON にしてごまかそうかなぁ....
    1. プロンプトの状態でほっとくと電源が勝手に切れてしまう問題を何とかしないと，好き勝手ができない．
    1. とりあえずヒントは MC13892 のデータシートにあった．こんなふうにプロセッサと協調して電源制御をする石があるのね．
    1. kobo には TI の MSP430 が載っていて，それの代わりをしているのではなかろうか．この辺りを頼りにカーネルソースでも見てみるか．
    1. usbtty を有効にしてみたい(殻割りせずに bootcmd を自由に変えたい)
    1. さらにホームボタン押下時だけは bootdelay=-1 で起動させてみたい
  * Linux カーネルの見直し
    1. とりあえず CONFIG\_ARM\_L1\_CACHE\_SHIFT=5 は Cortex-A8 的に間違っている
    1. ~~gadget ドライバも入れ替えて起動するところまでは確認できたものの，udev との相性が悪いのか USB プラグインに全く反応しない．困った．~~ 何回か OFF/ON してたら，突然正常動作するようになった．何だコリャ．
    1. ~~現状の uImage は zImage 由来の圧縮カーネルになっているが Image 由来の非圧縮カーネルにすると起動が早くなるか見極める~~ そんなに起動が遅くないのでヤメ．
    1. freescale 社が提供しているパッチが全部あたってない(関係ないものもあるはずだけど)
    1. 1GHz 動作用のパッチの中身を確認してみる(ENGR00180382-1: Add 1GHz working point support，ENGR00180382-2: Change SW1 operating mode for 1GHz WP あたり)
  * zlib neon 実装の watching
    1. オリジナルの adler32 neon 実装差分を見つけた． http://blackfin.uclinux.org/git/?p=users/vapier/zlib.git;a=summary
    1. メンテナンスが続いているように見受けられる https://github.com/CyanogenMod/android_external_zlib/tree/mr1.1-staging なるものを発見した．上は adler32 だけ neon 化した結果，下はさらに -DARM\_HAVE\_NEON をつけ inffast も neon 化した結果．slhash.c の neon 実装があるのは評価できるが deflate でしか使われない．
```
# time ./test.sh 7.gz
real    0m 32.62s
user    0m 15.99s
sys     0m 5.16s
```
```
# time ./test.sh 7.gz
real    0m 30.02s
user    0m 16.74s
sys     0m 4.40s
```
    1. こんなのも発見してしまった．http://www.opensource.apple.com/source/zlib/zlib-43/
  * udev 入れ替えの見極め
    1. SD や USB の抜き差しのトリガは udev が握っているので，これを入れ替えると何かが良くなるだろうか．
    1. 結局ゴニョゴニョして udev-175 をコンパイル．http://labs.beatcraft.com/ja/index.php?bc10-router%2Fbuildroot#o9d998ff にお世話になった．これ，動くんかいな．
    1. やっぱり正常動作しない，わはは．udev の中身を理解できていないうちは手を出さないことにする．

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

# おわりに #
本文書の最新版は http://code.google.com/p/kobohack-j/wiki/README で公開されているので，近況についてはこちらを参照されたい．