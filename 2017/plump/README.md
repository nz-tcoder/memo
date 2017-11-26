## iTunesのライブラリファイルを調べる

### はじめに
アドベンドカレンダーのネタとして、「replでSQL」というものを
考えていましたが、SQLを使うならそこそこの数のデータがないと面白く
ないので、まずデータを作るところから始めました。

自分の手元にあるそこそこの数のデータとしては、
iTunesに掘り込んだCD(200くらい？)があります。
iTunesではその情報がXMLファイルに保存されるので、
XMLファイルを解析してデータベースに突っ込んでいけば、
簡単にデータを作ることできるはず。
なのですが、意外といろいろあったので、これをまずネタにしました。

### iTunes Library.xml
 iTunes Library.xmlはXMLファイルですが、(macOSで従来から使われていた)plistを
XMLフォーマットにした感が強く、XMLをパーズするだけで(後続で使える)データ構造
が得るのは難しそうです。

なお、[iTunes ライブラリファイルについて](https://support.apple.com/ja-jp/HT201610)
に書かれているように、iTunes 12.2 以降では、
「iTunes Library.xml」ファイルは作成されませんので、設定を変更することになります。
私の場合は、古いバージョンから使っていたせいかiTunes Library.xmlは
作成されていました。

私のiTunes Library.xml(ファイルの頭から一曲分のレコード)は以下のようになります。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.
com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>Major Version</key><integer>1</integer>
        <key>Minor Version</key><integer>1</integer>
        <key>Application Version</key><string>12.6.1.25</string>
        <key>Date</key><date>2017-07-01T23:30:20Z</date>
        <key>Features</key><integer>5</integer>
        <key>Show Content Ratings</key><true/>
        <key>Library Persistent ID</key><string>B982A8C26EB13FBC</string>
        <key>Tracks</key>
        <dict>
                <key>898</key>
                <dict>
                        <key>Track ID</key><integer>898</integer>
                        <key>Size</key><integer>9284778</integer>
                        <key>Total Time</key><integer>458579</integer>
                        <key>Disc Number</key><integer>1</integer>
                        <key>Disc Count</key><integer>1</integer>
                        <key>Track Number</key><integer>1</integer>
                        <key>Track Count</key><integer>8</integer>
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.
com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>Major Version</key><integer>1</integer>
        <key>Minor Version</key><integer>1</integer>
        <key>Application Version</key><string>12.6.1.25</string>
        <key>Date</key><date>2017-07-01T23:30:20Z</date>
        <key>Features</key><integer>5</integer>
        <key>Show Content Ratings</key><true/>
        <key>Library Persistent ID</key><string>B982A8C26EB13FBC</string>
        <key>Tracks</key>
        <dict>
                <key>898</key>
                <dict>
                        <key>Track ID</key><integer>898</integer>
                        <key>Size</key><integer>9284778</integer>
                        <key>Total Time</key><integer>458579</integer>
                        <key>Disc Number</key><integer>1</integer>
                        <key>Disc Count</key><integer>1</integer>
                        <key>Track Number</key><integer>1</integer>
                        <key>Track Count</key><integer>8</integer>
                       <key>Year</key><integer>1986</integer>
                        <key>Date Modified</key><date>2009-11-08T00:36:07Z</date
>
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

### plump

### 試し読み

### 全件読む


