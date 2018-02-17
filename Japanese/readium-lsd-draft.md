---
layout: post
title:  "Readium License Status Document Specification 1.0"
date:   2016-11-25 10:00:00 +0100
categories: specifications
permalink: /readium-lsd-specification/
---

Copyright 2016, Readium Foundation. All Rights Reserved.

Document Revision: 1.2

この文書は、Readium License Status Document Specification 1.0の日本語抄訳である。この抄訳は参考であって規定ではない。翻訳上の誤りもありうる。

## 1. 概要

### 1.1 目的と範囲

{:.information}
**この節は参考である。**

この仕様、License Status Document (1.0)は、DRMライセンスの状態を表すための文書を定義し、さらにDRMライセンスの状態に関連するインタラクションを定義する。これらのインタラクションは、コンテンツプロバイダと読書システムの両方にとって追加の責任を課す。

主な目的の1つは、公共図書館での貸出をサポートすることでうる。ユーザーは、期限前に貸し出し期間を延長するか、期限前に貸し出しを取り消すことができる。

### 1.2. 用語

この仕様では、以下のEPUBおよびLicensed Content Protection[[LCP](#normative-references)]用語を使用している。

**出版物**

相互に関連する[リソース](https://www.idpf.org/epub/30/spec/#gloss-publication-resource-cmt-or-foreign)の集まりとして構成され、[EPUB 3仕様](https://www.idpf.org/epub/30/spec/#sibling-specs)で定義されているEPUBコンテナにパッケージ化された論理ドキュメントエンティティ。


**ユーザー**

EPUBリーディングシステムを使用してEPUB出版物を消費する個人。

**EPUBリーディングシステム（またはリーディングシステム）**

[EPUB 3仕様](https://www.idpf.org/epub/30/spec/#sibling-specs)に準拠した形でEPUB出版物を処理してユーザに提示するシステム。

さらに、次の用語を使用する。

**コンテンツプロバイダ**

ライセンス文書と出版物をユーザーに付与する権限をもつもの（通常はWebサーバー）。

**ライセンス文書**

さまざまのキーへの参照と、ある出版物の特定のDRMが適用する制限とを含む文書。

**ステータス文書**

ライセンス文書の履歴に関する情報と、現在のステータスおよび利用可能なインタラクションを含む文書。

### 1.3. 例

{:.information}
**この節は参考である。**

<em>次の例では、ステータス文書は、ライセンスが単一のデバイスに登録されており、更新または返却される可能性があることを示している。</em>


    {
      "id": "234-5435-3453-345354",
      "status": "active",
      "message": "Your license is currently active and has been used on one device.",

      "updated": {
        "license": "2016-08-05T00:00:00Z",
        "status": "2016-08-08T00:00:00Z"
      },

      "links": [
        {"rel": "license",
         "href": "https://example.org/license/35d9b2d6",
         "type": "application/vnd.readium.lcp.license-1.0+json"},
        {"rel": "register",
         "href": "https://example.org/license/35d9b2d6/register{?id,name}",
         "type": "application/vnd.readium.license.status.v1.0+json",
         "templated": true},
        {"rel": "return", 
         "href": "https://example.org/license/35d9b2d6/return{?id,name}",
         "type": "application/vnd.readium.license.status.v1.0+json",
         "templated": true},
        {"rel": "renew",
         "href": "https://example.org/license/35d9b2d6/renew",
         "type": "text/html"},
        {"rel": "renew",
         "href": "https://example.org/license/35d9b2d6/renew{?end,id,name}",
         "type": "application/vnd.readium.license.status.v1.0+json",
         "templated": true}
      ],
      "potential_rights": {
        "end": "2014-09-13T00:00:00Z"
      },
      "events": [
        {"type": "register", "name": "eBook App (Android)", "timestamp": "2016-07-14T00:00:00Z", "id": "709e1380-3528-11e5-a2cb-0800200c9a66"}
      ]
    }

### 1.4 Conformance Statements

この文書中のキーワード<b>MUST</b>, <b>MUST NOT</b>, <b>REQUIRED</b>, <b>SHALL</b>, <b>SHALL NOT</b>, <b>SHOULD</b>, <b>SHOULD NOT</b>, <b>RECOMMENDED</b>, <b>MAY</b>, 及び<b>OPTIONAL</b>は[[RFC2119](#normative-references)]に従って解釈される。

この仕様のすべての節は規定であるが、「この節は参考である」という参考ステータスレベルがあればその限りではない。節および付録に参考ステータスが適用されていれば、その子である内容及びサブセクション（もしあれば）にも適用される。

この仕様のすべての例は参考である。

## 2. ステータス文書

### 2.1. コンテンツの適合性

ステータス文書は、以下のすべての基準を満たさなければならない（MUST）</b> 。

文書プロパティ

[JSON](#normative-references)で定義されているJSONドキュメントの適合性制約を満たさなければならない（MUST）。単一のJSONオブジェクトとして構文解析できなければならない（MUST）。UTF-8を使用して符号化されなければならない（MUST）。

ステータス文書へのアクセスは、いかなる形式の認証も要求してはならない（MUST NOT）。

ステータス文書は、安全でない接続で提供されるべきではない（SHOULD NOT）。

ステータス文書のMIMEメディアタイプは`application/vnd.readium.license.status.v1.0+json`であり、HTTPサーバはContent-Typeヘッダを適切に設定しなければならない（MUST）。

### 2.2. コア情報

ステータス文書は、以下の情報を含まなければならない（MUST）。

<table class="table-bordered large">
  <tr>
    <th>キー</th>
    <th>意味</th>
    <th>フォーマット/データタイプ</th>
  </tr>
  <tr>
    <td>id</td>
    <td>ステータス文書に関連付けられたライセンス文書の一意の識別子</td>
    <td>文字列</td>
  </tr>
  <tr>
    <td>status</td>
    <td>ライセンスの現在の状態</td>
    <td>セクション2.3で定義されているように限定された語彙</td>
  </tr>
  <tr>
    <td>message</td>
    <td>ライセンスの現状についてのメッセージであり、ユーザーへの表示を意図したもの</td>
    <td>文字列</td>
  </tr>
</table>

<br />
メッセージキーに提供されるすべてのメッセージは、ステータス文書への要求に含まれるHTTPの`Accept-Language`ヘッダーに基づいてローカライズされるべきである（SHOULD）。

### 2.3. 状態

`status`フィールドには、ライセンスの現在のステータスが簡潔に記載される。次の値が使用できる。

<table class="table-bordered large">
  <tr>
    <th>値</th>
    <th>意味</th>
  </tr>
  <tr>
    <td>ready</td>
    <td>ライセンス文書は利用可能だが、ユーザーはまだライセンスおよび/またはステータス文書にアクセスしていない。</td>
  </tr>
  <tr>
    <td>active</td>
    <td>ライセンスは有効であり、このデバイスにおいて使用できると正常に登録された。 ライセンス文書に登録リンクが含まれていない場合、またはライセンス自体による登録メカニズムがない場合は、これがデフォルト値である。</td>
  </tr>
  <tr>
    <td>revoked</td>
    <td>ライセンスはもはやアクティブではなく、発行者によって無効にされている。</td>
  </tr>
  <tr>
    <td>returned</td>
    <td>ライセンスはもはやアクティブではなく、ユーザーによって無効にされている。</td>
  </tr>
  <tr>
    <td>cancelled</td>
    <td>ライセンスは、アクティベーション前に取り消されたため無効にされている。</td>
  </tr>
  <tr>
    <td>expired</td>
    <td>ライセンスは有効期限切れなので無効にされている</td>
  </tr>
</table>

### 2.4. タイムスタンプ

ステータス文書は、`updated`オブジェクトを含まなければならず（MUST）、このオブジェクトは以下のキーとタイムスタンプを含まなければならない。

<table class="table-bordered large">
  <tr>
    <th>キー</th>
    <th>意味</th>
    <th>フォーマット/データタイプ</th>
  </tr>
  <tr>
    <td>license</td>
    <td>ライセンス文書が最後に更新された日時</td>
    <td>ISO 8601の日付と時刻</td>
  </tr>
  <tr>
    <td>status</td>
    <td>ステータス文書が最後に更新された日時</td>
    <td>ISO 8601の日付と時刻</td>
  </tr>
</table>

### 2.5. リンク

ステータス文書は`links`オブジェクトを含まなければならない（MUST） 。

ステータス文書は、その関係が`license`に設定されているリンクを少なくとも1つ含まなければならない（MUST） 。

#### 2.5.1 ライセンス関係
<table class="table-bordered large">
  <tr>
    <th>関係</th>
    <th>意味</th>
    <th>テンプレートをもつかどうか</th>
    <th>必須かどうか</th>
    <th>HTTP動詞</th>
  </tr>
  <tr>
    <td>license</td>
    <td>ステータス文書に関連付けられたライセンス文書の場所</td>
    <td>持たない</td>
    <td>必須</td>
    <td>GET</td>
  </tr>
</table>

#### 2.5.2. リンクオブジェクト

リンクに含まれる各Linkオブジェクトは、次のキーをサポートする。

<table class="table-bordered large">
  <tr>
    <th>名</th>
    <th>値</th>
    <th>フォーマット/データタイプ</th>
    <th>必須かどうか</th>
  </tr>
  <tr>
    <td>href</td>
    <td>リンクの場所</td>
    <td>URIまたはURIテンプレート</td>
    <td>必須</td>
  </tr>
  <tr>
    <td>rel</td>
    <td>文書へのリンク関係</td>
    <td>公知の関係値のリスト、拡張のためのURI</td>
    <td>必須</td>
  </tr>
  <tr>
    <td>title</td>
    <td>リンクのタイトル</td>
    <td>文字列</td>
    <td>必須ではない</td>
  </tr>
  <tr>
    <td>type</td>
    <td>外部リソースのMIMEメディアタイプ</td>
    <td>MIMEメディアタイプ</td>
    <td>必須ではないが、強く推奨される</td>
  </tr>
  <tr>
    <td>templated</td>
    <td>hrefがURIテンプレートであることを示す</td>
    <td>論理値</td>
    <td>必須ではなく、デフォルト値はfalse</td>
  </tr>
  <tr>
    <td>profile</td>
    <td>外部リソースを識別するために使用されるプロファイル</td>
    <td>URI</td>
    <td>必須ではない</td>
  </tr>
</table>

### 2.6. 潜在的な権利

ステータス文書は、`potential_rights`オブジェクトを使用して、ライセンス文書に許される潜在的な変更のリストを提供してもよい（MAY）。

この仕様は、潜在的な権利を表す以下の表現を定義する。

<table class="table-bordered large">
  <tr>
    <th>キー</th>
    <th>意味</th>
    <th>フォーマット/データタイプ</th>
  </tr>
  <tr>
    <td>end</td>
    <td>ライセンスが終了する日時</td>
    <td>ISO 8601の日付と時刻</td>
  </tr>
</table>


### 2.7. イベント

ステータス文書は、ライセンス文書のステータスの変更についてのイベントの順序付けられたリストを提供してもよい（ MAY） 。

これらのイベントは、`events`オブジェクトに記録されており、各イベントは以下のキーによって記述される。

<table class="table-bordered large">
  <tr>
    <th>キー</th>
    <th>意味</th>
    <th>フォーマット/データタイプ</th>
  </tr>
  <tr>
    <td>type</td>
    <td>イベントのタイプを特定する。</td>
    <td>3. インタラクションの2. ステータス文書に定義されるlink関係</td>
  </tr>
  <tr>
    <td>name</td>
    <td>クライアントの名前，対話中にクライアントによって提供される</td>
    <td>文字列</td>
  </tr>
  <tr>
    <td>id</td>
    <td>クライアントを特定するもの，対話中にクライアントによって提供される</td>
    <td>文字列</td>
  </tr>
  <tr>
    <td>timestamp</td>
    <td>イベントが発生した日時</td>
    <td>ISO 8601の日付と時刻</td>
  </tr>
</table>

次の`type`値が許可されている。

<table class="table-bordered large">
  <tr>
    <th>名</th>
    <th>値</th>
  </tr>
  <tr>
    <td>register</td>
    <td>デバイスによる正常な登録イベントを通知する。</td>
  </tr>
  <tr>
    <td>renew</td>
    <td>更新イベントが成功したことを通知する。</td>
  </tr>
  <tr>
    <td>return</td>
    <td>成功したリターンイベントを通知する。</td>
  </tr>
  <tr>
    <td>revoke</td>
    <td>失効イベントを通知する。</td>
  </tr>
  <tr>
    <td>cancel</td>
    <td>キャンセルイベントを通知する。</td>
  </tr>
</table>

## 3. インタラクション

標準化された文書に加えて、LSD仕様では多数のインタラクションも定義されています。それらはすべて`links`オブジェクトを通じて公開される。

### 3.1. エラー処理

クライアントとエンドユーザの両方に一貫した動作を提供するには、すべてのサーバが[[RFC7807](#normative-references)]で定義されている詳細なエラー情報JSONオブジェクトを用いてエラーを扱わけなればならない。さらに、エラー情報JSONオブジェクトには次の要件が追加される。

* サーバーは 、詳細なエラー情報JSONオブジェクトにおいて`title`と`type`を返さなければならない（MUST）。

* サーバは、クライアントが送信した`Accept-Language`ヘッダに基づいて、 `title`と`detail`の両方をローカライズすることが望ましい(SHOULD)。

この仕様で定めるインタラクションのそれぞれについて、インタラクションが失敗したときにクライアントに返されるさまざまの失敗タイプを規定しています。

### 3.2. ライセンスのステータスの確認

クライアントは、ライセンス文書を更新するためにステータス文書を定期的に取り出すべきである(SHOULD)。

ステータス文書の`updated`オブジェクトの`license`タイムスタンプがライセンス文書のローカルコピーに含まれているタイムスタンプよりも新しい場合、クライアントはライセンス文書を再度ダウンロードし、以前のコピーを新しいものに置き換えなければならない（MUST）。

ステータス文書の`status`値が、対応する最新のライセンス文書と矛盾する場合は、ライセンス文書が優先される。

ステータス文書が利用できない場合、またはクライアントがインターネット接続を取得できない場合、ライセンス文書に関連付けられた出版物にユーザーがアクセスすることを禁止してはならない（MUST NOT）。

### 3.3. デバイスの登録

登録は、あるライセンス文書とのインタラクションがあったクライアントがいくつあるのかというヒント情報をサーバーに知らせるためにある。

クライアントが初めてライセンス文書を開き、関連付けられたステータス文書にアクセスするとき、

* ステータス文書に公開されているリンクを使用してクライアントを登録しようとしなければならない（MUST）。

* 登録が失敗した場合、ライセンス文書に関連付けられた出版物にユーザーがアクセスすることをクライアントはブロックしてはならない(MUST NOT)。

* ライセンス文書を最初に開いたときに登録を行えなかった場合は、クライアントはあとで登録を試みなければならない（MUST）。

登録のとき、クライアントは、どのステータス文書とのインタラクションの場合でも、そのデバイスに対する同一の識別子を常に送信しようとしなければならない（MUST）。プロバイダとの今後のインタラクションでは、同じ識別子/名前を使用すべきである（SHOULD）。クライアントは、一意の識別子を生成するときにユーザーのプライバシーを考慮することが望ましい（SHOULD）。たとえば、ソフトウェアのインストール中にランダムな文字列を生成するなど。複数のプロバイダー間にわけるユーザーの追跡を防ぐために、クライアントはプロバイダーごとに別のデバイスIDを生成してもよい（MAY）。

プロバイダーがライセンスの乱用を監視するために登録を用いるなら、プロバイダーは偽造された登録を防止するよう注意しなければならない(SHOULD)。

<table class="table-bordered large">
  <tr>
    <th>関係</th>
    <th>意味</th>
    <th>テンプレート付きかどうか</th>
    <th>必須かどうか</th>
    <th>HTTPの動詞</th>
  </tr>
  <tr>
    <td>register</td>
    <td>新しいデバイスをライセンスに関連付ける</td>
    <td>テンプレート付き</td>
    <td>必須ではない</td>
    <td>POST</td>
  </tr>
</table>

<br />

<table class="table-bordered large">
  <tr>
    <th>パラメータ</th>
    <th>フォーマット</th>
    <th>形式</th>
    <th>必須かどうか</th>
  </tr>
  <tr>
    <td>id</td>
    <td>文字列</td>
    <td>デバイスを一意に特定する識別子</td>
    <td>必須</td>
  </tr>
  <tr>
    <td>name</td>
    <td>文字列</td>
    <td>人間が読めるデバイス名</td>
    <td>必須</td>
  </tr>
</table>

<br />
*単純なアクティベーションリンクの例。*

    {
      "links": [
        {"rel": "register", 
         "href": "https://example.org/license/aaa-bbbb-ccc/register{?id,name}",
         "type": "application/vnd.readium.license.status.v1.0+json",
         "templated": true}
      ]
    }

*予想される行動*


<table class="table-bordered large">
  <tr>
    <th>サーバー側の動作</th>
    <th>HTTPステータスコード</th>
    <th>クライアント側の動作</th>
  </tr>
  <tr>
    <td>サーバーは'id'で識別されるデバイスを登録し、更新されたステータス文書を返す。
サーバーは、更新されたオブジェクトのステータスキーに含まれるステータス文書のタイムスタンプを更新しなければならない（MUST）。

ステータスが以前にreadyに設定されていた場合は、サーバーによってactiveに更新しなければならない（MUST）。
また、サーバーは、ステータス文書のイベントオブジェクトに新しいイベントを追加してもよい（MAY）。</ td>
    <td>200</td>
    <td style="width:150px">クライアントは再度デバイスを登録しようとてはならない（MUST NOT）。</td>
  </tr>
</table>

<br />
*失敗モード*

<table class="table-bordered large">
  <tr>
    <th>タイプ</th>
    <th>ステータスコード</th>
    <th>タイトル</th>
  </tr>
  <tr>
    <td>http://readium.org/license-status-document/error/registration</td>
    <td>400</td>
    <td>デバイスを正しく登録できませんでした。</td>
  </tr>
  <tr>
    <td>http://readium.org/license-status-document/error/server</td>
    <td>5xx</td>
    <td>予期しないエラーが発生しました。</td>
  </tr>
</table>

### 3.4. 出版物の返却

出版物の返却は、主に図書館での利用事例を対象としており、新規の貸し出しのために早期に出版物を顧客は返すことができる。

返却が成功しなかった場合、クライアントは後でライセンスを再度返却しようとするべきであり(SHOULD)、必ずしもユーザに再度尋ねる必要はない。

<table class="table-bordered large">
  <tr>
    <th>関係</th>
    <th>意味</th>
    <th>テンプレート付きか</th>
    <th>必須かどうか</th>
    <th>HTTP動詞</th>
  </tr>
  <tr>
    <td>return</td>
    <td>ライセンスをすぐに無効にするように求める</td>
    <td>テンプレート付き</td>
    <td>必須ではない</td>
    <td>PUT</td>
  </tr>
</table>

<br />

<table class="table-bordered large">
  <tr>
    <th>パラメタ</th>
    <th>フォーマットFormat</th>
    <th>意味</th>
    <th>必須かどうか</th>
  </tr>
  <tr>
    <td>id</td>
    <td>文字列</td>
    <td>デバイスを一意に特定する識別子</td>
    <td>必須ではない</td>
  </tr>
  <tr>
    <td>name</td>
    <td>文字列</td>
    <td>人間に読めるデバイス名</td>
    <td>必須ではない</td>
  </tr>
</table>

<br />
*単純な返却リンクの例。*


    {
      "links": [
        {"rel": "return",
         "href": "https://example.org/license/aaa-bbbb-ccc/return{?id,name}",
         "type": "application/vnd.readium.license.status.v1.0+json",
         "templated": true}
      ]
    }


*Expected Behavior*

<table class="table-bordered large">
  <tr>
    <th>サーバー側の動作</th>
    <th>HTTPステータスコード</th>
    <th>クライアント側の動作</th>
  </tr>
  <tr>
    <td>サーバーは更新されたステータス文書を返さなければならない（MUST）。ステータス文書の`status`は、以前に "active"に設定されていた場合には"returned"に、以前に "ready"に設定されていた場合には"cancelled"に設定されていなければならない（MUST）。

サーバーは、`updated`オブジェクトの`status`と`license`の両方のキーに含まれるタイムスタンプ（ライセンス文書のそれとステータス文書のそれ）を更新しなければならない（MUST）。

サーバーは、ステータス文書の`events`オブジェクトに新しいイベントを追加してもよい（MAY）。</td>
    <td>200</td>
    <td>クライアントは更新されたステータス文書をダウンロードしなければならない（MUST）。

クライアントは、ユーザーが出版物をまた開くことを許可してはならない（MUST NOT）。

クライアントはライセンスをもう一度返却しようとの試みるべきではない（SHOULD NOT）。</td>
  </tr>
</table>

<br />
*失敗モード*

<table class="table-bordered large">
  <tr>
    <th>タイプ</th>
    <th>ステータスコード</th>
    <th>タイトル</th>
  </tr>
  <tr>
    <td>http://readium.org/license-status-document/error/return</td>
    <td>400</td>
    <td>出版物をきちんと返却することができませんでした。</td>
  </tr>
  <tr>
    <td>http://readium.org/license-status-document/error/return/already</td>
    <td>403</td>
    <td>出版物はすでに返却されています。</td>
  </tr>
  <tr>
    <td>http://readium.org/license-status-document/error/return/expired</td>
    <td>403</td>
    <td>出版物はすでに期限切れです。</td>
  </tr>
  <tr>
    <td>http://readium.org/license-status-document/error/server</td>
    <td>5xx</td>
    <td>予期しないエラーが発生しました。</td>
  </tr>
</table>

### 3.5. ライセンスの更新

ライセンスの更新も、主に図書館での利用事例を対象としており、顧客はローンを更新してある期間延長できる。

<table class="table-bordered large">
  <tr>
    <th>関係</th>
    <th>意味</th>
    <th>必須かどうか</th>
  </tr>
  <tr>
    <td>renew</td>
    <td>ライセンスの有効期限を延長する</td>
    <td>必須ではない</td>
  </tr>
</table>

<br/>

<table class="table-bordered large">
  <tr>
    <th>メディアタイプ</th>
    <th>意味</th>
    <th>テンプレート付きかどうか</th>
    <th>動詞</th>
  </tr>
  <tr>
    <td>text/html</td>
    <td>人間による対話操作が必要となるURL。HTMLページを返す。対話操作は、ライセンス文書の期間を延ばすことに最終的には繋がるべきである（SHOULD）。</td>
    <td>テンプレート付きではない</td>
    <td>GET</td>
  </tr>
  <tr>
    <td>application/vnd.readium.license.status.v1.0+json</td>
    <td>ライセンス文書をプログラムで更新できるURL。 ステータス文書を返す。</td>
    <td>テンプレート付き</td>
    <td>PUT</td>
  </tr>
</table>

<br />
これらのパラメータは、ライセンス文書がプログラムによって更新される場合にのみ適用される。

<table class="table-bordered large">
  <tr>
    <th>パラメタ</th>
    <th>フォーマット</th>
    <th>意味</th>
    <th>必須かどうか</th>
  </tr>
  <tr>
    <td>id</td>
    <td>文字列</td>
    <td>デバイスを一意に特定する識別子</td>
    <td>必須ではない</td>
  </tr>
  <tr>
    <td>name</td>
    <td>文字列</td>
    <td>人間が読めるデバイス名</td>
    <td>必須ではない</td>
  </tr>
  <tr>
    <td>end</td>
    <td>ISO 8601</td>
    <td>ライセンスの新しい有効期限</td>
    <td>必須ではない</td>
  </tr>
</table>

<br />

*2つの更新リンクを持つ例：1つは人間による対話操作（HTML）を指し、もう1つは適切な情報をサーバーに送信できる（ステータス文書を返す）クライアントを指す。*

    {
      "links": [
        {"rel": "renew",
         "href": "https://example.org/license/aaa-bbbb-ccc/renew",
         "type": "text/html"},
        {"rel": "renew"
         "href": "https://example.org/license/aaa-bbbb-ccc/renew{?end,id,name}",
         "type": "application/vnd.readium.license.status.v1.0+json",
         "templated": true}
      ]
    }

これらのステータスコードおよび挙動は、リンクがステータス文書を返す場合にのみ適用される。

*期待される振る舞い*

<table class="table-bordered large">
  <tr>
    <th>サーバー側の振る舞い</th>
    <th>HTTPステータスコード</th>
    <th>クライアント側の振る舞い</th>
  </tr>
  <tr>
    <td>サーバーは更新されたステータスドキュメントを返さなければならない（MUST）。    サーバーは、`updated`オブジェクトの`status`キーと`license`キーに含まれるライセンス文書のスタンプとステータス文書のタイムスタンプの両方を更新しなければならない（MUST）。
サーバーはステータス文書の`events`オブジェクトに新しいイベントを追加してもよい（MAY）。
</td>
    <td>200</td>
    <td style="width:150px">クライアントは更新されたステータス文書をダウンロードしなければならない（MUST）。クライアントは後でライセンスをもう一度更新しようとしてもよい（MAY）。</td>
  </tr>
</table>
<br/>

<br />
*失敗モード*

<table class="table-bordered large">
  <tr>
    <th>タイプ</th>
    <th>ステータスコード</th>
    <th>タイトル</th>
  </tr>
  <tr>
    <td>http://readium.org/license-status-document/error/renew</td>
    <td>4xx</td>
    <td>出版物のライセンスをきちんと更新することができませんでした。</td>
  </tr>
  <tr>
    <td>http://readium.org/license-status-document/error/renew/date</td>
    <td>403</td>
    <td>不正確な更新期間。出版物のライセンスは更新できませんでた。</td>    
  </tr>
  <tr>
    <td>http://readium.org/license-status-document/error/server</td>
    <td>5xx</td>
    <td>予期しないエラーが発生しました。</td>
  </tr>
</table>

## 4. Readium Licensed Content Protection 1.0との関係

### 4.1. 導入

{:.information}
**この節は参考である。**

この仕様は、コンテンツプロバイダーがLCPライセンス文書[LCP](#normative-references)]をステータス文書に接続する方法を定義します。ステータス文書は、ライセンスの現在のステータスを記述する。

LCPライセンス文書に構造要素を追加するものではありませんが、この仕様が定義するリンク関係と潜在的な権利をLCPライセンス文書に書いてもよい(MAY)。

### 4.2. 状態関係

この仕様は、LCPライセンス文書の`links`オブジェクトで使用される新しいリンク関係を定義します。

<table class="table-bordered large">
  <tr>
    <th>関係</th>
    <th>意味</th>
    <th>必須かどうか</th>
  </tr>
  <tr>
    <td>status</td>
    <td>このライセンス文書に関連するステータス文書の場所</td>
    <td>必須ではない</td>
  </tr>
</table>

<br />

リレーションステータスを持つリンクが`links`オブジェクトに存在しない場合、コアLCP仕様に従ってLCPライセンス文書をクライアントは処理しなければならない（MUST）。

リレーションステータスを持つリンクが`links`オブジェクトに存在する場合、クライアントはこの仕様の[3節](#3-interactions)に記載された拡張ルールに従ってステータス文書を処理しなければならない（MUST） 。

ステータスリンクに記載されているURIは、有効なステータス文書を参照していなければならない（MUST） 。

## 引用規定

[JSON] [The application/json Media Type for JavaScript Object Notation (JSON)](https://www.ietf.org/rfc/rfc4627).

[RFC2119] [Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119).

[RFC78707] [JSON Problem Details](https://tools.ietf.org/html/rfc7807).

[URI-Template] [URI Template](https://tools.ietf.org/html/rfc6570).

[LCP] [Readium Licensed Content Protection](http://readium.github.io/readium-lcp-specification).
