## lispだじゃれ

この記事は[関西Lispユーザ会アドベントカレンダー](https://adventar.org/calendars/2490)8日目です。

2日目に続き、だじゃれネタです。[LispギャグAdvent Calendar](https://atnd.org/events/22826)の感化を受けています。

#### lispらしさ
質問者: 「lispを象徴する関数は何ですか？」  
lisper: 「symobl-function やな」

---

#### CLOSのスローガン
「継承を使うように設計しよう!」

---

#### クロージャ
lisper: 「クロージャは変数を閉じ込めることができます。」

```
CL-USER> (setq inc 
	       (let ((counter 0))
		 (lambda () (incf counter))))
```

聞き手: 「へー」

lisper: 「そして、閉じ込めた変数にアクセスすることもできます。」


```

CL-USER> (funcall inc)
1
CL-USER> (funcall inc)
2
```

聞き手: 「ほー」

例は[Let Over Lambda](https://letoverlambda.com/)から。

---

#### 羊料理の店にて
店員:「当店のおすすめは"マトンのステーキ"と"ラム肉の赤ワイン煮"です。」  
liper: 「僕はラムだ。」

---

#### 実体験
「引数を高階関数にしたら、デバッグトレースを見た時に後悔したことがある。」

---

#### liperの野望
「マクロをネタに色んなカンファレンスに出まくろう。」

---

#### loopに転向したcommon lisperのつぶやき
「最近、再帰を使っていない。」

---

#### キャンプでの調理方法
「キャンプで食材の調理方法に困った時、unixerは"焼けば食える"の法則に
従って何でも焼き、lisperは"煮れば食える"の法則に従って何でも鍋に放り込む。
main-framerはその時、ただ涙がこぼれるだけらしい。」

---

#### 無題
```
(defun leopordp (arg)
  (eval arg))
```

---

お粗末。