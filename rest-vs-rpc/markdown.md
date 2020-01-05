
name: inverse
layout: true
class: center, middle, inverse
---

class: center, middle

# REST vs. RPC

2020/01/21 Tech-Tall

APD 木利友一

---
layout: false

# Agenda

1. RPC とはなにか
2. REST とはなにか
3. REST と RPC の違い

---

# はじめに

- ちょっとスライドを作りすぎてしまった感があります。
  - 議論の時間が少ないかもしれないので、ご了承ください。
- 技術選択や考え方については、宗教戦争じみた話になりがちです。
  - 個々人でそれぞれの理解・解釈を整理してもらえると嬉しいです。

---

# 今日のテーマの背景

前職で Cloud Provider としての API 設計を RESTful API として設計していたがつらかった

- 階層構造を持つリソースを扱うのがつらい
  - Network - Hypervisor - VLAN - VM ...
- 階層構造を持つリソースに対する操作を HTTP Method とマッピングしていくのがつらい
  - ネットワークの結線って、どのリソースに対する操作なんだろう
    - Network ? VLAN ? それとも、VM に対して新しく NIC というリソースを定義?
- PUT/POST/DELETE/GET を各リソース毎に実装していくのがつらい
  - 開発工数そのものがつらい
  - そもそも実装の必要があるか?という話はあるが

---

# REST ではない選択肢としての RPC

- Dropbox は API v2. で REST をやめ、実質的に RPC に移行した
  - [Scaling and securing the Dropbox API](https://getputpost.co/scaling-and-securing-the-dropbox-api-974daa4e96be)
      - But trying to force REST on something that doesn’t fit can result in pain, as we learned with the previous version of our API.
- DynamoDB 等の AWS サービスも RPC を採用しているものもある
  - [DynamoDB や Route53 などの AWS API が独特な仕様なので紹介](https://gist.github.com/voluntas/811240c5b6a169ae1c6ac401e0197417)
      - HTTP Header でメソッドを指定
          - `X-Amz-Target: DynamoDB_20120810.PutItem `

---

# mode 2 共有会

- RESTful API の設計 (とくにリソース設計)って結構むずかしいよねという話になった
- そういう議事録に対し、伊藤さんが techball の声をかけてくれた

---

# 最初に聞いておきたい

1. プロジェクトで API サーバを開発したことがある人?
2. API のスタイルとして Restful API を選んだ人?

---

# API のスタイル

- 世の中 Restful API が主流のように見える <sup>[要出典]</sup>
- 一方で、RPC 型のスタイルも見られるようになってきた。

---

# RPC 型の例

- [JSON-RPC](https://www.jsonrpc.org/)
- [XML-RPC](http://xmlrpc.com/)
- [gRPC](https://grpc.io/)

---

# 第3の道

- [GraphQL](https://graphql.org/)
- Others

---

# REST にするか、RPC にするか

T&I は技術的選択に責任を持つことも多いと思います。

- その技術選択にきちんとした理由付けを自分の中でしていますか?
- それをプロジェクトの関係者に説明できますか?

.footnote[ちなみに、ぼくはそこを説明したことはないです(プロジェクトに参画したとき、だいたい API のスタイルは決まっていた)]

---
template: inverse

# RPC とは何なのか

---
layout: false
## RPC

- `R`emote `P`rocedure `C`all
- [wikipedia (en)](https://en.wikipedia.org/wiki/Remote_procedure_call)
  - remote procedure call (RPC) is when a computer program causes a procedure (subroutine) to execute in a different address space (commonly on another computer on a shared network), which is coded as if it were a normal (local) procedure call, without the programmer explicitly coding the details for the remote interaction.
  - 他のコンピュータ上の procedure を (プログラマが明示的に気にすることなくあたかもローカルの procedure call のように)呼び出せる

Resource ではなく、**Procedure** を対象にしていることがポイント

---

それではいくつか具体的な例を見てみましょう

---

## 例: JSON-RPC

```json
POST /myservice HTTP/1.1
Host: rpc.example.com
Content-Type: application/json
Content-Length: ...
Accept: application/json

{
    "jsonrpc": "2.0",
    "method": "sum",
    "params": { "b": 34, "c": 56, "a": 12 },
    "id": 123
}
```

---

## 例: XML-RPC

```xml
POST /RPC2 HTTP/1.0
User-Agent: Frontier/5.1.2 (WinNT)
Host: betty.userland.com
Content-Type: text/xml
Content-length: 181

<?xml version="1.0"?>
<methodCall>
    <methodName>examples.getStateName</methodName>
    <params>
        <param>
            <value><i4>41</i4></value>
        </param>
    </params>
</methodCall>
```

---

# Fielding の言う RPC

Roy T. Fielding は REST の提唱者です。

![Roy T. Fielding](https://roy.gbiv.com/pics/roy_fielding.jpg)

[REST を提唱した論文](https://www.ics.uci.edu/~fielding/pubs/dissertation/evaluation.htm#sec_6_5)の中で、Fielding は RPC について以下のように記述しています。

- > What distinguishes RPC from other forms of network-based application communication is .red[**the notion of invoking a procedure on the remote machine**],
- > What makes HTTP significantly different from RPC is that .red[**_the requests are directed to resources using a generic interface with standard semantics_**] that can be interpreted by intermediaries almost as well as by the machines that originate services.

---
template: inverse

# RESTful API とは何なのか

---
layout: false
## REST

- `RE`epresentational `S`tate `T`ransfer
- [Roy T. Fielding](https://roy.gbiv.com/) の博士論文 ["Architectural Styles and the Design of Network-based Software Architectures"](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm) で定義

---

## REST

- WEB の｢アーキテクチャスタイル｣
  - 複数のアーキテクチャに共通する性質、様式、作法あるいは流儀を指す言葉
  - ｢クライアント・サーバ｣スタイルに対し、いくつかの制約が追加されたもの
- API の形態に REST を適用したものが RESTful API (という定義にします)
  - wikipedia では [RESTful Web Services](https://en.wikipedia.org/wiki/Representational_state_transfer) と呼んでいます

---

## REST における重要な概念

.left-column[
  ### Resource
]
.right-column[
> Any information that can be named can be a resource

WEB 上に存在する、**名前**を持ったありとあらゆる情報
]

---

## REST における重要な概念

.left-column[
  ### Resource
]
.right-column[
> Any information that can be named can be a resource

WEB 上に存在する、**名前**を持ったありとあらゆる情報

- 名前: URI
  - 他のリソースと区別するために一意の名前を持つ
- リソースには状態がある。
  - 状態が変化すると、リソースの表現も変化する。
]

---

## REST における重要な概念

.left-column[
  ### Resource
  ### Representation
]
.right-column[
> A representation is a sequence of bytes, plus representation metadata to describe those bytes.

- サーバとクライアントの間でやり取りするデータのこと
- `representation metadata` の例:
    - media type
    - last-modified time
- 表現形式はさまざま。
  - TXT、JSON、XML、etc.
]

---

# REST の導出

- REST というアーキテクチャスタイルは、既存の Client-Server に、さらに制約を課していったもの。
- 皆さんのこれまでの RESTful API はどこまでこれらの制約を満たしているでしょうか
  - 全部満たしていないと REST と呼べないわけではないです。

---

# REST の導出

.left-column[
  ### Client-Server
 ]

.right-column[
クライアントはサーバにリクエストを送り、サーバはそれに対してレスポンスを返す。

#### 利点

- クライアントをマルチプラットフォームにできる
- クライアントに UI を担当させることで、サーバ側をシンプルにできる
  - スケーラビリティの向上に寄与
- コンポーネント(クライアント、サーバそれぞれ)を独立して進化させられる
]

---

# REST の導出

.left-column[
  ### Client-Server
  ### Stateless
 ]

.right-column[
クライアントの状態をサーバで管理しない

#### 利点

- サーバ側の実装を簡略化できる
- リクエストに応えた後、サーバは計算リソースを開放可能 (scalability)
- 監視システムは、リクエストの内容さえ見ればその要求についてすべてわかる (visibility)
- 一応、博士論文には reliability の向上についても記載がありましたがよくわからん。

#### 欠点

- 同じ情報がリクエスト毎に繰り返される ((network) performance)
- クライアント側に状態を持つので、アプリの一貫した挙動を制御しにくい
  - 多様なクライアントに対応しないといけない
]

---

# REST の導出

.left-column[
  ### Client-Server
  ### Stateless
  ### Cacheability
 ]

.right-column[
リソースの鮮度に基づいて、一度取得したリソースをクライアント側で使い回す。

(→レスポンスには、キャッシュ可能か否かをラベル付けする必要がある)

#### 利点

- サーバ･クライアント間の通信削減によるネットワーク帯域･処理時間の削減 (efficiency、(user-perceived) performance)

#### 欠点

- 古いデータを取得しかねないことによる情報の信頼性の低下 (reliability)
]

---

# REST の導出

.left-column[
  ### Client-Server
  ### Stateless
  ### Cacheability
  ### Uniform Interface
 ]

.right-column[
URI で指し示したリソースに対する操作を、統一した限定的なインタフェースで行う。

- たとえば HTTP/1.1 では、Method は `GET` や `POST` などに限定される。

#### 利点

- インタフェースの柔軟性に制限を加えることで、全体のアーキテクチャがシンプルになる
- インタフェースの統一によりクライアント･サーバの実装の独立性が向上 (independent evolvability)

#### 欠点

- アプリケーションに最適化できない (efficiency)
]

---

# REST の導出

.left-column[
  ### Client-Server
  ### Stateless
  ### Cacheability
  ### Uniform Interface
  ### Layered System
 ]

.right-column[

各コンポーネントが、相互作用している直接のレイヤー以外を「見る」ことができない。
これにより、階層化アーキテクチャを構成できる。

#### 利点

- システム全体の複雑さが制限でき、各サブシステムの独立性を促進できる (independence)
- サーバ・クライアント間に LB や Proxy、Firewall 等を設置できる

#### 欠点

- データ処理のオーバヘッド、レイテンシの増大 ((user-perceived performance))
]

---

# REST の導出

.left-column[
  ### Client-Server
  ### Stateless
  ### Cacheability
  ### Uniform Interface
  ### Layered System
  ### Code on Demand (Optional)
 ]

.right-column[

プログラムコードをサーバからダウンロードし、クライアント側でそれを実行する。

例
- Applets (無言)
- Flash (無言)
- JavaScript ...?
  - REST の恩恵…なのか…? ホントに?

#### 利点

- クライアントの事前実装が不要となりクライアントを簡素化できる
- サービス展開後に機能をダウンロードできるようになるため拡張性が向上する

#### 欠点

- アプリケーションプロトコルの可視性が低下する
]

---

## REST のまとめ

Restful API を構築するためには、リソースとその状態、そしてその表現形式を定義する必要がある。

### アーキテクチャに加わる制約

- Client-Server
- Stateless
- Cacheability
- Uniform Interface
- Layered System
- Code on Demand (Optional)

---
template: inverse

# REST と RPC の違い、メリットとデメリット

---
layout: false

## 例: HTTP Method から見る REST、RPC の違い

- REST
  - PUT/GET/POST/DELETE/PATCH
- RPC:
  - XML-RPC: POST
  - JSON-RPC: (通常は) POST
  - gRPC: POST

## この違いはどこから来ているのか

- REST
  - Resource は操作対象
  - それに対する操作は HTTP Method でマッピングされる
    - CRUD に対応する HTTP Method を使用する必要がある
- RPC
  - "Procedure Call" であり、実施すべき操作は "Procedure" そのもので示されている
    - HTTP Method で操作を語る必要がない
    - だから、「リクエストの情報」さえ送信できれば、HTTP Method は何でも良い
      - 通常は POST で実装されるが、[JSON-RPC では GET で実装されるケースもある](https://www.simple-is-better.org/json-rpc/transport_http.html#get-request)?

---

## 例: URI に見る REST・RPC の違い

- REST: [Best Practices for Designing a Pragmatic RESTful API](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api) より
  - `GET /tickets`  - Retrieves a list of tickets
  - `GET /tickets/12` - Retrieves a specific ticket
  - `POST /tickets` - Creates a new ticket
  - `PUT /tickets/12` - Updates ticket #12
  - `PATCH /tickets/12` - Partially updates ticket #12
  - `DELETE /tickets/12` - Deletes ticket #12
- RPC:
  - `/myservice`
  - `/RPC2`

## この違いはどこから来ているのか

- REST:
  - 操作したい個々のリソースの URI を指定する
    - アドレス指定可能な単位はリソースである
  - システムの動作は、リソースの背後に隠される。
    - エンティティの作成、更新、または削除の副作用として
- RPC:
  - メソッドを呼び出すエンドポイントの URI を指定する
  - アドレス指定可能な単位はプロシージャ
    - 問​​題ドメインのエンティティはプロシージャの後ろに隠される

---
# RPC のメリデメ

### メリット

- API 設計の優先度はクライアント・サーバ双方のプログラミングの容易さと実行効率であり、RPC はそれに適合しやすい
  - API のプロシージャの学習は新しいプログラミングライブラリの学習と類似
  - RPC の実装も効率的である傾向がある
  - プログラマにとって、簡単かつ直接的
    - 1 つのプログラムでプロシージャを作成し、別のプログラムからプロシージャを呼び出すのは API に限ったことではない

### デメリット

- システム間の結合度を増す
  - 各サブシステムが「Procedure」で結合される。結果として、Procedure の持つ基本的な仮定が両システムで満たされなければならない。
- 個々の RPC それぞれを学ぶ必要があり、異なるサービスのプロシージャ間の共通性や予測可能性はほとんどない

---

## REST のメリデメ

### メリット

1. 各サービスの API に、共通性、予測可能性がある
   1. 操作 (HTTP Method) が限られるため、API 利用者からは何ができるのかが容易に理解できる
2. Procedure での結合ではなく問題ドメインを表現するエンティティの結合になるため、システム間の結合度を弱くできる
   1. だからこそ、任意のシステムとの連携がしやすい
   2. see: [結合度の概念モデル](https://ja.wikipedia.org/wiki/%E7%B5%90%E5%90%88%E5%BA%A6#%E7%B5%90%E5%90%88%E5%BA%A6%E3%81%AE%E6%A6%82%E5%BF%B5%E3%83%A2%E3%83%87%E3%83%AB)
3. REST 周りのエコシステムの発展
4. [それぞれのリソースが固有のURIを持っているので、キャッシュ、コピー、ブックマークすることが簡単にできる](https://ja.wikipedia.org/wiki/Representational_State_Transfer#REST%E5%AF%BERPC)
6. HTTP との親和性が高く、L7 で動作する LB、Proxy 等、HTTP の周りに発展してきた各種 SW/HW をそのまま利用できる
7. フロントエンド技術との親和性も高い

### デメリット

1. リソース設計が難しい
   1. 多段の階層関係の存在するリソースが出てくるとつらい。本当につらい
   2. [RESTful Web アプリの設計レビューの話](https://www.slideshare.net/t_wada/restful-web-design-review) by t_wada
)
2. 開発すべき API 数が膨大になる
   1. 各リソースに対して GET/POST (+PUT/DELETE) を必要とすることが多い
   2. 見積シートは API 本数が工数に効くので、適切に係数を調整しないと価格競争力がなくなる
3. 実装ルールが統一されていない (REST は仕様ではないので、言ったもの勝ち)

---

# なぜ REST が敬遠されてきているか

[クックパッドがgRPCを採用するまで
サービス間通信で抱えていた課題と、RubyでgRPCを運用するための工夫](https://logmi.jp/tech/articles/320715)より

> さっき「RESTfulなAPIを（Garageで）作れる」って言ったんですけど、RESTなエンドポイントへのマッピングが困難なケースが増えてきているんですね。
![gRPC in Cookpad](https://img.logmi.jp/articles/VyLoQyN5zX7u1D5vNjhMrJ.jpg)


---

# まとめ

- REST を選択することにより
  - システム間の結合度を弱くできる
  - フロントエンド技術や、HTTP エコシステムとの親和性も高い
  - ただし、それらは設計難易度、実装コストとのトレードオフがある
- RPC を選択することにより
  - 設計・実装は容易 (慣れている)
  - ただし、REST に比してシステム間の結合度は高くなる
- すべて REST でなくても良い文脈がある
  - マイクロサービス間通信で REST は必要か?

## 興味深い例

- Micro Services 間通信には gRPC が良く使われる
- その Micro Services を浮かべる Kubernetes の各コンポーネントは REST API で結合されている

---

# 参考文献

- [クックパッドがgRPCを採用するまで
サービス間通信で抱えていた課題と、RubyでgRPCを運用するための工夫](https://logmi.jp/tech/articles/320715)
- [なぜポストREST APIが求められるのか？　REST APIがカバーできない2つの要因とその対策](http://kageura.hatenadiary.jp/entry/2018/01/11/%E3%81%AA%E3%81%9C%E3%83%9D%E3%82%B9%E3%83%88REST_API%E3%81%8C%E6%B1%82%E3%82%81%E3%82%89%E3%82%8C%E3%82%8B%E3%81%AE%E3%81%8B%EF%BC%9F_REST_API%E3%81%8C%E3%82%AB%E3%83%90%E3%83%BC%E3%81%A7%E3%81%8D)
- [HTTP API の設計方向](https://medium.com/@voluntas/http-api-%E3%81%AE%E8%A8%AD%E8%A8%88%E6%96%B9%E5%90%91-7ccaca671d9d)
- [結合度の概念モデル](https://ja.wikipedia.org/wiki/%E7%B5%90%E5%90%88%E5%BA%A6#%E7%B5%90%E5%90%88%E5%BA%A6%E3%81%AE%E6%A6%82%E5%BF%B5%E3%83%A2%E3%83%87%E3%83%AB)
- [Best Practices for Designing a Pragmatic RESTful API](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)
- [REST vs. RPC: what problems are you trying to solve with your APIs?](https://cloudblog.withgoogle.com/products/application-development/rest-vs-rpc-what-problems-are-you-trying-to-solve-with-your-apis/amp/)
- [RESTful Web アプリの設計レビューの話](https://www.slideshare.net/t_wada/restful-web-design-review)
