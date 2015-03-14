

間違いがあったら指摘してね．

# 6.4 デバイスの初期化 #
## 6.4.1 メモリマップ ##
図 6.4 は i.MX50 に対する OCRON と OCRAM のメモリマップを表している．

OCRAM の空き領域はアドレス 0xf800\_6000 から 0xf801\_7fff の合計 72KB に制限される．それに続く 24KB は，BT\_MMU\_DISABLEOCOTP ビットが切れて(ワンタイムプログラムヒューズによって有効になって)いなければ，MMU page table に予約される．BT\_MMU\_DISABLEOCOTP ビットが切れている時は，OCRAM の 0xf800\_6000 から 0xf801\_dfff の領域は空き領域とみなせるだろう．

# 6.5 ブートデバイス (内部ブート) #
## 6.5.3 拡張デバイスサポート ##
i.MX50のROM(ブートコードを格納した内蔵ROM)は、MMC/eMMCやSD/eSDに準拠したデバイスからの起動をサポートしている。

(中略)

MMC/eMMC/SD/eSDはeSDHC1-4モジュールのいずれにも接続でき、2KBのデータを内蔵RAMへコピーすることによって起動できる。ROMコード(ブートコード)は、ブートイメージ内のImage Vector Table header value (0xD1)をチェックした後にDCD(Device Configuration Data)のチェックを実行する。DCDの抽出に成功した後で、ブートイメージをコードの実行が発生する場所のRAMにコピーするために、コピー先のポインタ(アドレス)と長さ(大きさ)を抽出する。

> i.MX50内蔵のブートコードが、デバイス(例:SDカード)の先頭から2KBを内蔵RAMにコピーする。その中にはDCDとIVTが含まれていて、DCDに従いデバイスの初期設定を、IVTに従いコードの実行を開始するということ。あくまでソフト視点でブートのプロセスだけを読み解くならDCDを理解する必要はない。

# 6.6 プログラムイメージ #
## 6.6.1 Image Vector Table と Boot Data ##
Image Vector Table(IVT)は、ブートの完全な実行に必要なデバイスに格納された残りのデータコンポーネント(前出の2KBに収まらなかった、ブートに必要な残りの部分)をROMが読みだすためのデータ構造である。IVTはROMによって起動プロセス中に使用する(参照する)イメージのエントリーポイント(アドレス)、Device Configuration Data(DCD)へのポインタ、その他のポインタが含まれている。ROMはi.MX50に接続されたブートデバイスによって予め決められた固定のアドレスを用いてIVTを参照する。各ブートデバイスタイプの、IVTへのベースアドレスからのオフセット(アドレス)と、初期プログラムのイメージサイズを表6-22に示す。

> SDカードの場合は初期イメージサイズが2KBであるが、先頭の512byteはMBRであることが多いので、ユーザが自由に使えるのは後半1.5KBである。しかもIVTへのオフセットが1KBなので連続領域として使用できるのは後半の1KBだけ、ということになる。これがDCDの最大サイズに制限をかけている。ユーザの実行コードをここに置くことはないだろうと思うので、まず問題にはならない。

> これよりむしろ問題なのは、この仕組み(IVTがオフセット1KBの位置に固定されている)のためGUIDパーティションテーブル(いわゆるGPT)を使うことができないこと。でも、まぁ組み込み機器でGPT使うことなんてあまりないんだろう。

> kobo の IVT は u-boot のソースコードに以下のように記載されている(board/freescale/mx50\_rdp/flash\_header.S)．先頭のブランチ命令と CONFIG\_FLASH\_HEADER\_OFFSET によって先頭に 1KB の空きを作っている．リンカスクリプトが読める人は u-boot.lds も参照してみると，これがバイナリの先頭にくることが理解できるはず．
```
.section ".text.flasheader", "x"
        b       _start
        .org    CONFIG_FLASH_HEADER_OFFSET

/* First IVT to copy the plugin that initializes the system into OCRAM */
ivt_header:        .long 0x402000D1    /* Tag=0xD1, Len=0x0020, Ver=0x40 */
app_code_jump_v:   .long 0xF8006458    /* Plugin entry point */
reserv1:           .long 0x0
dcd_ptr:           .long 0x0
boot_data_ptr:     .long 0xF8006420
self_ptr:          .long 0xF8006400
app_code_csf:      .long 0x0           /* reserve 4K for csf */
reserv2:           .long 0x0
boot_data:         .long 0xF8006000
image_len:         .long 4*1024        /* Can copy upto 72K, OCRAM free space */
plugin:            .long 0x1           /* Enable plugin flag */

/* Second IVT to give entry point into the bootloader copied to DDR */
ivt2_header:       .long 0x402000D1    //Tag=0xD1, Len=0x0020, Ver=0x40
app2_code_jump_v:  .long _start   // Entry point for the bootloader
reserv3:           .long 0x0
dcd2_ptr:          .long 0x0
boot_data2_ptr:    .long boot_data2
self_ptr2:         .long ivt2_header
app_code_csf2:     .long 0x0 // reserve 4K for csf
reserv4:           .long 0x0
boot_data2:        .long TEXT_BASE
image_len2:        .long _end - TEXT_BASE
plugin2:           .long 0x0
```
> ひとつ目の IVT には，内蔵 ROM によって読み出されていない残りの flash\_header.S を内蔵 RAM に読み出し，その中に実装されているプラグインを実行するよう記載されている．

> 前出の通り内蔵 ROM は先頭の 2KB しか読み出さないのだが，flash\_header.S は 2KB を超えている(flash\_header.o を逆アセンブルしてみればわかるのだが最終アドレスは 0xa0f で 0x7ff を超えている)．チップの初期化コードを DCD を使わずに通常のアセンブラで実装しているものだからこういうことになっていると思われ，なんかダサい．ここらへん i.MX6(というか SABRE ボード) の実装を真似て書き直すとスッキリしそうな感じがする．

> アドレスが 0xf800\_0000 辺りから始まっているように書かれているのは，0xf800\_0000 から 0xf801\_ffff までの 128KB には OCRAM(On Chip RAM) が実装されているから．この RAM は外付けの DRAM と違って，あれこれ設定することなく電源 ON でそのまま使える．

> ふたつ目の IVT には u-boot 本体を読み出し，その中のエントリアドレスへジャンプするように記載されている．ひとつ目の IVT にあるプラグインの実行によって DRAM が使えるようになったから，ということである．エントリアドレスの物理アドレスって.... とリンカスクリプトを見てみると text セグメントは 0x0 から始まっているようだ．しかし DRAM の物理アドレスは 0x7000\_0000 から始まるので，前出の BT\_MMU\_DISABLEOCOTP によって MMU が有効になっており 0x0 に実行コードが置ける状態になっているのだろうと推測できる(ホントか怪しい)．