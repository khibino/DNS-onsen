# DNSフルリゾルバの研究開発

2023.09.09

@khibino

----

## 自己紹介

@khibino / 日比野 啓

* 以前は ISP で RADIUS 認証サーバ、認証システムの設計と開発
* 今は研究開発で DNSフルリゾルバの開発
* DNS 関連を始めて一年半くらいの新参者です

----

## 実装について

* レポジトリ https://github.com/kazu-yamamoto/dnsext
    * 主なライブラリ
        * dnsext-types
        * dnsext-svcb
        * dnsext-dnssec
        * dnsext-do53
        * dnsext-dox
    * フルリゾルバ dnsext-full-resolver
* 実装言語は Haskell

----

## フルリゾルバの構成

* 反復検索
* キャッシュ
* サーバー機能 (UDP, TCP, DoT, DoH2, DoH3, DoQ)
* DNSSEC検証

----

## 反復検索

* QNAME minimization の単純な実装
   * たとえば "www.iij.ad.jp" なら <br>
     ".", "jp.", "ad.jp.", "iij.ad.jp." "www.iij.ad.jp." の順に反復する
   * NameErr が返る場合でも、反復検査を継続する(その繰り返しは有限回で停止する)
       * Empty Non-Terminal で NameErr が返る場合の問題は回避される
* NS のキャッシュから委任情報を復元
* 問い合わせ結果は ranking answer 以上のものを返す(glue は返さない)

----

## サーバー機能 (UDP, TCP, DoT, DoH2, DoH3, DoQ)

* 後ほどデモで

----

## キャッシュ

* 優先度付き待ち行列
  * キーからの検索の他に、優先度の低いデータを削除できる
* 優先度付き待ち行列を利用してキャッシュを実装
  * キー: (ドメイン名, タイプ, クラス)
  * 優先度: キャッシュの破棄時刻 <br> ( <現在時刻> + TTL = <破棄時刻> )
  * 値: RRset または否定応答用の情報

----

## DNSSEC検証

* RRSIG(署名)検証
    * 返答の RRset の署名を検証する
* DS(委任の署名)検証
    * 委任先のゾーンの鍵の DIGEST が一致するか確認する
* NSEC/NSEC3
    * NameErr, NoData, 署名無し委任, ワイルドカード展開の場合の証拠を確認する

----

## デモ

* 反復検索
    * dug コマンドによるデモ
	    * www.iij.ad.jp. A
	    * does-not-exists.iij.ad.jp. A
	    * www.iij.ad.jp. NS
	    * insecure.mufj.jp. A
	    * c.uecac.jp. A
		* cname.small-is-beautiful.jp. A
	    * fake2.ed448.mufj.jp. A
* DoH/DoQ
    * bowline を起動して問い合わせてみる
* フルリゾルバのキャッシュを見る
    * bowline の管理ポートからキャッシュの内容を覗き見
