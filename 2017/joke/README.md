## lispだじゃれ

この記事は[関西Lispユーザ会アドベントカレンダー](https://adventar.org/calendars/2490)8日目です。

2日目に続き、だじゃれネタです。[LispギャグAdvent Calendar](https://atnd.org/events/22826)に感化を受けています。

#### lispらしさ
質問者: 「lispを象徴する関数は何ですか？」
lisper: 「それは symobl-function やがな」

#### CLOSのスローガン
継承を使うように設計しよう!

#### クロージャ
lisper: 「クロージャは変数を閉じ込めることができます。」

```
(let ((counter 0))
  (lambda () (incf counter)))
```

例は[Let Over Lambda](https://letoverlambda.com/)から。