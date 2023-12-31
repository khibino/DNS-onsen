# 難しかったもの

@khibino

----

## RRの正規化順序(Canonical Order)

https://github.com/kazu-yamamoto/dnsext/issues/113

* RDATA の順序はワイヤーフォーマットのバイト列の順序
    * うっかり長さを連結してしまっていてハマりました
    * RDATA にドメイン名しか入っていない場合でも、ドメイン名の正規化順序とは異なるので注意

----

## 署名検証時の TTL の復元

https://github.com/kazu-yamamoto/dnsext/pull/128

* RRSIG にある original TTL で復元するのが正しい
    * jp. の権威サーバから、original TTL とは異なる TTL で返る例があった ( SOA だったと思います )

----

## ゾーンの親子同居

https://github.com/kazu-yamamoto/dnsext/issues/87

* 親子同居の場合、子ゾーンへの委任情報が返らない.
    * 親ゾーンと子ゾーンの権威サーバ集合が同じと仮定してよい?
        * 反復検索で問い合わせ先を分散させている場合、一部だけ同居だと変なことが起こりそう

----

## ゾーンの親子同居向けの特殊処理

https://github.com/kazu-yamamoto/dnsext/issues/126

* 親子同居で委任情報がもらえないので、DNSSEC 対応の場合にも DS が取得できない
    * DS を補うためのクエリが必要
	    * 余計に DS をクエリしてしまうバグ

----

## ドメイン名圧縮

* ワイヤーフォーマット内で、ドメイン名の文字列の領域を節約
    * ドメイン名文字列の代わりに、文字列を指すポインタを利用する
	* 文字列の途中でもポインタが使える

https://github.com/kazu-yamamoto/dnsext/issues/41

* ワイヤーフォーマットが正規化された表現にならない
    * DNSSEC の検証が必要なときには、圧縮無しの正規化ワイヤーフォーマットに直す必要がある

----

## ドメイン名圧縮アンチパターン

RFC9267 section-2

* ドメイン名圧縮の展開時に、結果のバイト列が 255バイトを越えてはいけない
* ポインタは直接ポインタを指してはいけない
* ポインタはDNSメッセージの範囲外を指してはいけない
* 無限ループしてはいけない

----

## ドメイン名圧縮関連の解釈器のバグ

ドメイン名 "." は 0x00 でいいはずなのに、0x00 を指すポインタを埋める実装があるらしい

https://github.com/kazu-yamamoto/dnsext/issues/82

----

## NSEC/NSEC3

* とにかく実装が面倒. とくに NSEC3
* ドメイン名の正規化順序は NSEC/NSEC3 で使う
    * https://datatracker.ietf.org/doc/html/rfc4034#section-6.1
