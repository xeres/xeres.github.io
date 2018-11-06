---
title: "なぜ Zone Apex に CNAME を設定できないのか"
date: 2018-11-06 21:40:00 +0900
---

## 発端

ELB の話をしていて、Zone Apex に CNAME を設定できないのは何故かという話になった。

<iframe src="//www.slideshare.net/slideshow/embed_code/key/sm3ZFfLpee1hxt?startSlide=19" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/AmazonWebServicesJapan/aws-black-belt-online-seminar-2016-elastic-load-balancing" title="AWS Black Belt Online Seminar 2016 Elastic Load Balancing" target="_blank">AWS Black Belt Online Seminar 2016 Elastic Load Balancing</a> </strong> from <strong><a href="//www.slideshare.net/AmazonWebServicesJapan" target="_blank">Amazon Web Services Japan</a></strong> </div>

* Zone Apex (www.example.com ではなく example.com を指定)の場合
    - 通常の DNS サーバーでは CNAME 設定不可

## 結論

結論から言うと ELB の仕様は関係なく、DNS の仕様が理由みたい。

DNS の一般的な運用や設定の誤りを記載した [RFC1912](rfc1912) に、ずばりの記載がある。

> 2.4 CNAME records
>
> A CNAME record is not allowed to coexist with any other data.  In
> other words, if suzy.podunk.xx is an alias for sue.podunk.xx, you
> can't also have an MX record for suzy.podunk.edu, or an A record, or
> even a TXT record.  Especially do not try to combine CNAMEs and NS
> records like this!:

(拙訳)
> 2.4 CNAME レコード
>
> CNAME レコードはいかなる他のデータとも共存できません。言い換えれば、もし
> suzy.podunk.xx が sue.podunk.xx のための別名であるなら、あなたは suzy.podunk.edu に
> 対して、MX レコード、Aレコードまたは TXT レコードを設定することはできません。
> 特に CNAME と NS レコードをこのように組み合わせないでください。

Zone Apex (aka "naked domain", "bare domain") は、そのドメインに対しての
権威 DNS サーバーを指定するために NS レコードの設定が必須となるので、
Zone Apex に同時に CNAME を設定してはいけない、ということみたい。

じゃあ「CNAME レコードはいかなる他のデータとも共存できない」ってのは
DNS 自体の仕様の RFC に記載があるのかと思ったけど、ちょっと見つけられなかった。

DNS のコンセプト、仕様や実装を記載した RFC1034 や RFC1035 あたりに
"MUST NOT" な記載があるかと思ったが、自分には見つけられなかった
(どちらも既に Update されているので注意)。

## Route 53 でエイリアスの設定をすると回避できるのは何故？

[Route 53 のエイリアス](alias) は実際には A/AAAA レコードを返すため。

## エイリアスレコードのお得なところ

Zone Apex とか全然関係ないところで、Route 53 の仕様に「AWS リソースに対する
エイリアスクエリに課金されない」というものがあるので、何らかの理由で
CNAME レコードを設定したいケース以外ではエイリアスを利用したほうが良さそう。

[rfc1912]: https://tools.ietf.org/html/rfc1912
[alias]: https://docs.aws.amazon.com/ja_jp/Route53/latest/DeveloperGuide/resource-record-sets-choosing-alias-non-alias.html
