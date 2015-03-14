Mark4 用に 3.8.0 が出たので，普通にそっちを使いましょう(Mark3 用も出たのはうれしい)．

# Mark4 と Mark5 のソフトの差について雑多なメモ #
Mark5 向けに 3.7.0 がリリースされているが，これ Mark4 にも使えるのか調べたことをつらつらと．ちなみに Touch(N905B) に入れてしまった人もいる．http://blog.livedoor.jp/seikoku_/archives/52082352.html

結論としては Mark4 では動くと思う，Mark3 ではいつ動かなくなってもおかしくないから要注意，って感じだろうか．

3.5.0 は Mark4 と Mark5 の両方でリリースされているが，KoboRoot.tgz の差分が
  * /mnt/onboard/.kobo/slideshow 以下の画像ファイルの有無 (Mark4 にあるけど Mark5 には無い)
  * nickel 本体

のみである(nickel の差分が気になるが，突っ込んで調べる元気はない)．

Mark5 の 3.5.0 と 3.7.0 の `/etc/init.d/rcS` の差分を見ると...

3.5.0
```
	case $PRODUCT in
		kraken|phoenix)
			export COORDINATES="80 870 70 70 200 870 70 70";;
		dragon)
			export COORDINATES="120 1220 100 100 280 1220 100 100";;
		*)
			export COORDINATES="55 685 60 60 150 685 60 60";;
		
	esac
```

3.7.0
```
	case $PRODUCT in
		kraken|phoenix)
			export COORDINATES="80 870 70 70 200 870 70 70";;
		dragon|dahlia)
			export COORDINATES="120 1220 100 100 280 1220 100 100";;
		*)
			export COORDINATES="55 685 60 60 150 685 60 60";;
		
	esac
```

と `dahlia` 依存の実装が追加されていて，`kraken` まで考慮されている．800x600 機も **それ以外** という実装でなんとか助かっている，という感じか．

動作確認機(N905B)に 3.7.0 を入れてみたところ，問題なさそうなので kobohack-j の配布は継続する．読書機への導入は保留で．

以下，蛇足．

殻割りが難しくなり，内部ストレージが microSD でなくなったので積極的に購入する理由は全くない(glo で十分満足している)．H2O はどうなのだろう?

~~某所の情報が早かったことはこれまでもあったけど，いまだに海外のフォーラムや calibre 方面で 3.7.0 の話題が出ないのが気持ち悪い．~~ やとでた．