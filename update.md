# kobo-update-3.0.0 #
  * mark 3/4 用に出ている。既に URI も判明している。

# kobo-update-2.10.0 (6180f632e1, 13/10/21) #
  * 2.8.1b からの差分を見てみた．
  * 「同期/お知らせ」なる機能が追加された．
  * 噂通り Pocket 機能が追加された．
  * 「体験版アプリ」に「脱出ゲーム」なるものが追加された．
  * /etc/init.d/rcS にファクトリーリセットの仕掛けが組み込まれた，これは u-boot からファクトリーリセットの実装を追い出す布石だろうか．
  * unrar が arm バイナリに治ったようである(相変わらず 3.71 beta1)．
  * touch に入れてみたが特に問題ないようなので，以後はこれで動作確認を行う．
    * 確かに「脱出ゲーム」が出てこない．

|ファイル名|新規|更新|削除|備考|
|:--------------|:-----|:-----|:-----|:-----|
|`/etc/images/dragon-factory-reset.raw.gz`|◯|－|－|  |
|`/etc/images/dragon-reboot.raw.gz`|◯|－|－|  |
|`/etc/images/dragon-update-spinner-0-reset.raw.gz`|◯|－|－|  |
|`/etc/images/dragon-update-spinner-1-reset.raw.gz`|◯|－|－|  |
|`/etc/images/dragon-update-spinner-2-reset.raw.gz`|◯|－|－|  |
|`/etc/images/dragon-update-spinner-3-reset.raw.gz`|◯|－|－|  |
|`/etc/images/dragon-update-spinner-4-reset.raw.gz`|◯|－|－|  |
|`/etc/images/dragon-update-spinner-5-reset.raw.gz`|◯|－|－|  |
|`/etc/images/factory-reset.raw.gz`|◯|－|－|  |
|`/etc/images/kraken-factory-reset.raw.gz`|◯|－|－|  |
|`/etc/images/kraken-reboot.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-update-spinner-0.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-update-spinner-1.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-update-spinner-2.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-update-spinner-3.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-update-spinner-4.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-update-spinner-5.raw.gz`|－|◯|－|  |
|`/etc/images/phoenix-factory-reset.raw.gz`|◯|－|－|  |
|`/etc/images/pixie-factory-reset.raw.gz`|◯|－|－|  |
|`/etc/images/pixie-reboot.raw.gz`|－|◯|－|  |
|`/etc/images/pixie-update-spinner-0.raw.gz`|－|◯|－|  |
|`/etc/images/pixie-update-spinner-1.raw.gz`|－|◯|－|  |
|`/etc/images/pixie-update-spinner-2.raw.gz`|－|◯|－|  |
|`/etc/images/pixie-update-spinner-3.raw.gz`|－|◯|－|  |
|`/etc/images/pixie-update-spinner-4.raw.gz`|－|◯|－|  |
|`/etc/images/pixie-update-spinner-5.raw.gz`|－|◯|－|  |
|`/etc/images/reboot.raw.gz`|－|◯|－|  |
|`/etc/images/update-spinner-0.raw.gz`|－|◯|－|  |
|`/etc/images/update-spinner-1.raw.gz`|－|◯|－|  |
|`/etc/images/update-spinner-2.raw.gz`|－|◯|－|  |
|`/etc/images/update-spinner-3.raw.gz`|－|◯|－|  |
|`/etc/images/update-spinner-4.raw.gz`|－|◯|－|  |
|`/etc/images/update-spinner-5.raw.gz`|－|◯|－|  |
|`/etc/init.d/rcS`|－|◯|－|  |
|`/usr/local/Kobo/adobehost`|－|◯|－|  |
|`/usr/local/Kobo/fb2.xsl.gz`|－|◯|－|  |
|`/usr/local/Kobo/fickel`|－|◯|－|  |
|`/usr/local/Kobo/hindenburg`|－|◯|－|  |
|`/usr/local/Kobo/libadobe.so`|－|◯|－|  |
|`/usr/local/Kobo/libboggle.so`|－|◯|－|  |
|`/usr/local/Kobo/libcb.so`|－|◯|－|  |
|`/usr/local/Kobo/libchess.so`|－|◯|－|  |
|`/usr/local/Kobo/libcrossword.so`|－|◯|－|  |
|`/usr/local/Kobo/libiwnn.so.1.0.0`|－|◯|－|  |
|`/usr/local/Kobo/liblzma.so.5.0.4`|－|◯|－|  |
|`/usr/local/Kobo/libnickel.so.1.0.0`|－|◯|－|  |
|`/usr/local/Kobo/libpdb.so`|－|◯|－|  |
|`/usr/local/Kobo/libQtSolutions_IOCompressor-2.3.so.1.0.0`|－|◯|－|  |
|`/usr/local/Kobo/librushhour.so`|◯|－|－|  |
|`/usr/local/Kobo/libscribble.so`|－|◯|－|  |
|`/usr/local/Kobo/libsolitaire.so`|－|◯|－|  |
|`/usr/local/Kobo/libsudoku.so`|－|◯|－|  |
|`/usr/local/Kobo/libwikipedia.so`|－|◯|－|  |
|`/usr/local/Kobo/nickel`|－|◯|－|  |
|`/usr/local/Kobo/pickel`|－|◯|－|  |
|`/usr/local/Kobo/plugins/libaccessplugin.so`|－|◯|－|  |
|`/usr/local/Kobo/puz`|－|◯|－|  |
|`/usr/local/Kobo/revinfo`|－|◯|－|  |
|`/usr/local/Kobo/stockfish`|－|◯|－|  |
|`/usr/local/Kobo/udev/usb`|－|◯|－|  |
|`/usr/local/Kobo/unrar`|－|◯|－|  |
|`/usr/local/Kobo/zim`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Amasis-Bold.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Amasis-BoldItalic.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Amasis-Italic.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Amasis.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Avenir-Italic.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Avenir.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Caecilia-Bold.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Caecilia-BoldItalic.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Caecilia-Italic.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Caecilia.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Dyslexie-Bold.ttf`|◯|－|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Dyslexie-BoldItalic.ttf`|◯|－|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Dyslexie-Italic.ttf`|◯|－|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Dyslexie.ttf`|◯|－|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/georgia.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/georgiab.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/georgiai.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/georgiaz.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/GillSans-Italic.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/GillSans.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Malabar-Bold.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Malabar-BoldItalic.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Malabar-Italic.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/fonts/Malabar.ttf`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtCore.so.4.6.2`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtGui.so.4.6.2`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtNetwork.so.4.6.2`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtWebKit.so.4.6.2`|－|◯|－|  |
|`/usr/share/hyphen/hyph_nl_NL.dic`|◯|－|－|  |

# kobo-update-2.9.0 (042df55f04, 13/09/15) #
  * URI 見つけた，探せばふつーに見つかる．
  * URI の一部も kobo5 になってるから，現状では aura 6" のみなのだろうな．
  * uboot も カーネルも mark4 とは別物になってるが，ちょっと touch に入れてみようかな...
    * touch に入れてみたがフツーに動く．
    * 「本棚」ではなくて「コレクション」になっているが，名前が変わっただけのような．
    * 設定した時間になっても自動でスリープしなくなってしまったような気がする．
    * 見慣れないカーネルスレッドがあるが，これまでのバージョンでもあったっけ?
```
    :
  284 root       0:04 [EPDC Submit]
  286 root       0:00 [EPDC Interrupt]
    :
```

ページめくりながら top で見てると，それなりにCPUを使っているようだが，こんなのなかったと思われ．
```
Mem: 154488K used, 99492K free, 0K shrd, 2084K buff, 82472K cached
CPU: 17.6% usr  7.7% sys  0.0% nic 74.5% idle  0.0% io  0.0% irq  0.0% sirq
Load average: 0.54 0.20 0.12 2/54 914
  PID  PPID USER     STAT   VSZ %VSZ CPU %CPU COMMAND
  522     1 root     D     194m 78.4   0 18.4 /usr/local/Kobo/nickel -qws -skipF
  284     2 root     SW       0  0.0   0  5.4 [EPDC Submit]
    4     2 root     RW       0  0.0   0  0.7 [events/0]
  914   877 root     R     2248  0.8   0  0.5 top
  876   526 root     S     2536  1.0   0  0.1 {busybox} telnetd -i
```


# kobo-update-2.8.1b (b2c05cda0e, 13/09/20) #
  * またバージョン文字列を変えずに中身を変えてくるという暴挙にでたようだ．
  * /usr/local/Kobo/unrar が Intel 80386 用のオブジェクトに見えるのだけど，気のせい? cbr ファイル参照できなくね?
```
% file usr/local/Kobo/unrar
usr/local/Kobo/unrar: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs)....
```
    * この地雷を踏んだ人現る? http://www.mobileread.com/forums/showthread.php?t=224090
    * 2.6.2 でもこの症状がでているらしいので確認したら，確かにそうなってた．
    * しょうがないから電凸してみた．
|ファイル名|新規|更新|削除|備考|
|:--------------|:-----|:-----|:-----|:-----|
|`/usr/local/Kobo/adobehost`|－|◯|－|  |
|`/usr/local/Kobo/fb2.xsl.gz`|－|◯|－|  |
|`/usr/local/Kobo/fickel`|－|◯|－|  |
|`/usr/local/Kobo/hindenburg`|－|◯|－|  |
|`/usr/local/Kobo/hyphenDicts/hyph_nl.dic`|－|◯|－|  |
|`/usr/local/Kobo/libadobe.so`|－|◯|－|  |
|`/usr/local/Kobo/libboggle.so`|－|◯|－|  |
|`/usr/local/Kobo/libcb.so`|－|◯|－|  |
|`/usr/local/Kobo/libchess.so`|－|◯|－|  |
|`/usr/local/Kobo/libcrossword.so`|－|◯|－|  |
|`/usr/local/Kobo/libiwnn.so.1.0.0`|－|◯|－|  |
|`/usr/local/Kobo/liblzma.so.5.0.4`|－|◯|－|  |
|`/usr/local/Kobo/libnickel.so.1.0.0`|－|◯|－|  |
|`/usr/local/Kobo/libpdb.so`|－|◯|－|  |
|`/usr/local/Kobo/libQtSolutions_IOCompressor-2.3.so.1.0.0`|－|◯|－|  |
|`/usr/local/Kobo/libscribble.so`|－|◯|－|  |
|`/usr/local/Kobo/libsolitaire.so`|－|◯|－|  |
|`/usr/local/Kobo/libsudoku.so`|－|◯|－|  |
|`/usr/local/Kobo/libwikipedia.so`|－|◯|－|  |
|`/usr/local/Kobo/nickel`|－|◯|－|  |
|`/usr/local/Kobo/pickel`|－|◯|－|  |
|`/usr/local/Kobo/plugins/libaccessplugin.so`|－|◯|－|  |
|`/usr/local/Kobo/puz`|－|◯|－|  |
|`/usr/local/Kobo/revinfo`|－|◯|－|  |
|`/usr/local/Kobo/stockfish.so`|－|◯|－|  |
|`/usr/local/Kobo/unrar`|－|◯|－|  |
|`/usr/local/Kobo/zim`|－|◯|－|  |

# kobo-update-2.8.1 (9a63806202, 13/07/15) #
  * Mark 4 用しか出ないの? Mark 3 ハードウエアにもインストールした(できた)と報告はあるようだけど http://www.mobileread.com/forums/showthread.php?t=185660
  * ここからは Mark 4 用の差分
|ファイル名|新規|更新|削除|備考|
|:--------------|:-----|:-----|:-----|:-----|
|`/etc/images/kraken-ghostbuster.raw.gz`|◯|－|－|  |
|`/etc/images/kraken-on-0.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-on-1.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-on-10.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-on-2.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-on-3.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-on-4.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-on-5.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-on-7.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-on-8.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-on-9.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-on-0.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-reboot.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-update-spinner-0.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-update-spinner-1.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-update-spinner-2.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-update-spinner-3.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-update-spinner-4.raw.gz`|－|◯|－|  |
|`/etc/images/kraken-update-spinner-5.raw.gz`|－|◯|－|  |
|`/etc/init.d/rcS`|－|◯|－|  |
|`/etc/init.d/uprade-wifi.sh`|－|◯|－|  |
|`/usr/lib/libfreetype.so.6.6.2`|－|◯|－|  |
|`/usr/local/Kobo/libadobe.so`|－|◯|－|  |
|`/usr/local/Kobo/libchess.so`|－|◯|－|  |
|`/usr/local/Kobo/libcrossword.so`|－|◯|－|  |
|`/usr/local/Kobo/libnickel.so.1.0.0`|－|◯|－|  |
|`/usr/local/Kobo/libpdb.so`|－|◯|－|  |
|`/usr/local/Kobo/libscribble.so`|－|◯|－|  |
|`/usr/local/Kobo/libsolitaire.so`|－|◯|－|  |
|`/usr/local/Kobo/libsudoku.so`|－|◯|－|  |
|`/usr/local/Kobo/libwikipedia.so`|－|◯|－|  |
|`/usr/local/Kobo/nickel`|－|◯|－|  |
|`/usr/local/Kobo/revinfo`|－|◯|－|  |
|`/usr/local/Kobo/udev/sd`|－|◯|－|  |
|`/usr/local/Kobo/udev/usb`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtCore.so.4.6.2`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtGui.so.4.6.2`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtWebKit.so.4.6.2`|－|◯|－|  |

# kobo-update-2.6.2 #
  * 今度は Mark 3 用が出ないからスルー

# kobo-update-2.6.1 #
  * Mark 4 用のファームウエアが同じバージョンナンバーで差し替えリリースされているらしい(わたしはかんけーない)。
|ファイル名|新規|更新|削除|備考|
|:--------------|:-----|:-----|:-----|:-----|
|`/usr/local/Kobo/fb2.xsl.gz`|－|○|－|  |
|`/usr/local/Kobo/hindenburg`|－|○|－|  |
|`/usr/local/Kobo/libnickel.so.1.0.0`|－|○|－|  |
|`/usr/local/Kobo/libpdb.so`|－|○|－|  |
|`/usr/local/Kobo/libQtSolutions_IOCompressor-2.3.so.1.0.0`|－|○|－|  |
|`/usr/local/Kobo/libscribble.so`|－|○|－|  |
|`/usr/local/Kobo/libsolitaire.so`|－|○|－|  |
|`/usr/local/Kobo/libsudoku.so`|－|○|－|  |
|`/usr/local/Kobo/libwikipedia.so`|－|○|－|  |
|`/usr/local/Kobo/nickel`|－|○|－|  |
|`/usr/local/Kobo/revinfo`|－|○|－|  |
|`/usr/local/Kobo/stockfish`|－|○|－|  |

# kobo-update-2.6.0 #
  * 書きだすのがメンドクサイくらい差分あり
    * 突然 ABI を soft(softfp) から hard に変えてきた(互換性が無くなるから普通は途中でこんなことしない、クレイジー以外の何物でもない)
    * toolchain のコンフィグレーションなのか、デフォルトで arm 命令と thumb 命令のチャンポンを出すようになった(こっちは互換性を保てるからよいが、バイナリサイズと実行パフォーマンスのバランスを評価したほうが良い)
  * 改良点をモロパクリされてる感じ(オリジナリティは皆無だからいいんだけど)

  1. busybox が 1.22.0.kobo になっているようだ
  1. libpng が 1.6.2 になっているようだ
  1. libjpeg が jpeg-turbo-1.2.90 になっているようだ
  1. libz が 1.2.7.1 になっているようだ(しかも neon 化された adler32.c が入っているようだ)
  1. openssl が 1.0.1e になっているようだ

  * ~~これ，このまま使えばいいんちゃう?~~ ~~真面目に neon 化してないっぽいからダメだな，なんてハンパな仕事を....~~ 実害は無くなったかな?
    * いわゆる「ダメ文字」は ~~再現させてないからわからないし~~ 治っていないらしい(バージョン上げときゃいいのに dosfstools は 3.0.6 が入ってる)
  * ソース公開してないバージョンがあるから、また rakuten に電凸するか?

# kobo-update-2.5.2 #
|ファイル名|新規|更新|削除|備考|
|:--------------|:-----|:-----|:-----|:-----|
|`/usr/local/Kobo/adobehost`|－|○|－|  |
|`/usr/local/Kobo/libadobe.so`|－|○|－|  |
|`/usr/local/Kobo/libboggle.so`|－|○|－|  |
|`/usr/local/Kobo/libchess.so`|－|○|－|  |
|`/usr/local/Kobo/libcrossword.so`|－|○|－|  |
|`/usr/local/Kobo/libnickel.so.1.0.0`|－|○|－|  |
|`/usr/local/Kobo/libpdb.so`|－|○|－|  |
|`/usr/local/Kobo/libscribble.so`|－|○|－|  |
|`/usr/local/Kobo/libsudoku.so`|－|○|－|  |
|`/usr/local/Kobo/libwikipedia.so`|－|○|－|  |
|`/usr/local/Kobo/nickel`|－|○|－|  |
|`/usr/local/Kobo/revinfo`|－|○|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtGui.so.4.6.2`|－|○|－|  |

# kobo-update-2.5.1 #
|ファイル名|新規|更新|削除|備考|
|:--------------|:-----|:-----|:-----|:-----|
|`/usr/local/Kobo/libnickel.so.1.0.0`|－|○|－|  |
|`/usr/local/Kobo/nickel`|－|○|－|  |
|`/usr/local/Kobo/revinfo`|－|○|－|  |

# kobo-update-2.5.0 #
  1. 読みやすいレイアウトでEPUBを読むことができます
  1. ストアが使いやすくなりました．
  1. バグを修正しました．
|ファイル名|新規|更新|削除|備考|
|:--------------|:-----|:-----|:-----|:-----|
|`/etc/init.d/rcS`|－|○|－|  |
|`/etc/init.d/upgrade-wifi.sh`|－|○|－|  |
|`/usr/local/Kobo/adobehost`|－|○|－|  |
|`/usr/local/Kobo/crypto/libqca-ossl.so`|－|－|○|  |
|`/usr/local/Kobo/hindenburg`|－|○|－|  |
|`/usr/local/Kobo/libadobe.so`|－|○|－|  |
|`/usr/local/Kobo/libboggle.so`|－|○|－|  |
|`/usr/local/Kobo/libcb.so`|－|○|－|  |
|`/usr/local/Kobo/libchess.so`|－|○|－|  |
|`/usr/local/Kobo/libcrossword.so`|－|○|－|  |
|`/usr/local/Kobo/libnickel.so.1.0.0`|－|○|－|  |
|`/usr/local/Kobo/libpdb.so`|－|○|－|  |
|`/usr/local/Kobo/libqca.so`|－|－|○|  |
|`/usr/local/Kobo/libscribble.so`|－|○|－|  |
|`/usr/local/Kobo/libsolitaire.so`|○|－|－|  |
|`/usr/local/Kobo/libsudoku.so`|－|○|－|  |
|`/usr/local/Kobo/libwikipedia.so`|－|○|－|  |
|`/usr/local/Kobo/nickel`|－|○|－|  |
|`/usr/local/Kobo/revinfo`|－|○|－|  |
|`/usr/local/Kobo/stockfish`|－|○|－|  |
|`/usr/local/Kobo/udev/usb`|－|○|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtCore.so.4.6.2`|－|○|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtGui.so.4.6.2`|－|○|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtNetwork.so.4.6.2`|－|○|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtWebKit.so.4.6.2`|－|○|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtXml.so.4.6.2`|－|○|－|  |

# kobo-update-2.4.0(feb13) #
  1. 本を**端末から削除**して容量を空けることができます．端末から削除した本は後で再度ダウンロードできます．
  1. 本をライブラリから**完全に削除**することもできます．
|ファイル名|新規|更新|備考|
|:--------------|:-----|:-----|:-----|
|`/usr/local/Kobo/libicel.so.1.0.0`|－|○|  |
|`/usr/local/Kobo/nickel`|－|○|  |
|`/usr/local/Kobo/revinfo`|－|○|  |

# kobo-update-2.4.0 #
|ファイル名|新規|更新|備考|
|:--------------|:-----|:-----|:-----|
|`/bin/kobo_config.sh`|－|○|  |
|`/bin/ntx_hwconfig`|－|○|  |
|`/usr/local/Kobo/crypto/libqca-ossl.so`|－|○|  |
|`/usr/local/Kobo/hindenburg`|－|○|  |
|`/usr/local/Kobo/libadobe.so`|－|○|  |
|`/usr/local/Kobo/libboggle.so`|－|○|  |
|`/usr/local/Kobo/libcb.so`|－|○|  |
|`/usr/local/Kobo/libchess.so`|－|○|  |
|`/usr/local/Kobo/libcrossword.so`|－|○|  |
|`/usr/local/Kobo/liblzma.so`|○|－|xzutils|
|`/usr/local/Kobo/liblzma.so.5`|○|－|↑|
|`/usr/local/Kobo/liblzma.so.5.0.4`|○|－|↑|
|`/usr/local/Kobo/libscribble.so`|－|○|  |
|`/usr/local/Kobo/libsudoku.so`|－|○|  |
|`/usr/local/Kobo/libwikipedia.so`|○|－|ario?|
|`/usr/local/Kobo/nickel`|－|○|  |
|`/usr/local/Kobo/pickel`|－|○|  |
|`/usr/local/Kobo/revifo`|－|○|  |
|`/usr/local/Kobo/stockfish`|－|○|  |
|`/usr/local/Kobo/zim`|○|－|zim format?|
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtCore.so.4.6.2`|－|○|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtGui.so.4.6.2`|－|○|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtWebKit.so.4.6.2`|－|○|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/plugins/imageformats/libqtiff.so`|－|○|  |

# kobo-update-2.3.2 #
|ファイル名|新規|更新|備考|
|:--------------|:-----|:-----|:-----|
|`/bin/ntx_hwconfig`|－|○|  |
|`/usr/local/Kobo/adobehost`|－|○|  |
|`/usr/local/Kobo/hindenburg`|－|○|  |
|`/usr/local/Kobo/libadobe.so`|－|○|  |
|`/usr/local/Kobo/libboggle.so`|－|○|  |
|`/usr/local/Kobo/libcb.so`|－|○|  |
|`/usr/local/Kobo/libchess.so`|－|○|  |
|`/usr/local/Kobo/libcrossword.so`|－|○|  |
|`/usr/local/Kobo/libnickel.so.1.0.0`|－|○|  |
|`/usr/local/Kobo/libqdb.so`|－|○|  |
|`/usr/local/Kobo/libqca.so`|－|○|  |
|`/usr/local/Kobo/libscribble.so`|－|○|  |
|`/usr/local/Kobo/libsudoku.so`|－|○|  |
|`/usr/local/Kobo/nickel`|－|○|  |
|`/usr/local/Kobo/revinfo`|－|○|  |
|`/usr/local/Kobo/stockfish`|－|○|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtCore.so.4.6.2`|－|○|  |

# kobo-update-2.3.1 #
|ファイル名|新規|更新|備考|
|:--------------|:-----|:-----|:-----|
|`/bin/ntx_hwconfig`|－|○|  |
|`/etc/init.d/rcS`|－|○|  |
|`/etc/init.d/upgrade-wifi.sh`|－|○|  |
|`/usr/local/Kobo/adobehost`|○|－|  |
|`/usr/local/Kobo/adobehost-launcher.sh`|○|－|  |
|`/usr/local/Kobo/hindenburg`|－|○|  |
|`/usr/local/Kobo/libadobe.so`|－|○|  |
|`/usr/local/Kobo/libboggle.so`|－|○|  |
|`/usr/local/Kobo/libcb.so`|－|○|  |
|`/usr/local/Kobo/libchess.so`|－|○|  |
|`/usr/local/Kobo/libcrossword.so`|－|○|  |
|`/usr/local/Kobo/libnickel.so.1.0.0`|－|○|  |
|`/usr/local/Kobo/libqdb.so`|－|○|  |
|`/usr/local/Kobo/libqca.so`|－|○|  |
|`/usr/local/Kobo/libscribble.so`|－|○|  |
|`/usr/local/Kobo/libsudoku.so`|－|○|  |
|`/usr/local/Kobo/nickel`|－|○|  |
|`/usr/local/Kobo/revinfo`|－|○|  |
|`/usr/local/Kobo/stockfish`|－|○|  |
|`/usr/local/Kobo/udev/usb`|－|○|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtCore.so.4.6.2`|－|○|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtGui.so.4.6.2`|－|○|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtSvg.so.4.6.2`|－|○|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtWebKit.so.4.6.2`|－|○|  |
|`/usr/share/dbus-1/services/com.kobo.adobe.Parser.service`|○|－|  |
|`/usr/share/dbus-1/services/com.kobo.adobe.Reader.service`|○|－|  |
|`/usr/share/dbus-1/services/com.kobo.adobe.Search.service`|○|－|  |