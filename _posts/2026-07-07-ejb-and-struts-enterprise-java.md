---
layout: post
title: "EJBとStrutsを業務システムの文脈で理解する"
date: 2026-07-07
section: tech
kind: Tech
tags: [Java, EJB, Struts, Jakarta EE, 業務システム]
summary: "EJBとStrutsを、単なる古いJava技術ではなく、業務システムの責務分離・トランザクション・画面遷移の文脈から整理する。"
---

## はじめに

Javaの業務システムについて調べていると、**Struts** や **EJB** という名前が出てくる。

最初に見た印象だけでいうと、どちらも「古いJavaの技術」に見えやすい。実際、現在の新規開発では Spring Boot などを使う現場も多いと思う。ただ、金融系や大企業の基幹系システムでは、過去に作られたStruts/EJBベースのシステムを保守・改修する機会がまだ残っている。

この記事では、StrutsとEJBを単体の知識としてではなく、**業務システム全体の中で何を担当していたのか**という観点で整理する。

## まず役割を分ける

ざっくり言うと、StrutsとEJBは担当レイヤーが違う。

| 技術 | 主な担当 | 今の感覚で近いもの |
| --- | --- | --- |
| Struts | Web MVC、リクエスト処理、画面遷移 | Spring MVC / Controller |
| EJB | 業務ロジック、トランザクション、セキュリティ、非同期処理 | Service層 + トランザクション管理 + 一部DI/コンテナ機能 |

Strutsは、ブラウザから来たHTTPリクエストを受けて、どのActionを実行し、どのJSPや画面に遷移するかを管理する。

EJBは、業務ロジックをアプリケーションサーバー上のコンポーネントとして動かし、トランザクションやセキュリティなどの横断的な関心事をコンテナに任せるための仕組みだった。

つまり、かなり単純化するとこうなる。

```text
ブラウザ
  ↓ HTTP request
Struts Action
  ↓ 業務処理の呼び出し
EJB / Service
  ↓ DBアクセス
DAO / Repository
  ↓
DB
```

Strutsは「Webから入ってきた処理をさばく入口」、EJBは「業務処理を安全に実行する場所」と考えると理解しやすい。

## Strutsは何をしているのか

Struts、特にStruts 2では、リクエスト処理の中心に以下の3つがある。

- Interceptor
- Action
- Result

公式のCore Developers Guideでも、Struts 2はリクエストを `interceptors`、`actions`、`results` の3種類を使って処理すると説明されている。

### Action

Actionは、リクエストに対応して実行される処理単位。

たとえばログイン画面なら、`LoginAction` のようなクラスがあり、フォームから送られたIDやパスワードを受け取り、ログイン処理を呼び出す。

イメージとしては以下のような形になる。

```java
public class LoginAction {
    private String userId;
    private String password;

    public String execute() {
        boolean ok = loginService.login(userId, password);

        if (ok) {
            return "success";
        }
        return "input";
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

Strutsでは、Actionの戻り値である `"success"` や `"input"` をもとに、どの画面へ進むかを設定する。

```xml
<action name="login" class="example.LoginAction">
    <result name="success">/WEB-INF/jsp/home.jsp</result>
    <result name="input">/WEB-INF/jsp/login.jsp</result>
</action>
```

この構造は、今のSpring MVCでいうと、ControllerメソッドがView名を返す感覚に近い。

### Interceptor

Interceptorは、Actionの前後に挟まる共通処理。

たとえば以下のような処理が該当する。

- パラメータの受け取り
- バリデーション
- 型変換
- ログ出力
- 認証・認可のチェック
- 例外処理

業務システムでは、個別の画面処理よりも「全画面で共通して必要な処理」が多い。Interceptorは、その共通処理をActionから切り離すための仕組みとして重要になる。

### Result

Resultは、Action実行後にどこへ遷移するかを表す。

JSPを表示する場合もあれば、別のActionへリダイレクトする場合もある。

```xml
<result name="success">/WEB-INF/jsp/list.jsp</result>
<result name="error">/WEB-INF/jsp/error.jsp</result>
```

Strutsを読むときは、まずAction単体を見るよりも、`struts.xml` のAction定義とResult定義を合わせて見る方が流れを追いやすい。

## EJBは何をしているのか

EJBは、もともとは **Enterprise JavaBeans** の略で、現在はJakarta EEの文脈では **Jakarta Enterprise Beans** と呼ばれる。

EJBの目的は、業務ロジックを単なるJavaクラスとして置くのではなく、アプリケーションサーバーのコンテナ上で管理し、以下のような機能を利用できるようにすることだった。

- トランザクション管理
- セキュリティ
- 並行実行制御
- 非同期処理
- スケジューリング
- メッセージ駆動処理
- リモート呼び出し

業務システムでは、単にメソッドを呼べればよいわけではない。

たとえば銀行振込のような処理では、口座Aから引き落とし、口座Bへ入金し、履歴を登録する必要がある。このとき途中で失敗したら、全体をロールバックしなければならない。

```text
口座Aから引き落とし
  ↓
口座Bへ入金
  ↓
取引履歴を登録
```

この一連の処理を「全部成功するか、全部失敗するか」にするのがトランザクション管理であり、EJBはそのような業務処理を扱うために使われてきた。

## EJBの主な種類

EJBにはいくつか種類がある。

### Stateless Session Bean

状態を持たない業務処理を表す。

```java
@Stateless
public class TransferService {
    public void transfer(String from, String to, int amount) {
        // 引き落とし、入金、履歴登録など
    }
}
```

ログイン中のユーザーごとの状態をBean自体に持たないため、コンテナ側でインスタンスを再利用しやすい。業務サービスとしては一番イメージしやすい。

### Stateful Session Bean

クライアントごとの状態を保持するBean。

たとえば、複数ステップの入力画面や、セッションごとの作業状態を保持したい場合に使われる。ただし、状態を持つ分だけ管理が複雑になりやすい。

### Singleton Session Bean

アプリケーション内で1つだけ存在するBean。

アプリケーション全体で共有する設定やキャッシュのような用途が考えられる。ただし、共有状態を扱うため、並行アクセスには注意が必要になる。

### Message-Driven Bean

メッセージを受け取って非同期に処理するBean。

たとえば、キューに積まれた処理をバックグラウンドで順番に処理するようなケースで使われる。

```text
注文受付
  ↓
キューにメッセージを送信
  ↓
Message-Driven Bean が非同期処理
```

画面からのリクエストをその場で全部処理すると遅くなる場合や、後続処理を非同期化したい場合に出てくる考え方だと思う。

## StrutsとEJBはどうつながるのか

StrutsとEJBを組み合わせる場合、基本的にはStruts Actionから業務サービスを呼び出し、その業務サービスがEJBとして実装されている、という構造になる。

```text
LoginAction
  ↓
UserService
  ↓
UserServiceBean（EJB）
  ↓
UserDao
```

大事なのは、**Struts Actionに業務ロジックを書きすぎないこと**。

ActionはHTTPリクエスト、画面入力、画面遷移に近い場所なので、そこに振込処理や在庫引当処理のような本質的な業務ロジックを書き込むと、テストしづらく、再利用しづらく、変更にも弱くなる。

理想的には、Actionは薄く保つ。

```java
public class TransferAction {
    private TransferService transferService;
    private String fromAccount;
    private String toAccount;
    private int amount;

    public String execute() {
        transferService.transfer(fromAccount, toAccount, amount);
        return "success";
    }
}
```

Actionは入力を受け取り、業務サービスを呼び、結果に応じて画面遷移を返す。

一方、トランザクション境界や本当の業務ルールはService/EJB側に置く。

```java
@Stateless
public class TransferServiceBean implements TransferService {
    public void transfer(String fromAccount, String toAccount, int amount) {
        // ここで業務ルールとトランザクションを扱う
    }
}
```

この分離を意識すると、古い構成でも「どこに何を書くべきか」がかなり見えやすくなる。

## EJB 2系とEJB 3系では印象がかなり違う

EJBで混乱しやすいのは、バージョンによって書き方の印象が大きく違うこと。

古いEJB 2.x系では、Homeインターフェース、Remoteインターフェース、Deployment Descriptorなどが多く、かなり重厚な書き方になりやすかった。

一方、EJB 3以降ではアノテーションベースになり、見た目はかなり軽くなった。

```java
@Stateless
public class OrderService {
    public void register(Order order) {
        // 注文登録
    }
}
```

このため、現場で「EJB」と聞いたときは、どの世代のEJBなのかを確認した方がよい。

- EJB 2.x以前の重い構成なのか
- EJB 3以降のアノテーション中心なのか
- Entity Beanを使っているのか
- 永続化はJPAに寄っているのか

ここを確認しないと、「EJBは全部つらい」と雑に判断してしまう危険がある。

## Entity BeanとJPAの関係

EJBを調べると、Entity Beanという言葉も出てくる。

Entity Beanは、DBの永続データをEJBとして表現する仕組みだった。ただ、現在のJava/Jakarta EEの感覚では、永続化はJPAで扱う方が一般的だと思う。

そのため、古いシステムを読むときは、Entity Beanが残っているのか、JPAに置き換わっているのかを見ると、そのシステムの世代感が分かる。

```text
古い構成の例:
Entity Bean がDBレコードを表す

比較的新しい構成の例:
JPA Entity + Repository/DAO + Service/EJB
```

業務で読むなら、Entity Beanを深追いするより先に、今そのプロジェクトで実際に使われている永続化の方式を確認した方が実用的だと思う。

## Strutsで特に注意したいセキュリティ観点

Strutsは過去に脆弱性で話題になったこともあるため、保守するならセキュリティ面の確認は重要になる。

特に見たいポイントは以下。

- 使用しているStrutsのバージョン
- 既知脆弱性の対象になっていないか
- `devMode` が本番で有効になっていないか
- JSPを直接公開していないか
- Actionのsetterが不用意に公開されていないか
- OGNL式に外部入力を混ぜていないか
- Dynamic Method InvocationやStrict Method Invocationの設定

Struts公式のSecurity Guideでも、Struts 2自体は純粋なWebフレームワークであり、セキュリティ機構そのものを提供するわけではないと説明されている。

つまり、Strutsを使っているから安全なのではなく、アプリケーション側で適切に設定し、危険な使い方を避ける必要がある。

## 現代のSpring Bootと比べるとどう見えるか

今の感覚で置き換えると、だいたい次のように考えると分かりやすい。

| 旧来の構成 | 現代の構成で近いもの |
| --- | --- |
| Struts Action | Spring MVC Controller |
| Interceptor | HandlerInterceptor / Filter / AOP |
| Result | View名 / redirect / response |
| EJB Session Bean | Service + `@Transactional` |
| Message-Driven Bean | メッセージキューのConsumer |
| JNDI Lookup | DIコンテナによる注入 |

もちろん完全に同じではない。

ただ、責務の対応関係で見ると、Struts/EJBのコードも少し読みやすくなる。

## 現場で読むときの順番

Struts/EJBのシステムを読むなら、いきなり個別のJavaクラスを読むより、次の順番で追うのがよさそう。

1. `web.xml` を見る
2. StrutsのFilter/Dispatcher設定を見る
3. `struts.xml` を見る
4. URLとActionの対応を見る
5. Actionの `execute()` や指定methodを見る
6. Actionから呼ばれるService/EJBを見る
7. トランザクション境界を見る
8. DAO/JPA/SQLを見る
9. 画面JSPの入力項目とActionのプロパティ対応を見る

特に業務システムでは、画面、Action、Service、DBが密接につながっている。

```text
JSPの入力項目 name
  ↓
Actionのsetter / DTO
  ↓
Service/EJBの引数
  ↓
DAO / SQL
  ↓
DBカラム
```

この流れを一本の線として追えると、保守改修でかなり強くなると思う。

## まとめ

StrutsとEJBは、単に「昔のJava技術」として切り捨てるより、業務システムの責務分離を学ぶ題材として見ると価値がある。

StrutsはWeb層を担当し、リクエスト、Action、画面遷移を整理する。

EJBは業務ロジック層を担当し、トランザクション、セキュリティ、非同期処理などをコンテナに任せる。

この2つを合わせて見ると、古い業務システムがなぜ重厚な構造になっているのかが少し分かる。

現代のSpring Bootやクラウドネイティブな構成とは違うが、根本にある課題は今も同じだと思う。

- 画面入力をどう受け取るか
- 業務ロジックをどこに置くか
- トランザクションをどこで切るか
- 共通処理をどう分離するか
- セキュリティ上危険な入口をどう塞ぐか

古い技術を読むときは、名前や設定ファイルの多さに圧倒されやすい。

ただ、レイヤーごとの責務に分解すれば、今の技術にもつながる設計の考え方として読み替えられる。

## 参考

- [Jakarta Enterprise Beans](https://jakarta.ee/specifications/enterprise-beans/)
- [Jakarta Enterprise Beans 4.0](https://jakarta.ee/specifications/enterprise-beans/4.0/)
- [Apache Struts Core Developers Guide](https://struts.apache.org/core-developers/)
- [Apache Struts Action Configuration](https://struts.apache.org/core-developers/action-configuration)
- [Apache Struts Security Guide](https://struts.apache.org/security/)
