# kobo-update-3.0.1 (644485e229, 13/11/21) #
## Mark4 ##
  * またファイル名を変えずにアップデートしたらしい．デバイス情報では 3.0.1 と出るので，まだマシだが．
  * 3.0.0 からの差分を見てみた．
  * 恐らく libQtSql と libqsqlite.so が変わったせいで，リブートを繰り返すようになった．~~一旦 kobohack-j の公開を停止する．~~
    * sqlite のバージョンは変わっていないようであるが，いったい何があったんだよ．まったく．
```
2011-06-28 17:39:05 af0d91adf497f5f36ec3813f04235a6e195a605f
```
    * 当面の間 libqsqlite.so を抜いたものを公開する．さて rakuten に電凸だ．

|ファイル名|新規|更新|削除|備考|
|:--------------|:-----|:-----|:-----|:-----|
|`/usr/local/Kobo/fb2.xsl.gz`|－|◯|－|  |
|`/usr/local/Kobo/hindenburg`|－|◯|－|  |
|`/usr/local/Kobo/libboggle.so`|－|◯|－|  |
|`/usr/local/Kobo/libchess.so`|－|◯|－|  |
|`/usr/local/Kobo/libcrossword.so`|－|◯|－|  |
|`/usr/local/Kobo/libnickel.so.1.0.0`|－|◯|－|  |
|`/usr/local/Kobo/librushhour.so`|－|◯|－|  |
|`/usr/local/Kobo/libscribble.so`|－|◯|－|  |
|`/usr/local/Kobo/libsudoku.so`|－|◯|－|  |
|`/usr/local/Kobo/nickel`|－|◯|－|  |
|`/usr/local/Kobo/revinfo`|－|◯|－|  |
|`/usr/local/Kobo/stockfish`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtGui.so.4.6.2`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtSql.so.4.6.2`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtWebKit.so.4.6.2`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/plugins/sqldrivers/libqsqlite.so`|－|◯|－|  |

## Mark3 ##
|ファイル名|新規|更新|削除|備考|
|:--------------|:-----|:-----|:-----|:-----|
|`/usr/local/Kobo/fb2.xsl.gz`|－|◯|－|  |
|`/usr/local/Kobo/hindenburg`|－|◯|－|  |
|`/usr/local/Kobo/libchess.so`|－|◯|－|  |
|`/usr/local/Kobo/libcrossword.so`|－|◯|－|  |
|`/usr/local/Kobo/libnickel.so.1.0.0`|－|◯|－|  |
|`/usr/local/Kobo/librushhour.so`|－|◯|－|  |
|`/usr/local/Kobo/libscribble.so`|－|◯|－|  |
|`/usr/local/Kobo/libsudoku.so`|－|◯|－|  |
|`/usr/local/Kobo/nickel`|－|◯|－|  |
|`/usr/local/Kobo/revinfo`|－|◯|－|  |
|`/usr/local/Kobo/stockfish`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtGui.so.4.6.2`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtSql.so.4.6.2`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/lib/libQtWebKit.so.4.6.2`|－|◯|－|  |
|`/usr/local/Trolltech/QtEmbedded-4.6.2-arm/plugins/sqldrivers/libqsqlite.so`|－|◯|－|  |