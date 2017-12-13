## iTunesのライブラリファイルを調べる(その2)
この記事は[関西Lispユーザ会アドベントカレンダー](https://adventar.org/calendars/2490)13日目です。12日目の続きになります。

昨日はiTunes Library.xmlを読み込み、ハッシュ表のリストtrack-listを
作りました。今日は、track-listを調べます。

```
CL-USER> (length track-list)
5139
```

### 変なものを取り除く
iTunesはCDを読み込ませたものがほとんどですが、MP3音源や動画で取込んでしまった
ものもあります。CDを読み込んだら、設定されると思われるName(曲名)、
Album(アルバム名)、Artist(アーティスト名)、Total Time(曲の長さ)
が設定されているか調べてみます。

```
CL-USER> (loop for track in track-list
	    if (gethash "Name" track)
	      count track into name
	    if (gethash "Album" track)
	      count track into album
	    if (gethash "Artist" track)
	      count track into artist
	    if (gethash "Total Time" track)
	      count track into total-time
	    finally
	      (format t "Name: ~d~%Album: ~d~%Aritist: ~d~%Total Time: ~d~%"
		      name album artist total-time))
Name: 5137
Album: 5121
Aritist: 5121
Total Time: 5134
NIL
```

潔くName、Album、Artist、Total Timeがあるものに限定します。

```
5118
CL-USER> (length (setq track-list
		       (loop for track in track-list
			  if (and (gethash "Name" track)
				  (gethash "Album" track)
				  (gethash "Artist" track)
				  (gethash "Total Time" track))
			  collect track)))
5118
```

行き当たりばったりですが、一番長いなど目に付くものを見てみます。

#### 一番長い曲
一番長い曲の時間は次になります(12日目参照)。

```
CL-USER> (loop for track in track-list
	    if (gethash "Total Time" track)
	    maximize it)
10856232
```

ここでTotal Timeの単位はミリ秒です。

```
CL-USER> (truncate (truncate (/ 10856232 1000)) 60)
180
56
```

180秒です。3時間です、長過ぎます。確認するとこのトラックは某ラジオ番組の録音でした。
これをtrack-listから取除くと一番長い曲は、

```
CL-USER> (length (setq track-list
		       (loop for track in track-list
			  unless (= (gethash "Total Time" track) 10856232)
			  collect track)))
5117
CL-USER> (loop for track in track-list
	    if (gethash "Total Time" track)
	    maximize it)
3605054
CL-USER> (truncate (truncate 3605054 1000) 60)
60
5
```

となりました。60分でも長いと思われる方もいるかと思いますが、

```
3605054
CL-USER> (loop for track in track-list
	    if (= (gethash "Total Time" track) 3605054)
	    do
	      (format t "Artist: ~a~%Album: ~a~%Name: ~a~%"
		      (gethash "Artist" track)
		      (gethash "Album" track)
		      (gethash "Name" track)))
Artist: Mike Oldfield
Album: Amarok
Name: Amarok
NIL
```

Artist、AlbumからちゃんとしたCDであることが確認できます。
気になった方は、https://en.wikipedia.org/wiki/Amarok_(Mike_Oldfield_album) を参照してください。

#### 一番よく聞いた曲
昨日書いたように、

```
CL-USER> (loop for track in track-list
	    if (gethash "Play Count" track)
	    maximize it)
940
```

が最多ですが、このトラックはギター教則本の付録CDに入っている曲でした
もういくつかギター教則本のCDがあったので、これらを取り除くと、

```
CL-USER> (length track-list)
4947
```

となりました。一番聞いている曲の回数は、

```
CL-USER> (loop for track in track-list
	    if (gethash "Play Count" track)
	    maximize it)
53
```

こんなものかも知れません。

```
CL-USER> (loop for track in track-list
	    if (and (gethash "Play Count" track)
		    (= (gethash "Play Count" track) 53))
	    count track)
1
```

一曲だけのようです。

```
CL-USER> (loop for track in track-list
	    if (and (gethash "Play Count" track)
		    (= (gethash "Play Count" track) 53))
	    do
	      (format t "Album: ~a~%Name: ~a~%Artist: ~a~%" 
		      (gethash "Album" track)
		      (gethash "Name" track) (gethash "Artist" track)))
		      
Album: REBECCA IV ～Maybe tomorrow～
Name: Maybe Tomorrow
Artist: REBECCA
NIL
```

なるほどー。

### 曲を調べる
#### 最初にiTunesに登録した曲
一曲目の情報からDate Addedを見ていけば、最初に登録した曲が判るかと思い、
日付フォーマットからuniversal-timeに変換する関数を作って、Date Addedの
最小値を求めます。

```
(defun encode-utime (date-string)
  (ppcre:register-groups-bind ((#'parse-integer yyyy mm dd h m s))
      ("(\\d{4})-(\\d{2})-(\\d{2})T(\\d{2}):(\\d{2}):(\\d{2})Z"
       date-string)
    (encode-universal-time s m h dd mm yyyy)))
```

```
CL-USER> (loop for track in track-list
	    minimize (encode-utime (gethash "Date Added" track)))
3461671670
CL-USER> (decode-universal-time 3461671670)
50
27
0
12
9
2009
5
NIL
-9
```

2009年9月12日が一番古いということになります。が、もっと前からiTunesを使っています。
念のため、Date Modifiedの最小値を調べてみます。

```
CL-USER> (loop for track in track-list
	    minimize (encode-utime (gethash "Date Modified" track)))

3285704755
CL-USER> (decode-universal-time 3285704755)
55
45
8
14
2
2004
5
NIL
-9
```

2004年2月14日が一番古いとなります。まだこちらの方がしっくりきます。
しかし、一番最初にiTunesに登録した曲は判らないようです。

#### 一度も再生していない曲
Play Countに値が設定されるのは、iTuneあるいはiPodで再生してからです。
Play Countが設定されていないということはiTunesあるいはiPodで再生してない
ということになります。

```
CL-USER> (loop for track in track-list
	    if (gethash "Play Count" track)
	      count track into played
	    else
	      count track into not-played
	    finally
	      (format t "played: ~d~%not played: ~d~%" played not-played))
played: 3063
not played: 1884
CL-USER> (format t "~f" (/ 1884 (+ 3063 1884)))
0.38083687
NIL

```

曲数なら、全体の38%が(iTunes/iPodでは)再生していないということのようです。
通常のCDプレーヤで聞いているとは思うのですけど。

曲の時間ならどうなるでしょうか。

```
CL-USER> (loop for track in track-list
	    if (gethash "Play Count" track)
	      summing (gethash "Total Time" track) into played
	    else
	      summing (gethash "Total Time" track) into not-played
	    finally
	      (format t "played: ~:d~%not played: ~:d~%" played not-played))
played: 1,054,510,368
not played: 608,264,915
NIL
CL-USER> (format t "~f" (/ 608264915 (+ 1050303533 608264915)))
0.3667409
NIL
```

曲の長さでも同じようなものです。うーん…

#### 全部足すと
何の意味もないですが、全曲を聞くのにどれくらいかかるか調べてみます。

```
CL-USER> (loop for track in track-list
	    summing (gethash "Total Time" track))
1658568448
CL-USER> (truncate (truncate (/ 1658568448 1000)) (* 3600 24))
19
16968
CL-USER> (truncate (truncate (/ 1658568448 1000)) 3600)
460
2568

```

460時間と少々。一日中聞き続けたして、20日弱というところです。

### アルバムを調べる
#### アルバム数など
```
CL-USER> (length 
	  (remove-duplicates (loop for track in track-list
				collect (gethash "Album" track))
			     :test #'equal))
540
CL-USER> (length 
	  (remove-duplicates (loop for track in track-list
				collect (gethash "Artist" track))
			     :test #'equal))
235
```

アルバムは540枚(2枚組は2枚とカウント)、アーティストは235組
(表記のゆれでダブルカウントはあります)
という結果です。こんなに持っていたっけなあ。200枚はあったとは思っていましたが。

#### アルバムを聞いた回数
曲を何回聞いたかはPlay Countから判りますが、アルバムを聞いた回数
はどうでしょうか。アルバムを聞く、ということをどう定義するかにもよりますが、
同一アルバムの曲でPlay Countが最小となる回数をアルバムを聞いた回数と
定義することにしましょう。ランダムシャッフルで聞くよりもアルバムは頭から
通して聞くことが多いので、この定義はそんなに外れてはいないと思います。

```
CL-USER> (setq album-hash 
	       (loop with album = (make-hash-table :test #'equal)
		  for track in track-list
		  for album-name = (gethash "Album" track)
		  do 
		    (push track (gethash album-name album nil))
		  finally
		    (return album)))
#<HASH-TABLE :TEST EQUAL size 540/687 #x30201E79740D>
CL-USER> (loop for album being the hash-value in album-hash
	    maximize (reduce #'min album 
			     :key #'(lambda (x)
				      (or (gethash "Play Count" x) 0))))

34
```

34回が最も多いようです。アルバムは何でしょうか。

```
CL-USER> (loop for album being the hash-value in album-hash using (hash-key k)
	    if (= (reduce #'min album 
			  :key #'(lambda (x)
				   (or (gethash "Play Count" x) 0)))
		  34)
	    collect k)
("Systematic Chaos")
CL-USER> (gethash "Artist" (first (gethash "Systematic Chaos" album-hash)))
"Dream Theater"
T
```

Dream TheaterのSystematic Chaosでした。Dream Theaterは好きなバンドの一つで
アルバムもほとんど全部持っていますが、Systematic Chaosとは少々意外でした。

### おわりに
プログラミングは楽しいです。その理由の一つは、動かした結果が見える(分かる)こと
ではないかと思っています。入門書にあるHello Worldも動かした結果ではありますが、
何度も実行したいと思えるほどの魅力はありません。一方、動かした結果が予想から外れた
ものになると、次はどうなるだろう、次もやってみたいとなるのではないでしょうか。
今回のiTunes Library.xmlを読み込むのは、そういう意味で楽しいものでした。

明日は、dbym4820さんのCommon Lispでシステム構築する際のパス名取得について
です。

