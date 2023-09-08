
# トピック

----

## CNAME が返って NXDOMAIN

* 例: dig @ns1.yahoo.com. media.yahoo.com. A
* RFC 2308 によると、ドメイン名と CNAME は有るけど CNAME 先のドメイン名が無い
    * CNAME を検索 - dig @ns1.yahoo.com. media.yahoo.com. CNAME は NOERROR
* 反復検索中
    * 委任情報が無い、CNAME 先を確認する必要が無い
* A を検索
    * CNAME 先のドメイン名の権威サーバからの応答を確認する必要がある

----

## Empty Non-Terminal (ENT) に対して NXDOMAIN を返す権威サーバ

* 例: dig @danuoyinewns1.gds.alicdn.com.             com.danuoyi.alicdn.com. A <br>
  例: dig @danuoyinewns1.gds.alicdn.com.      alicdn.com.danuoyi.alicdn.com. A
    * dig @danuoyinewns1.gds.alicdn.com. sc02.alicdn.com.danuoyi.alicdn.com. A は有るのに途中の名前は NXDOMAIN
* 反復検索中
    * NOERROR で委任情報が無い場合と同様に扱った方が無難
    * RFC 8020 ( NXDOMAIN 下には何も無い ) に対応すると引けなくなる
