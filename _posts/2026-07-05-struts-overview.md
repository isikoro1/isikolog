---
layout: post
title: "Strutsについて整理する"
date: 2026-07-05 00:00:00 +0900
section: tech
tags:
  - Java
  - Struts
  - Web
  - MVC
summary: "仕事で Struts を使う可能性があるため、Java の Web アプリケーション向け MVC フレームワークとしての Struts の概要を整理する。"
---

## はじめに

仕事で Struts を使う可能性があるため、まずは概要を整理しておく。

Struts は Java の Web アプリケーション開発で使われてきた MVC フレームワークの一つで、特に古めの業務システムでは今でも使われていることがある。

最近の Java Web 開発では Spring Framework や Spring Boot を使うことが多いが、既存システムの保守・改修では Struts を読む機会もある。

この記事では、まず Struts がどういうものなのか、どのような部品で構成されているのかを整理する。

## Strutsとは

Struts は、Java の Web アプリケーション向けのフレームワークである。

Apache Struts の公式サイトでは、Struts は Java Web アプリケーションを作成するための無料・オープンソースの MVC フレームワークとして説明されている。

主な役割は、ブラウザから送られてきたリクエストを受け取り、対応する処理を実行し、その結果に応じて JSP などの画面へ遷移させること。

ざっくり言うと、次のような流れを担当する。

```text
ブラウザからリクエスト
  ↓
Struts がリクエストを受け取る
  ↓
対応する Action クラスを実行する
  ↓
処理結果に応じて JSP などの画面へ遷移する
```

Web アプリケーションでは、ユーザーが画面からボタンを押したり、フォームを送信したりする。

そのリクエストに対して、どの処理を実行するのか、処理後にどの画面を表示するのかを管理するのが Struts の大きな役割である。

## MVCフレームワークとしてのStruts

Struts は MVC フレームワークとして説明されることが多い。

MVC は、アプリケーションを次の3つに分けて考える設計の考え方である。

```text
Model
  データや業務処理を担当する

View
  画面表示を担当する

Controller
  リクエストを受け取り、処理の流れを制御する
```

Struts に当てはめると、おおまかには次のように考えられる。

```text
Model
  Service
  Entity
  DAO / Repository
  DBアクセス処理など

View
  JSP

Controller
  Action クラス
```

Struts の Action クラスは、Spring MVC でいう Controller に近い。

ただし、Struts では画面遷移の定義を Java コードに直接書くのではなく、XML の設定ファイルに書くことが多い。

この点は、最近のアノテーションベースのフレームワークに慣れていると少し分かりにくい部分だと思う。

## Strutsの主な構成要素

Struts を理解する上で、まず押さえておきたい部品は次のあたり。

```text
Action
  リクエストを受け取って処理を呼び出すクラス

JSP
  画面表示を担当するファイル

設定ファイル
  URL、Action、遷移先 JSP などの対応を定義する

Form / ActionForm
  画面から送信された入力値を受け取るためのクラス

Interceptor
  Action の前後に共通処理を挟む仕組み

Service
  業務処理を書く層

DAO / Repository
  DBアクセスを担当する層
```

この中で Struts らしいのは、Action と設定ファイルの関係である。

Action は処理の入口になるクラスで、設定ファイルは「どのリクエストがどの Action に対応するか」「Action の結果によってどの画面に進むか」を定義する。

## Actionとは

Action は、Struts における Controller のような役割を持つクラスである。

ユーザーが画面からリクエストを送ると、Struts は設定ファイルをもとに対応する Action を探して実行する。

例えば、ログイン処理であれば、次のような Action が考えられる。

```java
public class LoginAction {
    public String execute() {
        if (loginOk()) {
            return "success";
        }

        return "input";
    }

    private boolean loginOk() {
        // ログイン判定処理
        return true;
    }
}
```

ここで `execute()` は文字列を返している。

この `"success"` や `"input"` は、直接 JSP のファイル名を表しているわけではない。

Struts では、この戻り値をもとに設定ファイルを参照し、次に表示する画面を決める。

## 設定ファイルで画面遷移を定義する

Struts では、Action の戻り値と遷移先の対応を XML で定義する。

例えば Struts 2 では、`struts.xml` に次のような設定を書く。

```xml
<package name="default" extends="struts-default">
    <action name="login" class="com.example.action.LoginAction">
        <result name="success">/WEB-INF/jsp/home.jsp</result>
        <result name="input">/WEB-INF/jsp/login.jsp</result>
        <result name="error">/WEB-INF/jsp/error.jsp</result>
    </action>
</package>
```

この場合、Action の戻り値と遷移先は次のように対応する。

```text
return "success";
  → /WEB-INF/jsp/home.jsp

return "input";
  → /WEB-INF/jsp/login.jsp

return "error";
  → /WEB-INF/jsp/error.jsp
```

つまり、`execute()` が返す文字列は、設定ファイル上の `<result name="...">` と対応している。

Java コードだけを見ると遷移先が分からないため、Struts のコードを読むときは Action と設定ファイルをセットで確認する必要がある。

この仕組みは、Java コードの中に画面遷移を直接書くのではなく、外部の設定ファイルに対応表を持たせていると考えると分かりやすい。

イメージとしては、次のような switch 文に近い。

```java
String result = action.execute();

switch (result) {
    case "success":
        forward("/WEB-INF/jsp/home.jsp");
        break;

    case "input":
        forward("/WEB-INF/jsp/login.jsp");
        break;

    case "error":
        forward("/WEB-INF/jsp/error.jsp");
        break;
}
```

実際にこの switch 文を書くわけではないが、Struts の内部では Action の戻り値と設定ファイルの定義を照合して、次の画面を決めている。

## Strutsのリクエスト処理の流れ

Struts の基本的な処理の流れは、次のように考えられる。

```text
1. ブラウザからリクエストが送られる
2. Struts がリクエストを受け取る
3. 設定ファイルをもとに実行する Action を決める
4. Action の execute() が実行される
5. execute() が "success" や "input" などの文字列を返す
6. 設定ファイルの result 定義を見て遷移先を決める
7. JSP などの画面を表示する
```

図にすると、次のようなイメージになる。

```text
画面
  ↓
リクエスト
  ↓
Struts
  ↓
Action
  ↓
Service
  ↓
Action の戻り値
  ↓
設定ファイル
  ↓
JSP
```

Action はすべての業務処理を自分で行うというより、リクエストを受け取って Service などの処理を呼び出し、その結果に応じて戻り値を返す役割を持つ。

ただし、古いシステムでは Action に業務処理が多く書かれている場合もありそうなので、実際のコードを読むときは注意が必要である。

## Interceptorとは

Struts には Interceptor という仕組みもある。

Interceptor は、Action の前後に処理を挟むためのもの。

ビジネスロジックそのものというより、複数の Action に共通して必要になる処理をまとめる役割に近い。

例えば、次のような処理が Interceptor に向いている。

```text
ログインチェック
権限チェック
入力値の受け取り
型変換
バリデーション
ログ出力
例外処理
二重送信防止
```

イメージとしては、Action の前後にある関所のようなもの。

```text
リクエスト
  ↓
Interceptor
  ↓
Action
  ↓
Interceptor
  ↓
レスポンス
```

Service が業務処理の本体だとすると、Interceptor は業務処理の前後で必要になる共通処理を担当する。

レイヤードアーキテクチャで考えると、Interceptor は Web 層、または Presentation 層に近い。

Entity や Form のようにデータを持つものではなく、処理を挟む仕組みである。

## Formと入力値

Struts では、画面から送信された入力値を受け取る仕組みも重要になる。

Struts 1 では `ActionForm` というクラスが使われることが多い。

```text
JSP のフォーム
  ↓
ActionForm
  ↓
Action
```

Struts 2 では、Action 自体にプロパティと setter を用意して、そこに入力値を受け取ることがある。

```java
public class LoginAction {
    private String userId;
    private String password;

    public void setUserId(String userId) {
        this.userId = userId;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String execute() {
        // userId と password を使って処理する
        return "success";
    }
}
```

このように、Struts では画面からの入力値が Action や Form に渡され、それをもとに処理が行われる。

入力チェックも Struts では重要な要素だが、形式的なチェックと業務的なチェックは分けて考えた方がよさそう。

```text
形式的なチェック
  必須入力
  文字数
  数値形式
  日付形式

業務的なチェック
  ユーザーが存在するか
  残高が足りるか
  申請期限内か
  権限があるか
```

形式的なチェックは Web 層で行われることが多く、業務的なチェックは Service 側に置く方が自然だと思う。

## Struts 1 と Struts 2

Struts には大きく Struts 1 と Struts 2 がある。

名前は似ているが、仕組みには違いがある。

ざっくりした違いとしては、次のように整理できる。

```text
Struts 1
  struts-config.xml を使うことが多い
  ActionForm がよく出てくる
  古い業務システムで見かける可能性がある

Struts 2
  struts.xml を使うことが多い
  Interceptor の仕組みが重要
  Action にプロパティを持たせて入力値を受け取ることがある
```

現場で Struts を見るときは、まず Struts 1 なのか Struts 2 なのかを確認した方がよい。

確認するファイルとしては、次のあたりが候補になる。

```text
pom.xml / build.gradle
  依存ライブラリのバージョンを確認する

web.xml
  Struts の入口となる Servlet や Filter の設定を確認する

struts.xml / struts-config.xml
  Action と画面遷移の対応を確認する
```

Struts 1 と Struts 2 では設定ファイルやクラス構成が違うため、調べるときも混同しないようにしたい。

## Strutsを読むときに見るファイル

Struts のコードを読むときは、まず次のファイルを確認するとよさそう。

```text
pom.xml / build.gradle
  Struts のバージョンを確認する

web.xml
  Struts がどのようにリクエストを受け取るか確認する

struts.xml / struts-config.xml
  URL、Action、遷移先 JSP の対応を確認する

Action クラス
  execute() の処理と戻り値を確認する

JSP
  画面表示とフォーム送信先を確認する

Service
  実際の業務処理を確認する

DAO / Repository
  DBアクセス処理を確認する
```

特に重要なのは、Action と設定ファイルをセットで読むこと。

Action の `execute()` が `"success"` を返していても、それだけではどの画面に遷移するか分からない。

設定ファイルの `<result>` を確認して、初めて遷移先が分かる。

## 今回理解したこと

今回、Struts について調べて整理した内容は次の通り。

```text
Struts は Java の Web アプリケーション向け MVC フレームワークである
リクエストを Action に振り分け、処理結果に応じて JSP などへ遷移する
Action は Controller に近い役割を持つ
execute() の戻り値は設定ファイルの result name と対応する
画面遷移は struts.xml や struts-config.xml に定義される
Interceptor は Action の前後に共通処理を挟む仕組みである
Struts 1 と Struts 2 では構成や設定ファイルが異なる
```

まずは、Struts では Java コードだけで処理を追うのではなく、設定ファイルも一緒に確認する必要があることを押さえておきたい。

特に、Action の戻り値と設定ファイルの対応関係は重要そう。

```text
Action の execute()
  ↓
return "success";
  ↓
設定ファイルの result name="success"
  ↓
対応する JSP へ遷移
```

この流れを理解しておくと、Struts のコードを読むときに迷いにくくなると思う。

## まとめ

Struts は、Java の Web アプリケーションで使われる MVC フレームワークである。

Action、JSP、設定ファイル、Form、Interceptor などの部品があり、リクエストを受け取って処理を実行し、結果に応じて画面遷移を行う。

最近のフレームワークと比べると、XML 設定ファイルを見ないと処理の流れが分かりにくい部分がある。

そのため、Struts を読むときは次の流れを意識したい。

```text
JSP
  ↓
設定ファイル
  ↓
Action
  ↓
Service
  ↓
Action の戻り値
  ↓
設定ファイル
  ↓
JSP
```

まずは全体像として、Struts は「リクエスト」「Action」「設定ファイル」「JSP」をつなぐフレームワークだと理解しておく。

次は、Action の前後に処理を挟む Interceptor について、もう少し詳しく整理したい。

## 参考文献

この記事を書くにあたり、主に Apache Struts の公式ドキュメントを参考にした。

* [Apache Struts 公式サイト](https://struts.apache.org/)
  Struts の概要、公式ドキュメントへの入口として参照した。

* [Apache Struts - Core Developers Guide](https://struts.apache.org/core-developers/)
  Struts 2 の基本構成である Action、Result、Interceptor について確認した。

* [Apache Struts - Action Configuration](https://struts.apache.org/core-developers/action-configuration)
  Action mapping、`execute()` メソッド、Action の戻り値と result の対応について確認した。

* [Apache Struts - Interceptors](https://struts.apache.org/core-developers/interceptors)
  Interceptor が Action の前後に処理を挟む仕組みであり、バリデーション、型変換、ファイルアップロード、二重送信防止などに使われることを確認した。

* [Apache Struts - OGNL](https://struts.apache.org/tag-developers/ognl)
  Struts で使われる OGNL と ValueStack の概要を確認した。

* [Apache Struts - Security Guide](https://struts.apache.org/security/)
  Struts を扱う際のセキュリティ上の注意点を確認した。

* [Apache Struts - Releases](https://struts.apache.org/releases.html)
  Struts のリリース情報を確認した。
