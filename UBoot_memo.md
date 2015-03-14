自分向けのメモのつもり，読むな危険(ウソ)．

ちょっと U-Boot についていろいろ書いていく．

ついつい glo を買ってしまったのだが，これがなかなか値段差以上に使い心地がよい．touch を 1GHz 駆動にすれば同程度になりえるのか，という感じ．できる気がしないんだけど，まずはこれをテーマに．

# オリジナル U-Boot との差分 #
kobo 用として公開されているソースコードは，オリジナルに対して
  * Freescale 社による mx50\_rdp 向けのカスタマイズ
  * kobo 社による kobo ハードウエア向けのカスタマイズ

が，施されている．U-Boot をあれやこれやする前に，それぞれのカスタマイズ内容を明らかにしておくことが重要である(無用のフリーズを予防できる)．

現状の kobo 用公開ソースコードと Freescale 社の git リポジトリを比較してみたところ，どうやら以下の時点の U-Boot を起点に kobo 社のカスタマイズが入っているようである．

http://git.freescale.com/git/cgit.cgi/imx/uboot-imx.git/tree/?h=imx_v2009.08_11.04.01&id=rel_imx_2.6.35_11.03.00

imx\_v2009.08\_11.09.01 ブランチから i.MX6 の SABREAUTO 用のコードが入り始めたみたいであるが，その後のブランチでも i.MX5 向けの修正が入っているブランチがあるっぽい．12.02.01 あたりが最後だろうか．

2014 ベースの U-Boot にしたいのだけど，ここまで古いの使い続けられちゃうとちょっと無理かなぁとも思ってしまう．イマイチメリットが見いだせない，今でもさほど不便を感じてないし．

# mx50\_overclock #
大まかな処理の流れとしては....
  * 一度 C のコード実行を開始してから
  * クロックを変更するアセンブラコードを iram へコピーして
  * 割り込みを禁止状態にして
  * コピーした iram のアドレスにジャンプして
    * DRAM をセルフリフレッシュに入れて
    * PLL を変更し
    * DRAM を元の状態に戻して
  * C のコードに戻る
  * 割り込み禁止を解除

ということをやっているようである．

また，この処理を実行するのは HWConfig 内の CPUFreq が 0x02 の時で，ソースコードによると 0x01 が 800MHz とある．そんなわけで

  * HWConfig を v1.3 相当にアップデート(足りない部分はでっち上げる)

とやるだけで touch で mark4 用の U-Boot バイナリが動かせそうな気がしてきた．でもやっぱり flash\_header.S の差分が気になるなぁ．

# mark4 用の U-Boot バイナリ #
kobo-update-2.8.1 に含まれている U-Boot について．

  * u-boot\_mddr\_256-E606B0-K4X2G323PC.bin
  * u-boot\_mddr\_256-E50610-K4X2G323PC.bin
  * u-boot\_mddr\_256-E60610D-K4X2G323PC.bin
  * u-boot\_mddr\_512-E606C0-K4X2G323PC.bin

と 4 種類入っているかのように見えるが，上の 3 つは同一である．

# mark5 用の U-Boot バイナリ #
kobo-update-2.9.0 に含まれている U-Boot について．

  * u-boot\_mddr\_256-E606F0B-K4X2G323PC.bin

HWConfig をナンチャッテ v1.3 にした touch でも，特に不都合なく動いている．
ハックも済んでいて，外部 SD 起動時に LED が白色点灯するようになったのでわかりやすくなった．

例の箇所を 0x02 にして試してみたところ，これまで同様に安定動作はしなかった．わはは．

touch + mark5 用 U-Boot (HWConfig v1.3)
```
console=ttymxc0,115200 rootwait rw quiet lpj=3997696 root=/dev/mmcblk1p1 rootfstype=ext4 hwcfg_p=0x7ffffe00 hwcfg_sz=110 waveform_p=0x7fef3800 waveform_sz=1098951 mem=254M boot_port=0
```

# HWConfig #
v0.7(touch) と v1.3(glo) の差分は以下．

|イメージ上のアドレス| |touch (v0.7)|glo (v1.3)|touch (v1.3 相当値?)|
|:-----------------------------|:|:-----------|:---------|:----------------------|
|0x0008\_002d|RAMType|未定義|0x02 (K4X2G323PC)|0x02 (K4X2G323PC)|
|0x0008\_002e|UIConfig|未定義|0x00 (Normal)|0x00 (Normal)|
|0x0008\_002f|DisplayResolution|未定義|0x01 (1024x758)|0x00 (800x600)|
|0x0008\_0030|FrontLight|未定義|0x01 (TABLE0)|0x00 (No)|
|0x0008\_0031|CPUFreq|未定義|0x02 (1GHz)|0x01 (800MHz)|
|0x0008\_0032|HallSensor|未定義|0x01 (TLE4913)|0x00 (No)|

とりあえず上記の通り CPUFreq を 0x01 にして動作させてみたところ，通常通り使うことができた．よしよし．

0x02 にして動作させてみたところ，

  * LED が緑点灯でフリーズ
  * 画面表示は笑顔の kobo は表示されるがその下の流れる四角は表示されず
  * 笑顔の kobo の色がちょっと薄い

ということになった．1 バイト書き換えるだけで挙動が変わり，0x01 時の動作 0x02 時の動作がそれぞれ(ちゃんと動くかどうかは置いといて)安定しているので，ヨミはあたっているようである．

まてよ，カーネルや nickel も HWConfig 参照して動作しているんだよなぁ，ということは v1.3 に書き換えてしまうとカーネルも nickel も mark4 用を使わないとダメか? でも CPUFreq が 0x01 で動いたしなぁ....

カーネルまでは起動しているようなので，シリアル経由で調べてみることにしよう．


~~その前に外部 SD 起動ハッキングが先だ，うぅ．~~ ハッキング完了，無事起動．USB 経由の telnet 接続もできた．次から mark4 用も kobohack-j に入れよう．

結局変更した PLL が供給するクロックの範囲と実装に依存するんだよな．ほんとに CPU コアだけだったらシステムクロックの割り込み周期も変わらないけど，システムクロックの割り込み周期が変わっちゃったら CONFIG\_HZ の値通りに割り込みが上がってこないはず．

コードを読んでみると epdc の axi と pix が pll1 から供給される設定になっているようだ(ホントか?)．epdc って Electrophoretic display controller だから電気泳動ディスプレイってことで電子ペーパーのコントローラーだよね．色がちょっと薄いのはこのせい? ここクロックアップするのはいいのか? ディバイダちゃんと設定しなおして 800MHz 時と同じクロックにしないと.... やっぱカーネルは mark4 用使ったほうがいいのでは．

あ，ちなみに手持ちの touch で 1 台だけ v1.0 ってやつがあった．

# カーネルに渡す引数の違い #
loops\_per\_jiffy の値が異なる．

  * touch 系 3997696
  * glo 系 4997120

この値はカーネル起動時に計測して算出するようになっていて，例外としてカーネル引数で指定されている場合は計測せずに指定値で動作するようになっていのだったかと思う．

なので意図的に異なる値で動作させる必要がない限り，カーネル引数で渡す効果は起動時間の短縮だけじゃなかったかなーと．~~というわけで，lpj は未指定にしてみようかな．~~ CPUFreq を参照して値を変える実装があったので，勝手にそうなる．

touch 標準(root が mmcblk1p1 なのはゴニョゴニョしてるから....)
```
# cat /proc/cmdline
console=ttymxc0,115200 rootwait rw quiet lpj=3997696 root=/dev/mmcblk1p1 rootfstype=ext4 hwcfg_p=0x7ffffe00 hwcfg_sz=110 waveform_p=0x7fef3800 waveform_sz=1098951 mem=254M
```

glo 標準(教えてもらえた)
```
console=ttymxc0,115200 rootwait rw quiet lpj=4997120 root=/dev/mmcblk0p1 rootfstype=ext4 hwcfg_p=0x7ffffe00 hwcfg_sz=110 waveform_p=0x7feee200 waveform_sz=1121199 mem=254M mx50_1GHz 
```

やっぱり mx50\_1GHz がある．

さて 800MHz 設定のまま mark4 用の UBoot, カーネル，カーネルドライバを使って touch の起動を試してみたところ，特に何事も無く起動して，普通に本が読めた．うんうん，よしよし．ちゃんと usb gadget ドライバも動作する．

気を良くして，例の箇所を 0x01 から 0x02 に書き換えて実行させると....
  * 画面真っ黒でフリーズ
  * 緑 LED 点灯で笑顔の kobo が出て，さらに四角が流れている途中でフリーズ
  * その他いろいろ
と，最終的な現象が落ち着かず結果がバラバラになってしまう．0x01 に戻すと，何事もなかったように，普通に動作する．ハングアップ系らしく，usb gadget ドライバ経由の telnet も無理．ざーんねん．とりあえず /etc/init.d/rcS をあらかじめいじっておき，ほぼカーネル起動のみの状態で動作させてみるところからかな．

なんか，だんだん JTAG 欲しくなってきたかも(つなげらんないけど)，i.MX50 EVK ボードでデバッグした方が早かったりして．

続く(!?)．

```
Linux version 2.6.35.3-850-gbc67621+ (gallen@gallen-P5KPL-AM-BM) (gcc version 4.4.4 (4.4.4_09.06.2010) ) #617 PREEMPT Mon Apr 22 11:07:47 CST 2013
CPU: ARMv7 Processor [412fc085] revision 5 (ARMv7), cr=10c53c7f
CPU: VIPT nonaliasing data cache, VIPT nonaliasing instruction cache
Machine: Freescale MX50 Reference Design Platform
Memory policy: ECC disabled, Data cache writeback
On node 0 totalpages: 65024
free_area_init_node: node 0, pgdat 803b6e30, node_mem_map 803d7000
  DMA zone: 192 pages used for memmap
  DMA zone: 0 pages reserved
  DMA zone: 24384 pages, LIFO batch:3
  Normal zone: 316 pages used for memmap
  Normal zone: 40132 pages, LIFO batch:7
Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 64516
Kernel command line: console=ttymxc0,115200 rootwait rw quiet lpj=3997696 root=/dev/mmcblk1p1 rootfstype=ext4 hwcfg_p=0x7ffffe00 hwcfg_sz=110 aveform_p=0x7fef3800 waveform_sz=1098951 mem=254M
PID hash table entries: 1024 (order: 0, 4096 bytes)
Dentry cache hash table entries: 32768 (order: 5, 131072 bytes)
Inode-cache hash table entries: 16384 (order: 4, 65536 bytes)
Memory: 254MB = 254MB total
Memory: 253920k/253920k available, 6176k reserved, 0K highmem
Virtual kernel memory layout:
    vector  : 0xffff0000 - 0xffff1000   (   4 kB)
    fixmap  : 0xfff00000 - 0xfffe0000   ( 896 kB)
    DMA     : 0xf9e00000 - 0xffe00000   (  96 MB)
    vmalloc : 0x90000000 - 0xf4000000   (1600 MB)
    lowmem  : 0x80000000 - 0x8fe00000   ( 254 MB)
    pkmap   : 0x7fe00000 - 0x80000000   (   2 MB)
    modules : 0x7f000000 - 0x7fe00000   (  14 MB)
      .init : 0x80008000 - 0x80024000   ( 112 kB)
      .text : 0x80024000 - 0x8036a000   (3352 kB)
      .data : 0x8037e000 - 0x803b7b80   ( 231 kB)
SLUB: Genslabs=11, HWalign=32, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
Hierarchical RCU implementation.
      RCU-based detection of stalled CPUs is disabled.
      Verbose stalled-CPUs detection is disabled.
    :
```

さて，例の箇所を 0x02 に書き換えて nickel を起動させないようにして電源投入をしてみると，けっこう踏ん張る(なにそれ?)．

  * on-animator.sh によって 30 秒くらい四角が流れ
  * usb gadget 経由で telnet で入れるように

なることが多いが，結局あれこれやっているとハングアップしてしまう．

gloで使われているソフトをそのまま動かしているわけなので，ハードの差からくる現象のような気がしてならない．これをソフトでなんとかするのは，原因を調べるところから既に難しい．

「やがてダメになる」系なので RAM 周りっぽい気がするのだが，さてどうすっかなぁ.... 定数あたりかなぁ....

touch + mark5 用カーネル
```
Linux version 2.6.35.3-850-gbc67621+ (gallen@gallen-P5KPL-AM-BM) (gcc version 4.4.4 (4.4.4_09.06.2010) ) #2040 PREEMPT Thu Aug 22 13:13:27 CST 2013
CPU: ARMv7 Processor [412fc085] revision 5 (ARMv7), cr=10c53c7f
CPU: VIPT nonaliasing data cache, VIPT nonaliasing instruction cache
Machine: Freescale MX50 Reference Design Platform
Memory policy: ECC disabled, Data cache writeback
On node 0 totalpages: 65024
free_area_init_node: node 0, pgdat 803c3ac0, node_mem_map 803e5000
  DMA zone: 192 pages used for memmap
  DMA zone: 0 pages reserved
  DMA zone: 24384 pages, LIFO batch:3
  Normal zone: 316 pages used for memmap
  Normal zone: 40132 pages, LIFO batch:7
Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 64516
Kernel command line: console=ttymxc0,115200 rootwait rw quiet lpj=3997696 root=/dev/mmcblk1p1 rootfstype=ext4 hwcfg_p=0x7ffffe00 hwcfg_sz=110 waveform_p=0x7fef3800 waveform_sz=1098951 mem=254M boot_port=0
PID hash table entries: 1024 (order: 0, 4096 bytes)
Dentry cache hash table entries: 32768 (order: 5, 131072 bytes)
Inode-cache hash table entries: 16384 (order: 4, 65536 bytes)
Memory: 254MB = 254MB total
Memory: 253864k/253864k available, 6232k reserved, 0K highmem
Virtual kernel memory layout:
    vector  : 0xffff0000 - 0xffff1000   (   4 kB)
    fixmap  : 0xfff00000 - 0xfffe0000   ( 896 kB)
    DMA     : 0xf9e00000 - 0xffe00000   (  96 MB)
    vmalloc : 0x90000000 - 0xf4000000   (1600 MB)
    lowmem  : 0x80000000 - 0x8fe00000   ( 254 MB)
    pkmap   : 0x7fe00000 - 0x80000000   (   2 MB)
    modules : 0x7f000000 - 0x7fe00000   (  14 MB)
      .init : 0x80008000 - 0x80025000   ( 116 kB)
      .text : 0x80025000 - 0x80376000   (3396 kB)
      .data : 0x8038a000 - 0x803c4820   ( 235 kB)
SLUB: Genslabs=11, HWalign=32, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
Hierarchical RCU implementation.
      RCU-based detection of stalled CPUs is disabled.
      Verbose stalled-CPUs detection is disabled.
    :
```