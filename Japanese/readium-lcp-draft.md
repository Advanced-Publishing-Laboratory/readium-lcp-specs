---
layout: post
title:  "Readium Licensed Content Protection Specification 1.0"
date:   2018-02-06 10:00:00 +0100
categories: specification
permalink: /readium-lcp-specification/

---

Copyright 2016, Readium Foundation. All Rights Reserved.

Document Revision: 1.1

この文書は、Readium Licensed Content Protection Specification 1.0の日本語抄訳である。この抄訳は参考であって規定ではない。翻訳上の誤りもありうる。

## 1. 概要

### 1.1 目的と範囲

{:.information}
**この節は参考である。**

特定の権利と制約を持つユーザーに出版物を配信するために、コンテンツプロバイダーは出版物のリソースを暗号化し、何らかのライセンスを付与することができる。

この仕様、LCP（Licensed Content Protection）は標準ライセンス文書と暗号化プロファイルを定義する。これらは、EPUB 3出版物のリソースを暗号化し、権利と制限を表明し、特定のユーザーのためのライセンスに基づいて、リーディングシステムに暗号解除鍵を安全に配信するためものです。

また、LCPは、リーディングシステムが暗号化されたリソースにアクセスしてライセンスを確認できるように、簡単なパスフレーズベースの認証方法を定義する。

## 1.2. 用語

### EPUB用語

省略

### LCP用語

次の用語は、この仕様で定義される。

**保護された出版物**

この仕様に従って保護されている出版物。

**ライセンス認証機関**

Readium LCPエコシステムを運営する事業体。

**ライセンス文書**

さまざまなキーへの参照、関連する外部リソースへのリンク、保護された出版物に適用される権利と制限、およびユーザー情報を含む文書。

**コンテンツキー**

保護された出版物のリソースを暗号化するために使用される対称鍵。ライセンス文書では、このコンテンツキーはユーザキーを使用して暗号化される。

**ユーザーパスフレーズ**

ユーザーが入力した文字列で、ユーザーキーの生成に使用される。

**ユーザーキー**

コンテンツキーと選択されたユーザー情報フィールドを復号化するために使用されるユーザーパスフレーズのハッシュ。

**暗号化プロファイル**

特定の保護された出版物およびそれに付随するライセンス文書で使用される一連の暗号化アルゴリズム。

**コンテンツプロバイダ（またはプロバイダ）**

保護された出版物のLCPライセンスをユーザーに提供する事業体。

**プロバイダ証明書**

ライセンス文書に含まれる証明書であり、コンテンツプロバイダを特定し、ライセンス文書の署名を検証するために用いられるもの。

**ルート証明書**

プロバイダ証明書が有効であることを確認するために、リーディングシステムに埋め込まれた証明書。

## 1.3. LCPの概要と例

{:.information}
**この節は参考である。**

リソースを暗号化し、その暗号化プロファイルを公開するために、LCPは2つの異なるファイルを用いている。

* ライセンス文書 (`META-INF/license.lcpl`)

* `META-INF/encryption.xml`

これらのファイルは両方ともEPUBコンテナに含まれているが、ライセンス文書は最初はコンテナから切り離して送受信できる。[[OCF](#normative-references)]仕様に従い、`META-INF/encryption.xml`は、保護されている出版物リソースを特定し、それらを解読するのに必要なコンテンツキーを指し示す。 このコンテンツキーは、ライセンス文書の中にあり、LCPの3番目の要素であるユーザーキーを使用して暗号化されている。 ユーザ鍵は、ユーザパスフレーズのハッシュを計算することによって生成される。 ユーザーキーはコンテンツキーを解読するために使用され、コンテンツキーは出版物リソースを解読するために使用される。

さらに、ライセンス文書には、外部リソースへのリンク、ユーザーを特定する情報、およびユーザーに与えられる権利と与えられない権利に関する情報が含まれる。 権利情報には、ライセンスの有効期間、本の印刷またはコピーの可否などが含まれる。最後に、ライセンス文書には、そのコンポーネントの変更を防ぐための電子署名が常に含まれている。

図1は、LCPのさまざまなコンポーネント間の関係を示す。

![LCP components](/images/LCP_Archi_1.png)

### 出版物の保護

出版物を保護するために、プロバイダは次の手順を実行する。

1. 出版物に対する一意のコンテンツキーを生成する。

2. この出版物のライセンス文書を将来生成するため、このコンテンツキーを保存する。

3. 保護すべき各リソースを（必要なら圧縮したのち）そのキーを使用して暗号化する。

4. 保護されたリソースをコンテナに追加し、保護されていないバージョンを置き換える。

5. 保護された各リソースのEncryptedData要素を含む`META-INF/encryption.xml`文書（セクション2.2で説明する）を作成する。このEncryptedData要素はつぎのものを含む。

    a. 使用されるアルゴリズムを示すEncryptionMethod要素

    b. KeyInfo要素であって、ライセンス文書のコンテンツキーを参照するRetrievalMethod子を持つもの

    c. 保護されたリソースを特定するCipherData要素

6. コンテナに`META-INF/encryption.xml`を追加する。

これで出版物は保護されており（つまり、保護された出版物になっており）、1人以上のユーザーにライセンスを供与する準備ができた状態になる。

### 出版物にたいするライセンスの提供

保護された出版物をユーザーが要求したときは、プロバイダは次の手順に従って、保護された出版物のライセンスを取得する。

1. ユーザーのパスフレーズをハッシュしてユーザーキーを生成する（4.2節を参照）。 ユーザーとそのユーザーのパスフレーズは、既にプロバイダに知られているものとする。

2. ユーザーキーを使用して、保護されたパブリケーションのコンテンツキーを暗号化する。

3. 次の内容を持つライセンス文書(`META-INF/license.lcpl`)を作成する（第3章を参照）。

    a. このライセンスの一意のID

    b. ライセンス発行日

    c. コンテンツプロバイダを特定するURI

    d. 暗号化されたコンテンツキー

    e. ユーザーキーに関する情報

    f. 保護された出版物でもライセンス文書でもない場所に保存された追加情報へのリンク（オプション）

    g. ユーザーに付与される特定の権利に関する情報（オプション）

    h. ユーザーを識別する情報（オプション）。 このセクションのいくつかのフィールドは、ユーザーキーを使用して暗号化される場合がある。

4. ライセンス文書データのデジタル署名を生成し、ライセンス文書に追加する。

ライセンス文書と保護された文書をユーザーに配布するには2つの方法がある。

1. **保護された出版物の内部に含まれるライセンス文書:**  プロバイダは、保護れた出版物のコンテナにライセンス文書を追加してから、出版物をユーザーに配信する。

2. **切り離されて提供されるライセンス文書:** プロバイダは、保護された文書へのリンクをライセンス文書に埋め込み、ライセンス文書のみをユーザに配信する。 ライセンス文書を処理するリーディングシステムは、保護された出版物を取り出し、そのコンテナにライセンス文書を追加する。

いずれの方法を使用する場合でも、リーディングシステムには、保護された出版物とライセンス文書を含むEPUBコンテナが提示される。

### 出版物を読む

保護された出版物を復号化してレンダリングするために、ユーザーのリーディングシステムは次の手順に従う。

1. ライセンス文書の署名を確認する。

2. ユーザキーを取得する（すでに格納されている場合）か、ユーザパスフレーズをハッシュしてユーザキーを生成する。

3. ユーザーキーを使用してコンテンツキーを復号化する。

4. コンテンツキーを使用して保護されたリソースを復号化する。

## 1.4 適合性についての表明

この文書中のキーワード<b>MUST</b>, <b>MUST NOT</b>, <b>REQUIRED</b>, <b>SHALL</b>, <b>SHALL NOT</b>, <b>SHOULD</b>, <b>SHOULD NOT</b>, <b>RECOMMENDED</b>, <b>MAY</b>, 及び<b>OPTIONAL</b>は[[RFC2119](#normative-references)]に従って解釈される。

この仕様のすべての節は規定であるが、「この節は参考である」という参考ステータスレベルがあればその限りではない。節および付録に参考ステータスが適用されていれば、その子である内容及びサブセクション（もしあれば）にも適用される。

この仕様のすべての例は参考である。

# 2. 出版物

## 2.1. 導入

{:.information}
**この節は参考である。**

LCPは[[OCF](#normative-references)]仕様に従い、EP​​UBコンテナ内の暗号化されたリソースを特定する。具体的には、LCPは`META-INF/encryption.xml`を使用して、パッケージ文書に列挙されているどのリソースが暗号化されているかを示し、コンテンツキーを参照する。

## 2.1. 暗号化されたリソース

出版物が保護されているかどうかリーディングシステムが確実に判定できるように、いくつかのリソースの暗号化は禁止されている。

[[OCF](#normative-references)]仕様において以下のファイルの暗号化は禁止されており、LCPはそれを踏襲している。

* `mimetype`

* `META-INF/container.xml`

* `META-INF/encryption.xml`

* `META-INF/manifest.xml`

* `META-INF/metadata.xml`

* `META-INF/rights.xml`

* `META-INF/signatures.xml`

* EPUB `rootfiles` （すべてのレンディションのパッケージ文書）

さらに、この仕様では、以下のファイルを暗号化してはならない（MUST NOT）と規定している。

* `META-INF/license.lcpl`

* 出版物のいずれかのパッケージ文書から参照されるナビゲーション文書（パブリケーションマニフェストに載っている出版物ソースであって["nav"プロパティ](https://www.idpf.org/epub/30/spec/epub30-publications.html#sec-item-property-values))を持つもの） 

* 出版物のいずれかのパッケージ文書から参照されるNCX文書（パブリケーションマニフェストに記載されているすべてのパブリケーションリソースであってメディアタイプ "application/x-dtbncx+xml"を持つもの）

* カバー画像（パブリケーションマニフェストに記載されているパブリケーションリソースであって、["cover-image" プロパティ](https://www.idpf.org/epub/30/spec/epub30-publications.html#sec-item-property-values)を持つもの）

## 2.2. META-INF/encryption.xmlのLCPへの利用

[[OCF](#normative-references)]仕様で定義されている通り、どの出版物リソースが暗号化されているかを明示しなければならない。明示は、[[XML-ENC](#normative-references)]を用いて公知の`META-INF/encryption.xml`ファイルで行う。

LCPを使用して保護された出版物では、これらのリソースを暗号化するために使用されるキー（LCPコンテンツキー）を特定するための追加要件がある。

1. `ds:KeyInfo`要素は、`ds:RetrievalMethod`要素を使用してコンテンツキーを参照しなければならない（MUST）。


2. `ds:RetrievalMethod`の`URI`属性は、"license.lcpl#/encryption/content_key"という値を使用して、ライセンス文書に格納されている暗号化されたコンテンツキーを参照しなければならない（MUST）。このURIは[[JSON Pointer](#normative-references)]仕様に従う。

3. `Type`属性は、"`http://readium.org/2014/01/lcp#EncryptedContentKey`"という値を使用して、URIのターゲットが暗号化されたコンテンツキーであることを指定しなければならない（MUST）。

*次の例（[[OCF](#normative-references)]から借用したもの）では、image.jpegリソースはコンテンツキーを使用してAESで暗号化されている。*

```xml
<encryption
    xmlns ="urn:oasis:names:tc:opendocument:xmlns:container"
    xmlns:enc="http://www.w3.org/2001/04/xmlenc#"
    xmlns:ds="http://www.w3.org/2000/09/xmldsig#">

    <enc:EncryptedData Id="ED1">
        <enc:EncryptionMethod Algorithm="http://www.w3.org/2001/04/xmlenc#aes256-cbc"/>
        <ds:KeyInfo>
            <ds:RetrievalMethod URI="license.lcpl#/encryption/content_key"
                Type="http://readium.org/2014/01/lcp#EncryptedContentKey"/>
        </ds:KeyInfo>
        <enc:CipherData>
            <enc:CipherReference URI="image.jpeg"/>
        </enc:CipherData>
    </enc:EncryptedData>
</encryption>
```


# 3. ライセンス文書

## 3.1. 導入

{:.information}
****この節は参考である。**
`META-INF/encryption.xml`は、どのようにリソースが暗号化され、暗号化されたコンテンツキーがどこにあるかを記述するが、LCPに関するその他の関連情報はすべてライセンス文書に格納される。 

この仕様書は、ライセンス文書の構文、コンテナ内での場所、メディアタイプ、ファイル拡張子、および処理モデルを定義する。

## 3.2. コンテンツの適合性

ライセンス文書は、次のすべての基準を満たしていなければならない（MUST）。

文書のプロパティ

* [[JSON](#normative-references)]で定義されているJSON文書の適合性制約を満たさなければならない（MUST）。

* UTF-8を使用してエンコードしなければならない（MUST）。

ファイルのプロパティ

* ファイル名はファイル拡張子`.lcpl`を用いなければならない（MUST）。

* MIMEメディアタイプは`application/vnd.readium.lcp.license-1.0+json`である。

* コンテナ内での場所は、`META-INF/license.lcpl`でなければならない（MUST）。

## 3.3. コアライセンス情報

ライセンス文書には、次の名前と値のペアが含まれていなければならない（MUST）。

<table class="table-bordered large">
	<tr>
	  <th>名</th>
	  <th>値</th>
	  <th>フォーマット/データタイプ</th>
	</tr>
	<tr>
	  <td>id</td>
	  <td>ライセンスの一意の識別子</td>
	  <td>文字列</td>
	</tr>
	<tr>
	  <td>issued</td>
	  <td>ライセンスが最初に発行された日付</td>
	  <td>ISO 8601</td>
	</tr>
	<tr>
	  <td>provider</td>
	  <td>プロバイダ固有の識別子</td>
	  <td>URI</td>
	</tr>
</table>

<br>

さらに、ライセンス文書は以下の名前と値のペアを含んでいてもよい（MAY）。

<table class="table-bordered large">
	<tr>
	  <th>名前</th>
	  <th>値</th>
	  <th>フォーマット/データタイプ</th>
	</tr>
	<tr>
	  <td>updated</td>
	  <td>ライセンスが最後に更新された日付</td>
	  <td>ISO 8601</td	>
	</tr>
</table>


## 3.4. キーの配送:  encryptionオブジェクト

コアライセンス情報に加えて、ライセンス文書には、次の名前と値のペアを持つ`encryption`オブジェクトが含まれていなければならない（MUST）。

<table class="table-bordered large">
  <thead>
    <tr>
      <th>名前</th>
      <th>値</th>
      <th>フォーマット/データタイプ</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>profile</td>
      <td>このLCP保護された出版物で使用されている暗号化プロファイルを指定する</td>
      <td>URI</td>
    </tr>
  </tbody>
</table>

<br />
`encryption`オブジェクトは、`content_key`と`user_key`の2つのオブジェクトも含まなければならない（MUST）。 

`encryption/content_key`オブジェクトには、出版物リソースを暗号化するために使用されるコンテンツキー（ただしユーザキーを使用して暗号化されているもの）が含まれている。このオブジェクトは、次の名前と値のペアを含まなければならない（MUST）。

<table class="table-bordered large">
  <thead>
    <tr>
      <th>名前</th>
      <th>値</th>
      <th>フォーマット/データタイプ</th>
    </tr>
  </thead>
  <tbody>
  	<tr>
	    <td>encrypted_value</td>
	    <td>暗号化されたコンテンツキー</td>
	    <td>Base 64で符号化されオクテットシーケンス</td>
	  </tr>
	  <tr>
	    <td>algorithm</td>
	    <td>コンテンツキーを暗号化するために使用されるアルゴリズム。[XML-ENC]で定義されたURIを使用して特定される。これは、`encryption/profile`で特定された暗号化プロファイルで指定されたコンテンツキー暗号化アルゴリズムと一致しなければならない（MUST）。</td>
	    <td>URI</td>
	  </tr>
  </tbody>
</table>

<br />
`encryption/user_key`オブジェクトには、コンテンツキーを暗号化するために使用されたユーザーキーに関する情報が含まれている。このオブジェクトは、次の名前と値のペアを含まなければならない（MUST）。

<table class="table-bordered large">
  <thead>
   <tr>
     <th>名前</th>
     <th>値</th>
     <th>フォーマット/データタイプ</th>
   </tr>
  </thead>
  <tbody> 
   <tr>
     <td>text_hint</td>
     <td>ユーザーにユーザーパスフレーズを覚えておくのに役立つヒント</td>
     <td>文字列</td>
   </tr>
   <tr>
     <td>algorithm</td>
     <td>ユーザパスフレーズからユーザ鍵を生成するために使用されるアルゴリズムであり、[XML-ENC]で定義されているURIを用いて指定する。これは、`encryption/profile`で特定された暗号化プロファイルで指定されたユーザ鍵ハッシュアルゴリズムと一致しなければならない（MUST）。</td>
     <td>URI</td>
   </tr>
   <tr>
     <td>key_check</td>
     <td>ライセンス文書の`id`フィールドの値。ユーザーキーと、`algorithm/content_key/algorithm`におけるコンテンツキーの暗号化と同じアルゴリズムとを用いて暗号化されている。これは、リーディングシステムが正しいユーザーキーを保持していることを確認するために使用される。 </td>
     <td>Base 64 encoded octet sequence</td>
   </tr>
  </tbody>
</table>

<br />
*この例は、LCP 1.0の基本暗号化プロファイルを使用するライセンス文書の暗号化情報を示している。*

```json
{
  "id": "ef15e740-697f-11e3-949a-0800200c9a66",
  "issued": "2013-11-04T01:08:15+01:00",
  "updated": "2014-02-21T09:44:17+01:00",
  "provider": "https://www.imaginaryebookretailer.com",
  "encryption": {
    "profile": "http://readium.org/lcp/basic-profile",
    "content_key": {
      "encrypted_value": "/k8RpXqf4E2WEunCp76E8PjhS051NXwAXeTD1ioazYxCRGvHLAck/KQ3cCh5JxDmCK0nRLyAxs1X0aA3z55boQ==",
      "algorithm": "http://www.w3.org/2001/04/xmlenc#aes256-cbc"
    },
    "user_key": {
      "text_hint": "Enter your email address",
        "algorithm": "http://www.w3.org/2001/04/xmlenc#sha256",
        "key_check": "jJEjUDipHK3OjGt6kFq7dcOLZuicQFUYwQ+TYkAIWKm6Xv6kpHFhF7LOkUK/Owww"
    }
  },
  "links": "...",
  "rights": "...",
  "signature": "..."
}
```


## 3.5. 外部リソースの参照: linksオブジェクト

ライセンス文書は、`links`オブジェクトも含まなければならない（MUST） 。これは、ローカル環境下にないリソースにライセンス文書を関連付けるために使用される。

`links`オブジェクトにネストされた各リンクオブジェクトは、リンクURIとリンク関係を含み、その他のリンクプロパティのセットを含んでいてもよい。

### リンク関係

この仕様では、各リンクオブジェクトに次のリンク関係を導入する。

<table class="table-bordered large">
  <tr>
    <th>関係</th>
    <th>意味</th>
    <th>必須かどうか</th>
  </tr>
  <tr>
    <td>hint</td>
    <td>ユーザーパスフレーズについての追加情報をユーザが探しているなら、リーディングシステムがユーザーをリダイレクトできる場所</td>
    <td>必須</td>
  </tr>
  <tr>
    <td>publication</td>
    <td>ライセンス文書に関連付けられた出版物をダウンロードできる場所</td>
    <td>必須</td>
  </tr>
  <tr>
    <td>self</td>
    <td>リンク関係のIANAレジストリで定義されているように、「リンクの文脈の識別子を伝える」</td>
    <td>必須ではない</td>
  </tr>
  <tr>
    <td>support</td>
    <td>ユーザーサポートのためのリソース（Webサイト、電子メール、または電話番号のいずれか）</td>
    <td>必須ではない</td>
  </tr>
</table>

<br />

これらのリンク関係に加えて、この仕様は[LCP Link Relations Registry](#informative-references)を導入する。ライセンス文書で使用され、正式なLCP仕様書で宣言されているすべての正式なリンク関係は、レジストリで参照されなければならない 。

リンク関係は、ベンダー固有のアプリケーションのために拡張することもできる（MAY）。そのようなリンクは、文字列ではなくURIを使用して、リンク関係を明示しなければならない（MUST）。


### リンクオブジェクト

各リンクオブジェクトは、次のキーをサポートしている。

<table class="table-bordered large">
  <tr>
    <th>名前</th>
    <th>値</th>
    <th>フォーマット/データタイプ</th>
    <th>必須かどうか?</th>
  </tr>
  <tr>
    <td>href</td>
    <td>リンクする場所</td>
    <td>URI or URIテンプレート</td>
    <td>必須</td>
  </tr>
  <tr>
    <td>rel</td>
    <td>リンクの関係</td>
    <td>リンク関係を表す公知の値、またはは拡張を表すURI</td>
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
    <td>外部リソースのMIMEメディアタイプの値として期待しているもの</td>
    <td>MIMEメディアタイプ</td>
    <td>必須ではないが、強く推奨される</td>
  </tr>
  <tr>
    <td>templated</td>
    <td>hrefがURIテンプレートであることを示す。</td>
    <td>論理値</td>
    <td>必須ではない、省力すると"false"と見なされる</td>
  </tr>
  <tr>
    <td>profile</td>
    <td>外部リソースを指定するために使用するプロファイル</td>
    <td>URI</td>
    <td>必須ではない</td>
  </tr>
  <tr>
    <td>length</td>
    <td>コンテンツの長さ（オクテット単位）</td>
    <td>整数</td>
    <td>必須ではない</td>
  </tr>
  <tr>
    <td>hash</td>
    <td>リソースのSHA-256ハッシュ</td>
    <td>Base 64で符号化されたオクテットシーケンス</td>
    <td>必須ではない</td>
  </tr>
</table>

<br />

テンプレート化されたURIは[[URI-Template]](#informative-references)仕様に従う。

*この例では、ライセンス文書は出版物を参照し、ユーザーパスフレーズに関するヒントの場所を含み、拡張機能を使用して認証および推奨サービスを提供している。

```json
{
  "id": "ef15e740-697f-11e3-949a-0800200c9a66",
  "issued": "2013-11-04T01:08:15+01:00",
  "updated": "2014-02-21T09:44:17+01:00",
  "provider": "https://www.imaginaryebookretailer.com",
  "encryption": "...",
  "links": [
    { "rel": "publication",
      "href": "https://www.example.com/file.epub",
      "type": "application/epub+zip",
      "length": "264929",
      "hash": "8b752f93e5e73a3efff1c706c1c2e267dffc6ec01c382cbe2a6ca9bd57cc8378"
    },
    { "rel": "hint",
      "href": "https://www.example.com/passphraseHint?user_id=1234",
      "type": "text/html"
    },
    { "rel": "support",
      "href": "mailto:support@example.org"
    }, 
    { "rel": "support",
      "href": "tel:1800836482"
    },
    { "rel": "support",
      "href": "https://example.com/support",
      "type": "text/html"
    }, 
    { "rel": "https://mylcpextension.com/authentication",
      "href": "https://www.example.com/authenticateMe",
      "title": "Authentication service",
      "type": "application/vnd.myextension.authentication+json"
    },
    { "rel": "https://mylcpextension.com/book_recommendations",
      "href": "https://www.example.com/recommended/1", 
      "type": "text/html"
    },
    { "rel": "https://mylcpextension.com/book_recommendations",
      "href": "https://www.example.com/recommended/1.opds",
      "type": "application/atom+xml; profile=opds-catalog; kind=acquisition"}
  ], 
  "rights": "...",
  "signature": "..."
}
```


## 3.6. 権利と制限を特定する: rightsオブジェクト

ライセンス文書は、`rights`オブジェクトを使用して一連の権利を表現してもよい（MAY）。`rights`オブジェクトは以下のフィールドを含んでもよい（MAY）。

<table class="table-bordered large">
  <tr>
    <th>名前</th>
    <th>値</th>
    <th>フォーマット/データタイプ</th>
    <th>デフォルト</th>
  </tr>
  <tr>
    <td>print</td>
    <td>ライセンスの有効期間中に印刷可能な最大ページ数</td>
    <td>整数</td>
    <td>無制限</td>
  </tr>
  <tr>
    <td>copy</td>
    <td>ライセンスの有効期間中にクリップボードにコピーできる最大文字数</td>
    <td>整数</td>
    <td>無制限</td>
  </tr>
  <tr>
    <td>start</td>
    <td>ライセンス開始日時</td>
    <td>ISO 8601の日付</td>
    <td>なし（無制限のライセンス）</td>
  </tr>
  <tr>
    <td>end</td>
    <td>ライセンス終了日時</td>
    <td>ISO 8601の日付</td>
    <td>なし（無制限のライセンス）</td>
  </tr>
  <tr>
    <td>[拡張URI]</td>
    <td>[実装者による定義]</td>
    <td>[実装者による定義]</td>
    <td>[実装者による定義]</td>
  </tr>
</table>

<br/>
データ型として整数を使用するすべての名前と値のペア（この仕様のprintとcopy）は、値の先頭に0を使用してはならない（MUST NOT）（例えば、“09”ではなく"9"）。

`print`権については、次のようにページを定義する。

1. 固定レイアウトの場合は、出版物で定義されたページ

2. EPUBナビゲーション文書の[page-list nav要素](https://www.idpf.org/epub/30/spec/epub30-contentdocs.html#sec-xhtml-nav-def-types-pagelist)によって定義されたページ（存在する場合）

3. 他のすべての場合では、1024個のUnicode文字

`copy`権は、クリップボードにコピーする機能のみを対象とし、テキスト（イメージではない）に制限されている。

これらの権利に加えて、この仕様では[LCP Rights Registry](#informative-references)を導入する。ライセンス文書で使用され、公式のLCP仕様書で宣言されたすべての公式の権利情報は、レジストリで参照されなくてはならない（MUST） 。

`rights`オブジェクトは、実装者固有の権利をいくつでも含むように拡張することができる（MAY）。拡張した権利は 、実装者が決めるURIを用いて指定する（MUST）。

*この例では、ライセンス文書は以下の権利を付与している。印刷することはいっさい許可されず、この本について一回だけ最大2048文字までのコピーが許容され、ライセンスには有効期限がある。 この本の一部をtwitterに流す権利を付与するベンダー拡張もある。*

```json
{
  "id": "ef15e740-697f-11e3-949a-0800200c9a66",
  "issued": "2013-11-04T01:08:15+01:00",
  "updated": "2014-02-21T09:44:17+01:00",
  "provider": "https://www.imaginaryebookretailer.com",
  "encryption": "...",
  "links": "...",
  "rights": {
    "print": 0,
    "copy": 2048,
    "start": "2013-11-04T01:08:15+01:00",
    "end": "2013-11-25T01:08:15+01:00",
    "https://www.imaginaryebookretailer.com/lcp/rights/tweet": true
  },
  "signature": "..."
}
```

## 3.7. ユーザの特定: userオブジェクト

ライセンス文書は、ユーザについての情報を`user`オブジェクトを使用して埋め込むことができる（MAY）。`user`オブジェクトには次のフィールドがある。

<table class="table-bordered large">
  <tr>
    <th>名前</th>
    <th>値</th>
    <th>フォーマット/データタイプ</th>
    <th>必須かどうか</th>
  </tr>
  <tr>
    <td>id</td>
    <td>特定のプロバイダのユーザを一意に識別する識別子</td>
    <td>文字列</td>
    <td>必須ではないが、強く推奨する</td>
  </tr>
  <tr>
    <td>email</td>
    <td>ユーザの電子メールアドレス</td>
    <td>文字列</td>
    <td>必須ではない</td>
  </tr>
  <tr>
    <td>name</td>
    <td>ユーザの名前</td>
    <td>文字列</td>
    <td>必須ではない</td>
  </tr>
  <tr>
    <td>encrypted</td>
    <td>このライセンス文書で暗号化されているユーザオブジェクト値のリスト</td>
    <td>上記の名前または拡張子に一致する1つ以上の文字列からなる配列</td>
    <td>暗号化がいずれかのフィールドで用いられているなら必須</td>
  </tr>
  <tr>
    <td>[拡張URI]</td>
    <td>[実装者による定義]</td>
    <td>[実装者による定義]</td>
    <td>[実装者による定義]</td>
  </tr>
</table>

<br />
これらのユーザー情報に加えて、この仕様は[LCP User Fields Registry](#informative-references)を導入する。ライセンス文書で使用され、正式なLCP仕様書で宣言されているすべての正式なユーザーフィールドは、このレジストリで参照されなければならない（MUST）。

権利と同様に、`user`オブジェクトを拡張して任意の数の実装者固有のフィールドを含めてもよい（MAY）。各拡張フィールドは、実装者によって制御されるURIによって指定されなければならない（MUST）。

ユーザの個人情報を保護するために、これらのフィールドのいずれかを暗号化してもよい（MAY）。ただし、`encrypted`フィールドは平文のままでなければならない（MUST）。暗号化する場合、フィールド値は、ユーザキーと、`encryption/content_key`オブジェクトで指定されるものと同じ暗号化アルゴリズムを使用して暗号化しなければならない（MUST）。暗号化されたフィールドすべてのの名前を、`encrypted`配列に列挙しなければならない（MUST）。

*次の例では、ユーザID、プロバイダ、および電子メールが示されている。ユーザーが優先する自然言語を示す拡張子もある。電子メールは暗号化されている。*

```json
{
  "id": "ef15e740-697f-11e3-949a-0800200c9a66",
  "issued": "2013-11-04T01:08:15+01:00",
  "updated": "2014-02-21T09:44:17+01:00",
  "provider": "https://www.imaginaryebookretailer.com",
  "encryption": "...",
  "links": "...",
  "rights": "...",
  "user": {
    "id": "d9f298a7-7f34-49e7-8aae-4378ecb1d597",
    "email": "EnCt2b8c6d2afd94ae4ed201b27049d8ce1afe31a90ceb8c6d2afd94ae4ed201b2704RjkaXRveAAarHwdlID1KCIwEmS",
    "encrypted": ["email"],
      "https://www.imaginaryebookretailer.com/lcp/user/language": "tlh"
  },
  "signature": "..."
}
```

## 3.8. ライセンスへの署名: signaturesオブジェクト

[5. 5. 署名とPKI(公開鍵基盤)](#signature-and-public-key-infrastructure)で説明されている通り、ライセンス文書には変更されていないことを検証するためのデジタル署名が含まれている。ライセンス文書は、`signature`オブジェクトを使用して署名についての情報を含まなければならない（MUST）。 `signature`オブジェクトは、以下のフィールドを含まなければならない（MUST）。

<table class="table-bordered large">
  <tr>
    <th>名前</th>
    <th>値</th>
    <th>フォーマット/データタイプ</th>
  </tr>
  <tr>
    <td>algorithm</td>
    <td>識別された署名を計算するために使用されるアルゴリズムであり、[XML-SIG]で指定されたURIを用いて指定される。`encryption/profile`で指定された暗号化プロファイルで指定された署名アルゴリズムと一致しなければならない。</td>
    <td>URI</td>
  </tr>
  <tr>
    <td>certificate</td>
    <td>プロバイダ証明書：コンテンツプロバイダが使用するX509証明書</td>
    <td>Base 64で符号化されたDER形式の証明書</td>
  </tr>
  <tr>
    <td>value</td>
    <td>署名値</td>
    <td>Base 64で符号化されたオクテット列</td>
  </tr>
</table>

<br />

署名と証明書がどのように計算され、符号化され、処理されるべき（SHOULD）かの詳細については、[5. 署名とPKI(公開鍵基盤)](#signature-and-public-key-infrastructure)を参照。

*この例は、ライセンス文書の署名を示している。*

```json
{
  "id": "ef15e740-697f-11e3-949a-0800200c9a66",
  "issued": "2013-11-04T01:08:15+01:00",
  "updated": "2014-02-21T09:44:17+01:00",
  "provider": "https://www.imaginaryebookretailer.com",
  "encryption": "...",
  "links": "...",
  "rights": "...",
  "user": "...",
  "signature": {
    "algorithm": "http://www.w3.org/2001/04/xmldsig-more#rsa-sha256",
    "certificate": "MIIDEjCCAfoCCQDwMOjkYYOjPjANBgkqhkiG9w0BAQUFADBLMQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTETMBEGA1UEBxMKRXZlcnl3aGVyZTESMBAGA1UEAxMJbG9jYWxob3N0MB4XDTE0MDEwMjIxMjYxNloXDTE1MDEwMjIxMjYxNlowSzELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExEzARBgNVBAcTCkV2ZXJ5d2hlcmUxEjAQBgNVBAMTCWxvY2FsaG9zdDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAOpCRECG7icpf0H37kuAM7s42oqggBoikoTpo5yapy+s5eFSp8HSqwhIYgZ4SghNLkj3e652SALav7chyZ2vWvitZycY+aq50n5UTTxDvdwsC5ZNeTycuzVWZALKGhV7VUPEhtWZNm0gruntronNa8l2WS0aF7P5SbhJ65SDQGprFFaYOSyN6550P3kqaAO7tDddcA1cmuIIDRf8tOIIeMkBFk1Qf+lh+3uRP2wztOTECSMRxX/hIkCe5DRFDK2MuDUyc/iY8IbY0hMFFGw5J7MWOwZLBOaZHX+Lf5lOYByPbMH78O0dda6T+tLYAVzsmJdHJFtaRguCaJVtSXKQUAMCAwEAATANBgkqhkiG9w0BAQUFAAOCAQEAi9HIM+FMfqXsRUY0rGxLlw403f3YtAG/ohzt5i8DKiKUG3YAnwRbL/VzXLZaHru7XBC40wmKefKqoA0RHyNEddXgtY/aXzOlfTvp+xirop+D4DwJIbaj8/wHKWYGBucA/VgGY7JeSYYTUSuz2RoYtjPNRELIXN8A+D+nkJ3dxdFQ6jFfVfahN3nCIgRqRIOt1KaNI39CShccCaWJ5DeSASLXLPcEjrTi/pyDzC4kLF0VjHYlKT7lq5RkMO6GeC+7YFvJtAyssM2nqunA2lUgyQHb1q4Ih/dcYOACubtBwW0ITpHz8N7eO+r1dtH/BF4yxeWl6p5kGLvuPXNU21ThgA==",
    "value": "q/3IInic9c/EaJHyG1Kkqk5v1zlJNsiQBmxz4lykhyD3dA2jg2ZzrOenYU9GxP/xhe5H5Kt2WaJ/hnt8+GWrEx1QOwnNEij5CmIpZ63yRNKnFS5rSRnDMYmQT/fkUYco7BUi7MPPU6OFf4+kaToNWl8m/ZlMxDcS3BZnVhSEKzUNQn1f2y3sUcXjes7wHbImDc6dRthbL/E+assh5HEqakrDuA4lM8XNfukEYQJnivqhqMLOGM33RnS5nZKrPPK/c2F/vGjJffSrlX3W3Jlds0/MZ6wtVeKIugR06c56V6+qKsnMLAQJaeOxxBXmbFdAEyplP9irn4D9tQZKqbbMIw=="
  }
}
```


# 4. ユーザーキー

## 4.1. 導入

{:.information}
**この節は参考である。**

対称暗号化されたメッセージが機能するためには、送信者と受信者が合意鍵を共有する必要がある。 LCPでは、これがユーザーキーである。 プロバイダは、ライセンス文書内に置くコンテンツキーを保護するために、ユーザ鍵にアクセスする必要がある。リーディングシステムは、ライセンス文書内のコンテンツキーを復号するために、ユーザーキーを必要とする。

LCPは、ユーザキーを共有するためパスフレーズモデルを用いる。単純な実装では、新しいライセンス文書をリーディングシステムが受信すると、コンテンツキーを得るためにユーザにパスフレーズの入力を要求する。ユーザキーはこのユーザパスフレーズのハッシュとして、LCPでは定義している。 このパスフレーズは、ユーザー定義のパスワード、プロバイダ定義のパスワード、電子メールアドレス、ライブラリカード番号など、何でも構わない。

## 4.2. ユーザキーの計算

ユーザーパスフレーズは 、UTF-8でエンコードされた文字列でなければならない（MUST）。ユーザーパスフレーズの長さや内容に制限はない。どのように作成するかについての制限は一切ない。

ユーザーキーは、ライセンス文書の`encryption/user_key`オブジェクトで指定されるハッシュアルゴリズムを使用して、ユーザーパスフレーズにハッシュ関数を適用した結果である。ハッシングの前に、空白のエスケープ化や正規化を含むどんな種類の処理もユーザパスフレーズに対して行わってはならない(SHALL NOT)。

## 4.3. ヒント

ユーザーがパスフレーズを入力するのを助けるため、リーディングシステムがパスフレーズのプロンプトまたはヒントをユーザーに提供する2つの方法を、LCPライセンス文書はサポートしている。

1. `user_key/text_hint`フィールドは、ユーザパスフレーズの入力のときユーザに表示できる簡単なプロンプトです（たとえば「仮想的な書店のパスワードを入力してください」）。

2. `rel=hint`である`links`オブジェクトは、パスフレーズのヒントやその他の支援を提供する箇所を参照する。

`user_key/text_hint`の内容も、`rel=hint`である`links`オブジェクトが指すリソースの内容も、人間に読めるものであるべき（SHOULD）であり、ユーザに向けてのものであり、ユーザがパスフレーズを入力するのを助けるべきである（SHOULD）。

## 4.4. ユーザキーとユーザパスフレーズへの要求

保護された出版物にアクセスするためのプロセスを簡素化するために、プロバイダーは同じユーザーに発行されたすべてのライセンスに対して同じユーザーキーを使用すべきである（SHOULD）。

LCPのセキュリティは、主にユーザーキーとユーザーパスフレーズのセキュリティ次第である。 したがって、ライセンスについてのワークフロー全体でこれらを保護するために特別な注意を払うべきである（SHOULD）。

1. パスフレーズは、総当たり攻撃を防ぐのに十分なぐらい複雑にすべきである（SHOULD）。

2. パスフレーズは平文で送信してはならない（MUST NOT）。

3. プロバイダが複数のシステム間でユーザキー情報を共有する必要がある場合、ユーザキーを送信すべきであり、ユーザパスフレーズを送信すべきではない（SHOULD）。送信は、安全なチャネルを用いて行うべきである(HOULD)。

4. ユーザーのパスフレーズは 、リーディングシステムによって保存されてはならない（MUST NOT）。リーディングシステムはユーザキーだけを格納すべきである（SHOULD）。


# 5. 署名とPKI(公開鍵基盤)

## 5.1. 導入

{:.information}
**この節は参考である。**

ライセンス文書内のさまざまなオブジェクトを正確に表現することの重要性を考えると、ライセンス文書の内容が真正で変更されていないことをリーディングシステムに確認可能なことが重要である。これは、[[X509](#normative-references)]で定義されている公開鍵基盤(PKI)を介して検証されるデジタル署名によって行われる。

署名の計算はバイトストリームに対して行われ、結果は一意に定まる。しかし、ライセンス文書はJSON文書であり、複数の表現が同じ構造を表すかも知れない。したがって、リーディングシステムとコンテンツプロバイダとの間の安定した署名を保証するためには、文書を署名する前と、署名を検証する前に、いくつかの変換を適用しなければならない。

プロバイダがライセンス文書に署名するために必要な(REQUIRED)手順は次のとおりである。

1. ライセンス文書の内容（署名オブジェクトを除いたもの）を正規形に変換する。重要でない空白を取り除いたアルファベット順にする（[5.3節](#canonical-form-of-the-license-document)を参照）。

2. 正規形のライセンス文書の署名値は、暗号化プロファイルで指定されたアルゴリズムに従って計算される。これには通常、データのハッシュを取得し、秘密鍵を使用してデータを暗号化することが含まれる。 

3. 署名値とプロバイダ証明書は、[3.8節](#signing-the-license-the-signature-object)で説明した通りに`signature`オブジェクトのライセンス文書に追加される。

リーディングシステムが署名を検証するには、次の手順を順番に実行する必要がある。

1. ライセンス認証局からルート証明書を取得する。この証明書は、すべてのプロバイダ証明書を検証するためにリーディングシステムに埋め込まれている。

2. このルート証明書を使用して、`signature`オブジェクトに含まれるプロバイダ証明書を検証する。これによって、プロバイダ以外の人がライセンス文書を偽造することを防止する。

3. ライセンス文書から`signature`オブジェクトを取り除く。

4. ライセンス文書の残りの部分を正規形にする。

5. 暗号化プロファイルで指定された署名アルゴリズムを使用して正規形のライセンス文書データのハッシュを計算する。

6. プロバイダ証明書で与えられた公開鍵を使用して署名オブジェクトに含まれている署名値を解読し、計算されたハッシュと一致することを確認する。これによって、ライセンス文書の内容が変更されていないことを確認する。

プロバイダ証明書がライセンス文書に含まれており、ルート証明書が読み取りシステムに組み込まれているため、インターネットに接続されていなくても検証プロセス全体を実行できる。

リーディングシステムは、プロバイダ証明書が失効していないことを確認するために、Readium Foundationが管理する証明書失効リストをチェックする。オフラインでの読書が可能なようにしてある。つまり、ライセンス文書を処理するたびにリーディングシステムは失効リストを確認する義務はない。インターネット接続が利用可能な場合（例えば、新しいライセンスまたはブックをダウンロードするたびに）定期的にリストを更新するだけでよい。

## 5.2. 証明書

### 5.2.1 プロバイダ証明書

ライセンス認証機関が発行し、ルート証明書を使用して署名した[[X509](#normative-references)]v3形式の証明書を、コンテンツプロバイダは保有しなければならない（MUST）。これは、ここではプロバイダ証明書と呼ぶ。プロバイダ証明書は、コンテンツプロバイダを表すべきである（SHOULD）。

コンテンツプロバイダは、発行するすべてのライセンス文書においてプロバイダ証明書を`signature/certificate`フィールドで配布しなければならない。また、プロバイダ証明書の公開鍵とペアになっている秘密鍵を使用して、ライセンス文書に署名しなければならない（MUST）。ライセンス文書が有効であるとみなされるためには、ライセンス文書が発行された時点（'issued'フィールドに示されているもの）でプロバイダ証明書が有効でなければならず（MUST）、プロバイダ証明書は取り消されいててはならない（MUST NOT）。

### 5.2.2 ルート証明書

リーディングシステムは、[[X509](#normative-references)] v3形式のルート証明書をライセンス認証機関から入手しなければならず、それを最新の状態に保つべきである（SHOULD）。オフラインでの使用の
ために、リーディングシステムにルート証明書を埋め込まなければならない（MUST）。

## 5.3. ライセンス文書の正規形

ライセンス文書の正規形は、署名の計算および検証時に使用される。正規形のライセンス文書を作成するには、以下のシリアライゼーション規則に従わなければならない。

1. これは計算の結果であるため、ライセンス文書の`signature`オブジェクトを削除する必要がある（MUST）。

2. ライセンス文書のすべてのオブジェクトメンバ（名前と値のペア）は、UTF-8（United Character Setコードポイント値）での表現に従って名前の辞書順にソートされなければならない（MUST）。 このルールは再帰的であるため、メンバはオブジェクトのネストのすべてのレベルでソートされる。

3. 配列内では、要素の順序を変更してはならない（MUST NOT）。

4. 数字には、有意でない先頭または末尾のゼロを含めてはならない（MUST NOT）。小数部（非整数）を含む数は、大文字の "E"を使用して数、小数部、および指数（正規化された科学記法）として表現しなくてはならない（MUST）。

5. 文字列のエスケープは、[[JSON](#normative-references)]によって必要(REQUIRED)とされる文字に限って行わなければならない。バックスラッシュ（\）、二重引用符（"）、および制御文字（U+0000からU+001Fである。制御文字をエスケープする場合、16進数は大文字でなければならない（MUST）。

6. 重要でない空白（[[JSON](#normative-references)]) <b>MUST</b>で定義されている）は削除しなければならない（MUST）。文字列内にある空白は保持しなければならない（MUST）。

### 5.3.1. 例

{:.information}
**この節は参考である。**

この例では、リンク（ヒント）と基本的なユーザー情報を含むライセンス文書を使用する。

```json
{
  "id": "ef15e740-697f-11e3-949a-0800200c9a66",
  "issued": "2013-11-04T01:08:15+01:00",
  "updated": "2014-02-21T09:44:17+01:00",
  "provider": "https://www.imaginaryebookretailer.com",
  "encryption": {
    "profile": "http://readium.org/lcp/basic-profile",
    "content_key": {
      "encrypted_value": "/k8RpXqf4E2WEunCp76E8PjhS051NXwAXeTD1ioazYxCRGvHLAck/KQ3cCh5JxDmCK0nRLyAxs1X0aA3z55boQ==",
      "algorithm": "http://www.w3.org/2001/04/xmlenc#aes256-cbc"
    },
    "user_key": {
      "text_hint": "Enter your email address",
        "algorithm": "http://www.w3.org/2001/04/xmlenc#sha256",
        "key_check": "jJEjUDipHK3OjGt6kFq7dcOLZuicQFUYwQ+TYkAIWKm6Xv6kpHFhF7LOkUK/Owww"
    }
  },
  "links": [
    { "rel": "hint", "href": "https://www.imaginaryebookretailer.com/lcp/hint", "type": "text/html"}
  ],
  "user": { "id": "d9f298a7-7f34-49e7-8aae-4378ecb1d597"}
}
```

まず、この文書をソートする。すべての[[JSON](#normative-references)]オブジェクトをソートする必要がある。

```json
{
  "encryption": {
    "content_key": {
      "algorithm": "http://www.w3.org/2001/04/xmlenc#aes256-cbc",
      "encrypted_value": "/k8RpXqf4E2WEunCp76E8PjhS051NXwAXeTD1ioazYxCRGvHLAck/KQ3cCh5JxDmCK0nRLyAxs1X0aA3z55boQ=="
    },
    "profile": "http://readium.org/lcp/basic-profile",
    "user_key": {
      "algorithm": "http://www.w3.org/2001/04/xmlenc#sha256",
      "key_check": "jJEjUDipHK3OjGt6kFq7dcOLZuicQFUYwQ+TYkAIWKm6Xv6kpHFhF7LOkUK/Owww",
      "text_hint": "Enter your email address"
    }
  },
  "id": "ef15e740-697f-11e3-949a-0800200c9a66",
  "issued": "2013-11-04T01:08:15+01:00",
  "links": [
    {
      "rel": "hint",
      "href": "https://www.imaginaryebookretailer.com/lcp/hint",
      "type": "text/html"
    }
  ],
  "provider": "https://www.imaginaryebookretailer.com",
  "updated": "2014-02-21T09:44:17+01:00",
  "user": {"id": "d9f298a7-7f34-49e7-8aae-4378ecb1d597"}
}
```

文書がソートされたので、すべての空白と行末を取り除くことができる。

```json
{"encryption":{"content_key":{"algorithm":"http://www.w3.org/2001/04/xmlenc#aes256-cbc","encrypted_value":"/k8RpXqf4E2WEunCp76E8PjhS051NXwAXeTD1ioazYxCRGvHLAck/KQ3cCh5JxDmCK0nRLyAxs1X0aA3z55boQ=="},"profile":"http://readium.org/lcp/basic-profile","user_key":{"algorithm":"http://www.w3.org/2001/04/xmlenc#sha256","key_check":"jJEjUDipHK3OjGt6kFq7dcOLZuicQFUYwQ+TYkAIWKm6Xv6kpHFhF7LOkUK/Owww","text_hint":"Enter your email address"}},"id":"ef15e740-697f-11e3-949a-0800200c9a66","issued":"2013-11-04T01:08:15+01:00","links":[{"rel":"hint","href":"https://www.imaginaryebookretailer.com/lcp/hint","type":"text/html"}],"provider":"https://www.imaginaryebookretailer.com","updated":"2014-02-21T09:44:17+01:00","user":{"id":"d9f298a7-7f34-49e7-8aae-4378ecb1d597"}}
```


## 5.4. 署名の生成

ライセンス文書に署名するには、コンテンツプロバイダは次の手順を順番に実行しなければならない（MUST）。

1. コンテンツプロバイダは、[5.3. ライセンス文書の正規形](#canonical-form-of-the-license-document)で与えられた規則に従って、正規形のライセンス文書を作成しなければならない（MUST）。

2. コンテンツプロバイダは 、この正規形のライセンス文書を使用し、`algorithm`フィールドに記述されているアルゴリズムとプロバイダ証明書の秘密鍵とを使用して、署名を計算しなければならない（MUST） 。

3. 署名は、Base64エンコーディングを使用して`signature`オブジェクトの`value`フィールドに埋め込まなければならない（MUST）。

4. 署名を検証するために使用したプロバイダ証明書は、`certificate`フィールドに挿入しなければならない（MUST）。DER表記を使用し、Base 64を使用してエンコードしなければならない（MUST）。

### 5.4.1. 例

{:.information}
**この節は参考である。**

正規形のライセンス文書が与えられたとする。

```json
{"encryption":{"content_key":{"algorithm":"http://www.w3.org/2001/04/xmlenc#aes256-cbc","encrypted_value":"/k8RpXqf4E2WEunCp76E8PjhS051NXwAXeTD1ioazYxCRGvHLAck/KQ3cCh5JxDmCK0nRLyAxs1X0aA3z55boQ=="},"profile":"http://readium.org/lcp/basic-profile","user_key":{"algorithm":"http://www.w3.org/2001/04/xmlenc#sha256","key_check":"jJEjUDipHK3OjGt6kFq7dcOLZuicQFUYwQ+TYkAIWKm6Xv6kpHFhF7LOkUK/Owww","text_hint":"Enter your email address"}},"id":"ef15e740-697f-11e3-949a-0800200c9a66","issued":"2013-11-04T01:08:15+01:00","links":[{"rel":"hint","href":"https://www.imaginaryebookretailer.com/lcp/hint","type":"text/html"}],"provider":"https://www.imaginaryebookretailer.com","updated":"2014-02-21T09:44:17+01:00","user":{"id":"d9f298a7-7f34-49e7-8aae-4378ecb1d597"}}
```

暗号化プロファイルで要求されている署名アルゴリズムを使用して、コンテンツプロバイダは最初にライセンス文書をハッシュし、次のバイトシーケンスを得る。ここでは16進数で表す。

```json
23c68442c7214ba294ddd1a2902756e9fe575116a88f36e55baf94590a90c2ad
```

このSHA-256形式を、コンテンツプロバイダーの秘密鍵を使用して署名する。Base 64で表現して次の結果が得られる。

```json
q/3IInic9c/EaJHyG1Kkqk5v1zlJNsiQBmxz4lykhyD3dA2jg2ZzrOenYU9GxP/xhe5H5Kt2WaJ/hnt8+GWrEx1QOwnNEij5CmIpZ63yRNKnFS5rSRnDMYmQT/fkUYco7BUi7MPPU6OFf4+kaToNWl8m/ZlMxDcS3BZnVhSEKzUNQn1f2y3sUcXjes7wHbImDc6dRthbL/E+assh5HEqakrDuA4lM8XNfukEYQJnivqhqMLOGM33RnS5nZKrPPK/c2F/vGjJffSrlX3W3Jlds0/MZ6wtVeKIugR06c56V6+qKsnMLAQJaeOxxBXmbFdAEyplP9irn4D9tQZKqbbMIw==
```

この署名と証明書により、有効なライセンスを作成することができる（MAY）。 

```json
{
  "date": "2013-11-04T01:08:15+01:00",
  "encryption": {
    "content_key": {
      "algorithm": "http://www.w3.org/2001/04/xmlenc#aes256-cbc",
      "encrypted_value": "/k8RpXqf4E2WEunCp76E8PjhS051NXwAXeTD1ioazYxCRGvHLAck/KQ3cCh5JxDmCK0nRLyAxs1X0aA3z55boQ=="
    },
    "profile": "http://readium.org/lcp/basic-profile",
    "user_key": {
      "algorithm": "http://www.w3.org/2001/04/xmlenc#sha256",
      "text_hint": "Enter your email address"
    }
  },
  "id": "ef15e740-697f-11e3-949a-0800200c9a66",
  "links": [
    { "rel": "hint",
      "href": "https://www.imaginaryebookretailer.com/lcp/hint",
      "type": "text/html"
    }
  ],  
  "user": {"id": "d9f298a7-7f34-49e7-8aae-4378ecb1d597"},
  "signature": {
    "algorithm": "http://www.w3.org/2001/04/xmldsig-more#rsa-sha256",
    "certificate": "MIIDEjCCAfoCCQDwMOjkYYOjPjANBgkqhkiG9w0BAQUFADBLMQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTETMBEGA1UEBxMKRXZlcnl3aGVyZTESMBAGA1UEAxMJbG9jYWxob3N0MB4XDTE0MDEwMjIxMjYxNloXDTE1MDEwMjIxMjYxNlowSzELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExEzARBgNVBAcTCkV2ZXJ5d2hlcmUxEjAQBgNVBAMTCWxvY2FsaG9zdDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAOpCRECG7icpf0H37kuAM7s42oqggBoikoTpo5yapy+s5eFSp8HSqwhIYgZ4SghNLkj3e652SALav7chyZ2vWvitZycY+aq50n5UTTxDvdwsC5ZNeTycuzVWZALKGhV7VUPEhtWZNm0gruntronNa8l2WS0aF7P5SbhJ65SDQGprFFaYOSyN6550P3kqaAO7tDddcA1cmuIIDRf8tOIIeMkBFk1Qf+lh+3uRP2wztOTECSMRxX/hIkCe5DRFDK2MuDUyc/iY8IbY0hMFFGw5J7MWOwZLBOaZHX+Lf5lOYByPbMH78O0dda6T+tLYAVzsmJdHJFtaRguCaJVtSXKQUAMCAwEAATANBgkqhkiG9w0BAQUFAAOCAQEAi9HIM+FMfqXsRUY0rGxLlw403f3YtAG/ohzt5i8DKiKUG3YAnwRbL/VzXLZaHru7XBC40wmKefKqoA0RHyNEddXgtY/aXzOlfTvp+xirop+D4DwJIbaj8/wHKWYGBucA/VgGY7JeSYYTUSuz2RoYtjPNRELIXN8A+D+nkJ3dxdFQ6jFfVfahN3nCIgRqRIOt1KaNI39CShccCaWJ5DeSASLXLPcEjrTi/pyDzC4kLF0VjHYlKT7lq5RkMO6GeC+7YFvJtAyssM2nqunA2lUgyQHb1q4Ih/dcYOACubtBwW0ITpHz8N7eO+r1dtH/BF4yxeWl6p5kGLvuPXNU21ThgA==",
    "value": "q/3IInic9c/EaJHyG1Kkqk5v1zlJNsiQBmxz4lykhyD3dA2jg2ZzrOenYU9GxP/xhe5H5Kt2WaJ/hnt8+GWrEx1QOwnNEij5CmIpZ63yRNKnFS5rSRnDMYmQT/fkUYco7BUi7MPPU6OFf4+kaToNWl8m/ZlMxDcS3BZnVhSEKzUNQn1f2y3sUcXjes7wHbImDc6dRthbL/E+assh5HEqakrDuA4lM8XNfukEYQJnivqhqMLOGM33RnS5nZKrPPK/c2F/vGjJffSrlX3W3Jlds0/MZ6wtVeKIugR06c56V6+qKsnMLAQJaeOxxBXmbFdAEyplP9irn4D9tQZKqbbMIw=="
  }
}
```


## 5.5. 証明書と署名の検証

#### **5.5.1. 証明書の検証**

1. ライセンス文書が最後に更新されたときに証明書が期限切れになっていないことを確認しなければならない（MUST）。

2. ルートチェーン内のプロバイダ証明書の存在を検証しなければならない（MUST）。そのためには、ルート証明書の公開鍵を使用してプロバイダ証明書の署名をチェックしなければならない（MUST）。

3. [[X509](#normative-references)]で定義される通りに、証明書が取り消されなかったことを確認しなければならない。 ネットワーク接続が利用可能な場合、証明書の有効性をチェックする前に、証明書失効リストを更新しなければならない（MUST）。

#### **5.5.2. 署名の検証**

署名を検証するには、次の手順を順番に実行しなければならない（MUST）。

1. リーディングシステムは 、ライセンス文書から署名を抽出して削除しなければならない（MUST） 。 

2. [5.3. ライセンス文書の正規形](#canonical-form-of-the-license-document)で示された規則に従い、ライセンス文書の正規形を計算しなければならない（MUST）。

3. [5.4. 署名の生成](#generating-the-signature)で定義された通りに署名を再計算しなければならない（MUST）。

4. 計算された署名値が、前もってライセンス文書から抽出したものと一致することを確認しなければならない（MUST）。

# 6. 暗号化プロファイル

## 6.1. 導入

{:.information}
**この節は参考である。**

LCPは、[[XML-ENC](#normative-references)]と[[XML-SIG](#normative-references)]で定義されている標準の暗号化アルゴリズムに完全に準拠している。柔軟性を最大にするため、この仕様では暗号化アルゴリズムを限定しない。保護された出版物を処理するとき、どの暗号化アルゴリズムが使われているのかリーディングシステムが発見できるように、`encryption.xml`とライセンス文書は設計されている。

この発見プロセスを簡単にするために、LCP 1.0仕様は暗号化プロファイルという機構を設け、保護された出版物および関連するライセンス文書で使用できる暗号化アルゴリズムを限定できるようにしている。ある暗号化プロファイルに定められたアルゴリズムを実装するリーディングシステムは、その暗号化プロファイルを使用して保護された出版物を復号できることが保証される。検出を容易にするため、どの暗号化プロファイルが使われているかはライセンス文書で明示される。

この仕様は、Basic Encryption Profile 1.0を定め[[XML-ENC](#normative-references)]及び[[XML-SIG](#normative-references)]からいくつかのアルゴリズムを選定する。すべての将来の公式拡張またはベンダー固有の拡張は、このような暗号化プロファイルを定義し、[LCP Encryption Profiles Registry](#informative-references)に公開する。リーディングシステムは、用いられている暗号化プロファイの内容を知ることができる。

## 6.2. 暗号化プロファイルへの要求

すべての暗号化プロファイルは 、以下の対象についてのアルゴリズムを指定しなければならない（MUST）。

1. 出版物リソース

2. コンテンツキーとユーザーフィールド（暗号化されている場合）

3. ユーザーパスフレーズ

4. 署名

暗号化プロファイルで使用されるすべてのアルゴリズムは[[XML-ENC](#normative-references)]または[[XML-SIG](#normative-references)]で定義されていなければならない（MUST）。

どの暗号化プロファイルが使用されているかを、（ライセンス文書の`encryption`オブジェクトに含まれる）`profile`内で示すためのURIを決めなければならない（MUST）。

すべての暗号化プロファイルは 、明示的にレジストリで説明されているように、LCP暗号化プロファイルレジストリに登録されなければならない（MUST）。

## 6.3. 基本暗号化プロファイル 1.0

基本暗号化プロファイル1.0が使用されていることは、ライセンス文書のencryptionオブジェクトで`profile`属性値としてURL `http://readium.org/lcp/basic-profile`を指定することによって明示する。

基本暗号化プロファイル1.0では、次のアルゴリズムを用いる。

<table class="table-bordered large">
  <tr>
    <th>対象</th>
    <th>アルごりズム(名前)</th>
    <th>アルゴリズム(URI)</th>
    <th>指定される場所</th>
  </tr>
  <tr>
    <td>出版物リソース</td>
    <td>AES 256 bits CBC</td>
    <td>http://www.w3.org/2001/04/xmlenc#aes256-cbc</td>
    <td>encryption.xml</td>
  </tr>
  <tr>
    <td>コンテンツキー、ユーザーフィールド（暗号化されている場合）</td>
    <td>AES 256 bits CBC</td>
    <td>http://www.w3.org/2001/04/xmlenc#aes256-cbc</td>
    <td>ライセンス文書</td>
  </tr>
  <tr>
    <td>ユーザーパスフレーズ</td>
    <td>SHA-256</td>
    <td>http://www.w3.org/2001/04/xmlenc#sha256</td>
    <td>ライセンス文書</td>
  </tr>
  <tr>
    <td>署名</td>
    <td>RSA with SHA-256</td>
    <td>http://www.w3.org/2001/04/xmldsig-more#rsa-sha256
</td>
    <td>ライセンス文書</td>
  </tr>
</table>


# 7. リーディングシステムの挙動

## 7.1 LCPで保護された出版物の検出

リーディングシステムは、LCPによって出版物が保護されていることを以下のいずれかの方法で検出できる。

1. ライセンス文書(`META-INF/license.lcpl`)が存在する。

2. `META-INF/encryption.xml`中にLCPコンテンツキーを参照する（すなわち`Type`属性値が"`http://readium.org/2014/01/lcp#EncryptedContentKey`"である）`ds:KeyInfo/ds:RetrievalMethod`要素が存在する。

`encryption.xml`がLCPコンテンツキーを参照しているが、ライセンス文書がない場合、リーディングシステムはこれをユーザに報告すべきである（SHOULD）。 

## 7.2. ライセンス文書の処理

### 概要

ライセンス文書の処理では、リーディングシステムは理解できないすべての名前/値ペアを無視しなければならない（MUST）。

リーディングシステムは、暗号化されていないバージョンのコンテンツキーおよび/または暗号化されたユーザフィールドを保管してはならない（MUST NOT）。 

### ライセンス文書の検証

リーディングシステムは次のことを行わなければならない（MUST）。

1. ライセンス文書の構文と完全性を検証する

2. [5.5. 証明書と署名の検証](#validating-the-certificate-and-signature)で定義されている通りに署名を検証する。

### 出版物の入手

保護された出版物とは別にライセンス文書が配信される場合、リーディングシステムはつぎのことを行わなければならない（MUST）。

1. `links/publication/href`で示されたURLから、保護された出版物をダウンロードする。

保護された出版物ファイルをユーザがアクセスできるようにするリーディングシステムは、ダウンロードされた保護された出版物にライセンス文書をファイル`META-INF/license.lcpl`として追加しなければならない（MUST）。

ハッシュが提供されている場合、ダウンロード済みの保護された出版物に整合性があるかどうかリーディングシステムは確認すべきである（SHOULD）。

保護された出版物を取得できなかった場合は、リーディングシステムは、ユーザーに失敗を報告すべきである（SHOULD）。

## 7.3. ユーザキーの処理

リーディングシステムは次のことを行わなければならない（MUST）。

* ユーザにユーザパスフレーズの入力を促すときに、テキストヒントとURLを表示する

リーディングシステムは次のことを行うべきである（SHOULD）。

* ユーザーキーを安全に保存する

* 保護された出版物のユーザーパスフレーズを入力するようにユーザーに促す前に、以前に格納されたユーザーキーを試す

リーディングシステムは次のことを行ってもよい（MAY）。

* 別の技術を用いて、バックグラウンドでユーザーキーを検出して交換する。

* ライセンス文書を処理するための正しいユーザーキーの検出を最適化するために、特定のユーザーキーを特定のプロバイダーとユーザーIDに関連付ける。

リーディングシステムは次のことを行ってはならない（MUST NOT）。

* ユーザパスフレーズを保存する（保存してよいのはユーザキーだけ）

## 7.4. 署名の処理

リーディングシステムは次のことを行わなければならない（MUST）。

* [5.5. 証明書と署名の検証](#validating-the-certificate-and-signature)に従って署名とプロバイダ証明書を検証する。

* 定期的に証明書失効リストを更新する。

リーディングシステムは次のことを行うべきである（SHOULD）。

* ルート証明書を更新する。

リーディングシステムは次のことを行ってもよい（MAY）。

* ルート証明書が非推奨であっても、出版物をあえて開く。

リーディングシステムは次のことを行ってはならない（MUST NOT）。

* 署名またはプロバイダ証明書が無効な出版物を開く。

* ネットワーク接続が利用できない場合、または証明書失効リストが入手できない場合に、ユーザーが出版物を開くのを妨げる。

## 7.5. 出版物の処理

リーディングシステムは次のことを行わなければならない（MUST）。

* ライセンス文書に記載されているすべての権利制限を尊重する。

リーディングシステムは次のことを行ってもよい（MAY）。

* ライセンスが失効した場合、保護された出版物を削除する。

リーディングシステムは次のことを行ってはならない（MUST NOT）。

* 暗号化されていない出版物リソースを格納する。

# 引用規格

[JSON] [The application/json Media Type for JavaScript Object Notation (JSON)](https://www.ietf.org/rfc/rfc4627).

[JSON Pointer] [JavaScript Object Notation (JSON) Pointer](https://tools.ietf.org/html/rfc6901).

[OCF] [Open Container Format 3.0.1](https://www.idpf.org/epub/301/spec/epub-ocf.html).

[Publications] [EPUB Publications 3.0.1](https://www.idpf.org/epub/301/spec/epub-publications.html).

[X509] [Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile](https://tools.ietf.org/html/rfc5280).

[XML-ENC] [XML Encryption Syntax and Processing Version 1.1](https://www.w3.org/TR/xmlenc-core1/).

[XML-SIG] [XML Signature Syntax and Processing (Second Edition)](https://www.w3.org/TR/xmldsig-core/).

[URI-Template] [URI Template](https://tools.ietf.org/html/rfc6570).

[RFC2119] [Key words for use in RFCs to Indicate Requirement Levels](https://tools.ietf.org/html/rfc2119).

# 参考資料

LCP Link Relations Registry

LCP Rights Registry

LCP User Fields Registry

LCP Encryption Profiles Registry



