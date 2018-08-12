---
title: "翁長雄志、津川雅彦、両氏の死についてのツイートを Google Cloud Natural Language API にぶち込んでみた"
date: 2018-08-12 17:01:00 +0900
---

[翁長雄志、津川雅彦、両氏の死についてのツイートを分析し、左右どちらのほうがクズなのか計量的に分析してみました - 雑記（主に政治や時事について）][original]

上記の記事を読んで面白いと思ったのだけど、

> 最終的な選り分けは結局主観に頼るものだったので偏っている可能性がある

という記載が気になって、試しに [Google Cloud Natural Language API][api] に
ぶち込んだら客観的な分析になるのでは？…と思って試してみた。

## ソースコード

ほぼ、Google Cloud Natural Language API のサンプルどおり。
CSV を読み込んで API に食わせて、分析結果と原文を別の CSV に保存する感じにしてみた。

```python
#!/usr/bin/python3

import sys
import csv
from google.cloud import language
from google.cloud.language import enums
from google.cloud.language import types

client = language.LanguageServiceClient()

out_file = open(sys.argv[2], 'w')
writer = csv.writer(out_file, lineterminator='\n')

with open(sys.argv[1], 'r') as in_file:
    reader = csv.reader(in_file)

    for row in reader:
        document = types.Document(
            language="ja",
            content=row[5],
            type=enums.Document.Type.PLAIN_TEXT,
        )

        sentiment = client.analyze_sentiment(document=document).document_sentiment
        writer.writerow([sentiment.score, sentiment.magnitude, row[5]])

out_file.close()
```
こうやって使う。

```shell
python3 script.py onaga.csv onaga_result.csv
```

## 課金についての注意

Google Cloud Natural Language API の感情分析は API の呼び出し1回を1ユニットとして、
1ヶ月あたり5000〜100万ユニット以内の使用状況だと1000ユニットあたり1ドル
(ただし月5000ユニットまでは無料)という課金体型なのだが、元データは5万ツイートくらい
あって、そのままぶっ込むと50ドルくらい掛かってお財布に優しくないので、気をつけようね！

## 結果

よく分からなかった。

…いや、違うんです、ちょっと待って、帰らないで。

例えばこういうツイートがあるとするじゃないすか。これが感情分析にかけると `score` が
0.3、`magnitude` が `0.7` くらいのポジティブ扱いなんすよ。哀悼の意だぞ？

> ○○、亡くなられたんですね。心から哀悼の意を捧げます。

かと思えば、こういう人の死を貶めるような発言が `score` が `0.0`、`magnitude` が `0.1`
くらいのニュートラルな扱いになる。

> ○○、逝ったのか。ざまあああｗｗｗｗｗ

前半の「○○、逝ったのか。」と「ざまあああｗｗｗｗ」に切って食わせても、どちらの文も
score は `0.0`。

あ…やっぱ…日本語は難しいんやなって…。

## その他

最後になりましたが、翁長雄志氏、津川雅彦氏、両氏のご冥福をお祈りいたします。

[original]: http://www.po-jama-people.info/entry/2018/08/12/113435
[api]: https://cloud.google.com/natural-language/