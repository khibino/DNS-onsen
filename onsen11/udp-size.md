
UDP と IPフラグメンテーション - DNS アプリケーションからの視点
=======

DNS UDP53 の IP パケット
-----

```
  +--------+-----+------------------------------------+
  |   IP   | UDP |     UDP payload - DNS Message      |
  +--------+-----+------------------------------------+

            |                                         |
            |<--------       IP payload      -------->|
  |                                                   |
  |<--------          L2 data frame          -------->|

DNS メッセージパケットの概略
```

L2 ペイロードのサイズが足りないとどうなる

IPフラグメンテーション
-----

```
  +--------+-----+------------------------------------+
  |   IP   | UDP |     UDP payload - DNS Message      |
  +--------+-----+------------------------------------+
```

```
  +--------+-----+---------+
  |   IP   | UDP | (data)  |
  +--------+-----+---------+

           |<-IP payload ->|
  |<--- L2 data frame  --->|

  +--------+---------------+
  |   IP   |    (data)     |
  +--------+---------------+

           |<-IP payload ->|
  |<--- L2 data frame  --->|

  +--------+-----------+
  |   IP   |  (data)   |
  +--------+-----------+

IPフラグメンテーションの例
```

* パケットを L2 data frameに収まるように分割する
* 分割されたパケットが個別に送られる
    * 一般的に<strong>順序も入れ換わる</strong>


IPフラグメンテーションの再構成 - IPv4
-----

https://datatracker.ietf.org/doc/html/rfc791#section-3.1

Internet Header Format (IPヘッダ)

Figure 4. Example Internet Datagram Header

```
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version|  IHL  |Type of Service|          Total Length         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Identification        |Flags|      Fragment Offset    |  ⇐⇐
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Time to Live |    Protocol   |         Header Checksum       |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                       Source Address                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Destination Address                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

* Flags に more-fragments flag があるので最後かどうか分かる
* Fragment Offset(64ビット単位) で分割されたパケットが元のどの位置かわかる

IPフラグメンテーションの再構成 - IPv6
-----

```
  +------+-----------+----------------+-----------+-------------------------+
  | IPv6 | v6 Ext... | v6 fragment Hd | v6 Ext... |  IPv6 fragment payload  |
  +------+-----------+----------------+-----------+-------------------------+

  |<------ IPv6 ヘッダ + IPv6 拡張ヘッダ -------->|

IPv6 ではフラグメント情報が拡張ヘッダとなっている
```

https://datatracker.ietf.org/doc/html/rfc8200#section-4.5

Fragment Header

```
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Next Header  |   Reserved    |      Fragment Offset    |Res|M|  ⇐⇐
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         Identification                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

* M に more-fragments flag があるので最後かどうか分かる
* Fragment Offset(64ビット単位) で分割されたパケットが元のどの位置かわかる


BSDソケットアプリケーションからみた IP フラグメンテーション
-----

```
       ssize_t recvfrom(int sockfd, void *buf, size_t len,  ⇐ バイト列しか受けとれない
                        int flags,
                        struct sockaddr * src_addr,
                        socklen_t * addrlen);
```

* IPフラグメンテーションの再構成は、カーネルレベルで実行される
    * <strong>BSDソケットアプリケーションからは観測できない</strong>
    * 再構成が成功すれば、バッファが足りる限り、元の UDP のペイロードが一度で読めることが保証される
    * つまり、<strong>フラグメンテーションが起きなかったときと動作は変わらない</strong>
* 再構成後に UDP のチェックサムの検査が通ったものだけが、アプリケーションから読める

https://datatracker.ietf.org/doc/html/rfc768

User Datagram Protocol

```
                 +--------+--------+--------+--------+
                 |     Source      |   Destination   |
                 |      Port       |      Port       |
                 +--------+--------+--------+--------+
                 |                 |                 |
                 |     Length      |    Checksum     |
                 +--------+--------+--------+--------+
		 |  ...                              |
```

IP フラグメンテーションはアプリケーションから見えない
-----

* IPフラグメンテーションの再構成は、カーネルレベルで実行される
    * <strong>BSDソケットアプリケーションからは観測できない</strong>
    * 再構成が成功すれば、バッファが足りる限り、元の UDP のペイロードが一度で読めることが保証される
    * つまり、<strong>フラグメンテーションが起きなかったときと動作は変わらない</strong>
* 再構成後に UDP のチェックサムの検査が通ったものだけが、アプリケーションから読める

<strong>フラグメンテーションが起きているときに動かない場合、
アプリケーションより下でなにか問題がある</strong>

* IP フラグメンテーションを起こさないように、サイズを調整するのはアプリケーションでもできる

<strong>サイズをうまく調整できない場合、アプリケーションに問題がある</strong>


UDP パケットサイズを調整する - IPv4
-----

IP header + UDP headr + DNS message のサイズを
MTU (Maximum Transmission Unit） に収めたい

* MTU として期待して良いサイズは?

IPv4 の下限

https://datatracker.ietf.org/doc/html/rfc791#section-3.1

```
                                            All hosts must be prepared
    to accept datagrams of up to 576 octets (whether they arrive whole
    or in fragments).
```

```
                                                                    For
    example, this size allows a data block of 512 octets plus 64 header
    octets to fit in a datagram.
```

* 最小のサイズとして 576 bytes ( = 64 + 512 ) というお気持ちが書いてある
    * 64 bytes という謎のサイズ (IPv4 UDP, IPv6 UDP とも異なる)

UDP パケットサイズを調整する - IPv6
-----

IPv6 の下限

https://datatracker.ietf.org/doc/html/rfc8200#section-5

```
   IPv6 requires that every link in the Internet have an MTU of 1280
   octets or greater.  This is known as the IPv6 minimum link MTU.
```

IETF の会議で決めたときに 1280 bytes = 256 + 1024 というお気持ちがある
という話があった、と噂で聞きいた

RIPE の実験レポート 2013.10.02

https://labs.ripe.net/author/emileaben/ripe-atlas-packet-size-matters/

1280 を越えると、急にロス率が増える


UDP パケットサイズを調整する
-----

IPv4 header (拡張無し) + UDP header = 20 + 8 = 28

IPv6 header (拡張無し) + UDP header = 40 + 8 = 48

1280 - 48 = 1232

https://datatracker.ietf.org/doc/html/rfc9000#section-14

QUIC は少なくとも 1200 byte 必要 (たとえば DNS over QUIC)

```
              QUIC MUST NOT be used if the network path cannot support a
    maximum datagram size of at least 1200 bytes.
```

```
    Note: This requirement to support a UDP payload of 1200 bytes
    limits the space available for IPv6 extension headers to 32 bytes or IPv4
    options to 52 bytes if the path only supports the IPv6 minimum MTU of
    1280 bytes.
```

1280 - 1200 - 28 = 52

1280 - 1200 - 48 = 32


UDP パケットサイズを調整する - DNS
-----

UDP53 の DNS Message (= UDP payload) のサイズは制限されている

https://datatracker.ietf.org/doc/html/rfc6891#section-4.3

```
   Traditional DNS messages are limited to 512 octets in size when sent
   over UDP [RFC1035].
```

```
       +------------+--------------+------------------------------+
       | Field Name | Field Type   | Description                  |
       +------------+--------------+------------------------------+
       | NAME       | domain name  | MUST be 0 (root domain)      |
       | TYPE       | u_int16_t    | OPT (41)                     |
       | CLASS      | u_int16_t    | requestor's UDP payload size |  ⇐⇐
       | TTL        | u_int32_t    | extended RCODE and flags     |
       | RDLEN      | u_int16_t    | length of all RDATA          |
       | RDATA      | octet stream | {attribute,value} pairs      |
       +------------+--------------+------------------------------+
```

* EDNS0 無しの場合、最大で 512 byte
* EDNS0 有りの場合、最大サイズを EDNS0 ヘッダに書ける (16bit)
* 応答が最大サイズを越えた場合は TC bit を利用して、TCPフォールバックさせる

bowline での判定
-----

https://github.com/iijlab/dnsext/blob/d52aaf86c6e45e9252b5be786a58e976fd32e0a0/dnsext-iterative/DNS/Iterative/Server/Pipeline.hs#L175-L199

* additional section を抜かして、サイズが足りる場合は TC bit を利用しない
    * 必要な glue が落ちてしまうケースがある?

付録
-----

https://datatracker.ietf.org/doc/html/rfc8200#section-3

IPv6 Header Format

```
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |Version| Traffic Class |           Flow Label                  |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |         Payload Length        |  Next Header  |   Hop Limit   |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                         Source Address                        +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +                                                               +
   |                                                               |
   +                      Destination Address                      +
   |                                                               |
   +                                                               +
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

      Version             4-bit Internet Protocol version number = 6.

      Traffic Class       8-bit Traffic Class field.  See Section 7.

      Flow Label          20-bit flow label.  See Section 6.

      Payload Length      16-bit unsigned integer.  Length of the IPv6
                          payload, i.e., the rest of the packet
                          following this IPv6 header, in octets.  (Note
                          that any extension headers (see Section 4)
                          present are considered part of the payload,
                          i.e., included in the length count.)
```
