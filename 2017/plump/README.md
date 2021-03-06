## iTunesのライブラリファイルを調べる(その1)
この記事は[関西Lispユーザ会アドベントカレンダー](https://adventar.org/calendars/2490)12日目です。

### はじめに
元々は「lispでSQL」というネタを考えていました。
SQLを使う以上、そこそこの数のデータを扱わないと面白くない思っていたので、
まずデータを用意するところから始めましたが、作業をしていると色々あったので、
データを作ることに絞った話に変更しました。

手元にあるそこそこの数のデータとしては、
iTunesの音楽情報(CD200枚分くらい？)があります。
iTunesではその情報を一つのXMLファイルとして保存しています。
XMLファイルを解析してデータを読み解いていく様子を説明していきます。


### iTunes Library.xml
iTunes Library.xmlはXMLファイルですが、(macOSで従来から使われていた)plistを
XMLフォーマットにした感が強く、XMLをパーズするだけでは、その後のプログラムで使えるようなデータ構造が得るのは難しようです。そのため、iTunes Library.xml読み込み用のプロ
グラムがいくつか公開されています。
なお、[iTunes ライブラリファイルについて](https://support.apple.com/ja-jp/HT201610)
に書かれているように、iTunes 12.2 以降では、
「iTunes Library.xml」ファイルは作成されませんので、設定を変更することになります。
私の場合は、古いバージョンから使っていたせいかiTunes Library.xmlは
作成されていました。

私のiTunes Library.xml(ファイルの頭から一曲分、Locationのみ編集)は以下となっています。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.
com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>Major Version</key><integer>1</integer>
        <key>Minor Version</key><integer>1</integer>
        <key>Application Version</key><string>12.7.1.14</string>
        <key>Date</key><date>2017-11-26T08:51:50Z</date>
        <key>Features</key><integer>5</integer>
        <key>Show Content Ratings</key><true/>
        <key>Library Persistent ID</key><string>B982A8C26EB13FBC</string>
        <key>Tracks</key>
        <dict>
                <key>899</key>
                <dict>
                        <key>Track ID</key><integer>899</integer>
                        <key>Size</key><integer>9284778</integer>
                        <key>Total Time</key><integer>458579</integer>
                        <key>Disc Number</key><integer>1</integer>
                        <key>Disc Count</key><integer>1</integer>
                        <key>Track Number</key><integer>1</integer>
                        <key>Track Count</key><integer>8</integer>
                        <key>Year</key><integer>1986</integer>
                        <key>Date Modified</key><date>2009-11-08T00:36:07Z</date>
                        <key>Date Added</key><date>2009-09-12T00:27:50Z</date>
                        <key>Bit Rate</key><integer>160</integer>
                        <key>Sample Rate</key><integer>44100</integer>
                        <key>Play Count</key><integer>7</integer>
                        <key>Play Date</key><integer>3545667249</integer>
                        <key>Play Date UTC</key><date>2016-05-09T10:34:09Z</date>
                        <key>Skip Count</key><integer>4</integer>
                        <key>Skip Date</key><date>2014-11-10T13:29:06Z</date>
                        <key>Artwork Count</key><integer>1</integer>
                        <key>Persistent ID</key><string>98030EB29E1BA041</string>
                        <key>Track Type</key><string>File</string>
                        <key>File Folder Count</key><integer>5</integer>
                        <key>Library Folder Count</key><integer>1</integer>
                        <key>Name</key><string>Tango Zebra</string>
                        <key>Artist</key><string>Adrian Belew</string>
                        <key>Composer</key><string>Adrian Belew</string>
                        <key>Album</key><string>Desire Caught By The Tail</string>
                        <key>Genre</key><string>ロック</string>
                        <key>Kind</key><string>MPEG オーディオファイル</string>
                        <key>Location</key><string>file:///Users/...</string>
                </dict>

```

行数などは次のようになります。そこそこの大きさと言えるのではないでしょうか。

```
$ wc iTunes\ Music\ Library.xml 
215884  384497 9327870 iTunes Music Library.xml
$
```

### plump
XMLを扱うライブラリをインストールしていなかったので、quickdocでXMLを検索
しました。ダウンロードが多いplumpが無難そうです。

quicklispでインストールは何の問題もなく終了。さっそく読み込んでみます。


```
CL-USER> (time
          (setq node
                (plump:parse (format nil "~{~a~^ ~}"
                                     (with-open-file (st "iTunes Music Library.xml")
                                       (loop for line = (read-line st nil)
                                          while line collect line))))))
(SETQ NODE (PLUMP-PARSER:PARSE (FORMAT NIL "~{~a~^ ~}" (WITH-OPEN-FILE (ST "iTunes Music Library.xml") (LOOP FOR LINE = (READ-LINE ST NIL) WHILE LINE COLLECT LINE)))))
took 78,965,428 microseconds (78.965420 seconds) to run.
     39,196,153 microseconds (39.196150 seconds, 49.64%) of which was spent in GC.
During that period, and with 4 available CPU cores,
     77,536,000 microseconds (77.536000 seconds) were spent in user mode
      1,120,000 microseconds ( 1.120000 seconds) were spent in system mode
 1,007,046,246 bytes of memory allocated.
 142,860 minor page faults, 0 major page faults, 0 swaps.
#<PLUMP-DOM:ROOT #x302000CA3BFD>
```

`(format nil "~{~a~^ ~}"`は少々乱暴な方法とは思ったものの読み込むことができました。


##### 注意
sbclでは`Heap exhausted, game over.`が出ました。上の結果はcclで実行したものです。
環境により、sbclでも動くとは思いますが。

### パーズ結果
```
CL-USER> (length (plump:children node))
5
CL-USER> (loop for n across (plump:children node)
            do
              (format t "~a~%" n))
#<XML-HEADER version 1.0>
#<TEXT-NODE #x302000CA2F7D>
#<DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
#<TEXT-NODE #x302000CA2A5D>
#<ELEMENT plist #x302000CA255D>
NIL
CL-USER> (plump:text (aref (plump:children node) 1))
" "
CL-USER> (plump:text (aref (plump:children node) 3))
" "
CL-USER> (length (plump:children (aref (plump:children node) 4)))
3
```

パーズの結果、iTunes Library.xmlは次の5つのオブジェクトとなります。

1. xml-header。
2. text(空白のみ)。
3. doctype。
4. text(空白のみ)。
5. element。

最初の4つは実質的にデータはありません。5番目の`#<ELEMENT plist #x302000CA255D>`に20万行強のデータの全てが入っているということです。これ(mainとする)を調べてみます。

```
CL-USER> (setq main (aref (plump:children node) 4))
#<ELEMENT plist #x302000CA255D>
CL-USER> (length (plump:children main))
3
CL-USER> (loop for n across (plump:children main)
            do
              (format t "~a~%" n))
#<TEXT-NODE #x302000CA235D>
#<ELEMENT dict #x302000CA1E3D>
#<TEXT-NODE #x30201F60C7BD>
NIL
CL-USER> (plump:text (aref (plump:children main) 0))
" "
CL-USER> (plump:text (aref (plump:children main) 2))
" "
```

mainは三つのオブジェクトからなり、二つ目の`#<ELEMENT dict #x302000CA1E3D>`の中に
20万行強のデータの全てが入っていることが判ります。mainの子要素を調べてみます。

```
CL-USER> (aref (plump:children main) 1)
#<ELEMENT dict #x302000CA1E3D>
CL-USER> (length (plump:children (aref (plump:children main) 1)))
33
```

これくらいないなら、全て出力しても問題なしです。

```
CL-USER> (loop for n across (plump:children (aref (plump:children main) 1))
            do
              (format t "~a~%" n))
#<TEXT-NODE #x302000CA1DBD>
#<ELEMENT key #x302000CA193D>
#<ELEMENT integer #x302000CA13FD>
#<TEXT-NODE #x302000CA132D>
#<ELEMENT key #x302000CA0EAD>
#<ELEMENT integer #x302000CA096D>
#<TEXT-NODE #x302000CA089D>
#<ELEMENT key #x302000CA041D>
#<ELEMENT string #x302000C9FECD>
#<TEXT-NODE #x302000C9FDDD>
#<ELEMENT key #x302000C9F95D>
#<ELEMENT date #x302000C9F44D>
#<TEXT-NODE #x302000C9F32D>
#<ELEMENT key #x302000C9EEAD>
#<ELEMENT integer #x302000C9E97D>
#<TEXT-NODE #x302000C9E8AD>
#<ELEMENT key #x302000C9E42D>
#<ELEMENT true #x302000C9DDBD>
#<TEXT-NODE #x302000C9DD6D>
#<ELEMENT key #x302000C9D8ED>
#<ELEMENT string #x302000C9D39D>
#<TEXT-NODE #x302000C9D28D>
#<ELEMENT key #x302000C9CE0D>
#<TEXT-NODE #x302000C9CD2D>
#<ELEMENT dict #x302000C9C8AD>
#<TEXT-NODE #x302019B7053D>
#<ELEMENT key #x302019B700BD>
#<TEXT-NODE #x302019B6FFCD>
#<ELEMENT array #x302019B6FB4D>
#<TEXT-NODE #x30201F60D55D>
#<ELEMENT key #x30201F60D0DD>
#<ELEMENT string #x30201F60CBAD>
#<TEXT-NODE #x30201F60CA1D>
NIL
```

これまでのtext-nodeは全て空白のみでしたので、ここも空白のみの可能性が高いです。

```
#<ELEMENT plist #x302000C7E79D>
CL-USER> (remove-duplicates (loop for n across (plump:children main)
                               if (plump:text-node-p n)
                               collect (string-trim '(#\Space #\Tab) (plump:text n)))
                            :test #'equal)

("")
```

やはり空白のみでした。text-nodeは無視して問題なしです。text-node以外を出力します。

```
CL-USER> (loop for n across (plump:children (aref (plump:children main) 1))
            unless (plump:text-node-p n)
            do (format t "~a~%" n))
#<ELEMENT key #x302000CA193D>
#<ELEMENT integer #x302000CA13FD>
#<ELEMENT key #x302000CA0EAD>
#<ELEMENT integer #x302000CA096D>
#<ELEMENT key #x302000CA041D>
#<ELEMENT string #x302000C9FECD>
#<ELEMENT key #x302000C9F95D>
#<ELEMENT date #x302000C9F44D>
#<ELEMENT key #x302000C9EEAD>
#<ELEMENT integer #x302000C9E97D>
#<ELEMENT key #x302000C9E42D>
#<ELEMENT true #x302000C9DDBD>
#<ELEMENT key #x302000C9D8ED>
#<ELEMENT string #x302000C9D39D>
#<ELEMENT key #x302000C9CE0D>
#<ELEMENT dict #x302000C9C8AD>
#<ELEMENT key #x302019B700BD>
#<ELEMENT array #x302019B6FB4D>
#<ELEMENT key #x30201F60D0DD>
#<ELEMENT string #x30201F60CBAD>
NIL
```

全てelementです。tag-nameがkeyであるオブジェクトの子要素を出力すると、

```
CL-USER> (loop for n across (plump:children (aref (plump:children main) 1))
            if (and (plump:element-p n) 
                    (string= (plump:tag-name n) "key"))
            do
              (format t "~a~%" (plump:text (aref (plump:children n) 0))))
Major Version
Minor Version
Application Version
Date
Features
Show Content Ratings
Library Persistent ID
Tracks
Playlists
Music Folder
NIL
```

となり、XMLの冒頭との対応から、

```
#<ELEMENT key #x302000C9CE0D>     → Tracks
#<ELEMENT dict #x302000C9C8AD>
#<ELEMENT key #x302019B700BD>     → Playlists
#<ELEMENT array #x302019B6FB4D>
#<ELEMENT key #x30201F60D0DD>     → Music Folder
#<ELEMENT string #x30201F60CBAD>
```

となっていて、Tracksの次のオブジェクト`#<ELEMENT dict #x302000C9C8AD>`の中に
曲の情報が入っていそうです。

```
CL-USER> (setq tracks
               (loop for n across (plump:children (aref (plump:children main) 1))
                  if (and (plump:element-p n) 
                          (string= (plump:tag-name n) "dict"))
                  do
                    (return n)))
#<ELEMENT dict #x302000C9C8AD>
CL-USER> (length (plump:children tracks))
20557
```

まとまった数のデータがあります。やっと曲の情報にたどりつきました。

### 曲の情報を調べる
二万を超えるものを全部出力しているとシャレにならないので、
どういうオブジェクトがあるのかを調べてから出力します。

```
CL-USER> (remove-duplicates (loop for n across (plump:children tracks)
                               collect (class-name (class-of n))))
(PLUMP-DOM:ELEMENT PLUMP-DOM:TEXT-NODE)
CL-USER> (remove-duplicates (loop for n across (plump:children tracks)
                               if (plump:text-node-p n)
                               collect (string-trim '(#\Space #\Tab) (plump:text n)))
                            :test #'equal)
("")
CL-USER> (remove-duplicates (loop for n across (plump:children tracks)
                               if (plump:element-p n)
                               collect (plump:tag-name n))
                            :test #'equal)

("key" "dict")
CL-USER> (loop for n across (plump:children tracks)
            if (plump:element-p n)
              if (string= (plump:tag-name n) "key")
                count n into key-num
              else
                count n into dict-num
            finally
              (format t "key: ~d~%dict: ~d~%" key-num dict-num))
key: 5139
dict: 5139
NIL
```

text-nodeはここでも空白のみなので、無視してよいということが判ります。
elementはkeyとdictの二つのみで、個数が同じです。
XMLファイルの冒頭の例(一曲目)からkeyにはTrack IDが入り、曲名などは
dictの中にあると思われます。そして、keyの数(=dictの数)である5139が
iTunesに入っている曲数ということなります。
次は、一曲目をもう少し詳しく見てみます。

### 一曲目を詳しく

```
CL-USER> (setq sample
               (loop for n across (plump:children tracks)
                  if (and (plump:element-p n) 
                          (string= (plump:tag-name n) "dict"))
                  do
                    (return n)))
#<ELEMENT dict #x302000C7821D>
CL-USER> (remove-duplicates (loop for n across (plump:children sample)
                               collect (class-name (class-of n))))
(PLUMP-DOM:ELEMENT PLUMP-DOM:TEXT-NODE)

CL-USER> (remove-duplicates (loop for n across (plump:children sample)
                               if (plump:text-node-p n)
                               collect (string-trim '(#\Space #\Tab) (plump:text n)))
                            :test #'equal)
("")
```

今までと同じく、text-nodeは無視してよいことが判ります。

```
CL-USER> (remove-duplicates (loop for n across (plump:children sample)
                               if (plump:element-p n)
                               collect (plump:tag-name n))
                            :test #'equal)
("date" "integer" "key" "string")
CL-USER> (loop for n across (plump:children sample)
            if (plump:element-p n) 
              if (string= (plump:tag-name n) "date")
                count n into date-num
              else if (string= (plump:tag-name n) "integer")
                count n into integer-num
              else if (string= (plump:tag-name n) "key")
                count n into key-num
              else if (string= (plump:tag-name n) "string")
                count n into string-num
            finally
              (format t "date: ~d~%integer: ~d~%key: ~d~%string: ~d~%"
                      date-num integer-num key-num string-num))
date: 4
integer: 16
key: 29
string: 9
NIL
```

elementのtag-nameはdate、integer、key、stringの4つとなっています。

keyの個数(29) = dateの個数(4) + integerの個数(16) + stringの個数(9)

が成り立つので、keyが情報の種類を示していて、
date/integer/stringがそれぞれの値を示していると考えられます。
keyの中身を知るためには、さらの子要素を見ていきます。

```
CL-USER> (remove-duplicates (loop for n across (plump:children sample)
                               if (and (plump:element-p n)
                                       (string= (plump:tag-name n) "key"))
                               collect (length (plump:children n))))
(1)
CL-USER> (remove-duplicates (loop for n across (plump:children sample)
                               if (and (plump:element-p n)
                                       (string= (plump:tag-name n) "key"))
                               collect (class-name (class-of (aref (plump:children n) 0)))))
(PLUMP-DOM:TEXT-NODE)
```

子要素は一つだけで、全てtext-nodeです。tag-nameがinteger、string、dateの場合も
同様です。

次に二曲目以降(sample以外)も同じ構造なっているか確認します。

### 二曲目以降の確認
同じようなコードが出てくるので、関数を準備します。

```
(defun get-child (node func)
  "nodeの子要素でfuncがtとなるもののリストを返す"
  (loop for n across (plump:children node)
     if (funcall func n) collect n))

(defun length-check (node)
  "node 子要素、かつelementであるものについて、その子要素の数が1であるか調べる"
  (loop for n in (get-child node #'plump:element-p)
     always (= (length (plump:children n)) 1)))
```

それでは、全曲の情報について一曲目と同じ構造なのか調べます。

```
CL-USER> (loop for child in (get-child tracks
                                          #'(lambda (x)
                                              (and (plump:element-p x) 
                                                   (string= (plump:tag-name x)
                                                            "dict"))))
            always (length-check child))
NIL
```

残念ながら全曲が同じ構造ではありませんでした。
一曲目では、elementである子要素の子要素の数は全て1でしたが、
そうでない場合があります。では、子要素の数は全て1である場合とそうでない場合がどれくらいあるかを調べてみます。

```
CL-USER> (loop for child in (get-child tracks
                                       #'(lambda (x)
                                              (and (plump:element-p x) 
                                                   (string= (plump:tag-name x)
                                                            "dict"))))
            if (length-check child)
              count child into is-one
            else
              count child into is-not-one
            finally
              (format t "1: ~d~%not 1: ~d~%" is-one is-not-one))
1: 4955
not 1: 184
```

大半は成立ちますが、例外がいくつかあるようです。成立たない場合のtag-nameを調べてみます。

```
CL-USER> (remove-duplicates 
          (loop for child in (get-child tracks
                                        #'(lambda (x)
                                            (and (plump:element-p x) 
                                                 (string= (plump:tag-name x)
                                                          "dict"))))
             unless (length-check child)
             append (loop for n in (get-child child #'plump:element-p)
                       unless (= (length (plump:children n)) 1)
                       collect (list (plump:tag-name n) (length (plump:children n)))))
          :test #'equal)
(("true" 0))
```

tag-nameがtrueの場合は子要素の数は0となるようです。この場合は、tag-name
を子要素の値とすればよいでしょう。
このことを考慮して子要素の値を取出す関数を作ります。

```
(defun get-child-value (node)
  (let* ((tag (plump:tag-name node))
         (value (if (> (length (plump:children node)) 0)
                    (plump:text (aref (plump:children node) 0))
                    tag)))
    (if (string= tag "integer")
        (parse-integer value)
        value)))
```

全曲に対して実行してもエラーとならないことを確かめます。

```
CL-USER> (loop for child in (get-child tracks
                                       #'(lambda (x)
                                           (and (plump:element-p x) 
                                                (string= (plump:tag-name x)
                                                         "dict"))))
            always (get-child-value child))
T
```

ここまでできると曲の情報を、keyをキー(日本語なら何とかOKか)に、keyに続くelement
を値とするハッシュ表にすることをできます。一度ハッシュ表にしておけば、一番長い曲、
一番よく聞いた曲などの「検索」っぽいこともできるようになります。

### ハッシュ表
ハッシュ表を作るための関数`node->hash`を準備します。


```
(defun node->hash (node)
  "一曲分のオブジェクトを引数に取り、それをハッシュ表として返す"
  (loop for (k v) on (loop for n across (plump:children node)
                        if (plump:element-p n) collect n)
     by #'cddr
     with hash = (make-hash-table :test #'equal)
     do
       (setf (gethash (get-child-value k) hash)
             (get-child-value v))
     finally
       (return hash)))
```

sampleで動作を確認します。

```
CL-USER> (setq h (node->hash sample))
#<HASH-TABLE :TEST EQUAL size 29/60 #x30201F69D05D>
CL-USER> (gethash "Artist" h)
"Adrian Belew"
T
CL-USER> (gethash "Album" h)
"Desire Caught By The Tail"
T
CL-USER> (gethash "Name" h)
"Tango Zebra"
```

問題なさそうです。では、全曲について動かしてみます。

```
CL-USER> (length 
          (setq track-list
                (loop for n in (get-child tracks
                                          #'(lambda (x)
                                              (and (plump:element-p x) 
                                                   (string= (plump:tag-name x) "dict"))))
                     collect (node->hash n))))
5139
```

一番長い曲の時間と一番よく聞いた曲の回数は以下のように求めることができます。

```
CL-USER> (time (loop for track in track-list
                  if (gethash "Total Time" track)
                  maximize it))
(LOOP FOR TRACK IN TRACK-LIST IF (GETHASH "Total Time" TRACK) MAXIMIZE IT)
took 10,546 microseconds (0.010546 seconds) to run.
During that period, and with 4 available CPU cores,
     12,000 microseconds (0.012000 seconds) were spent in user mode
          0 microseconds (0.000000 seconds) were spent in system mode
10856232
CL-USER> (time (loop for track in track-list
                  if (gethash "Play Count" track)
                  maximize it))
(LOOP FOR TRACK IN TRACK-LIST IF (GETHASH "Play Count" TRACK) MAXIMIZE IT)
took 8,178 microseconds (0.008178 seconds) to run.
During that period, and with 4 available CPU cores,
     8,000 microseconds (0.008000 seconds) were spent in user mode
         0 microseconds (0.000000 seconds) were spent in system mode
940
```

5000強のデータの中から簡単に抽出することができます
(実はどちらも普通の曲と言えるものではないのですが)。


長くなったので、今日はここまでにします。明日はtrack-listをいろいろと調べます。
