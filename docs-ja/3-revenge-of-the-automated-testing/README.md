# 演習 3 - 自動テストの復讐

> 継続的なテスト - エンド ツー エンドのテストは良さそうに見えますが、すべてのバグをキャッチすることはできないため、常に悪いものです。私たちが本当に必要としているのは、継続的なテストです。

継続的デリバリーには、迅速で信頼できるフィードバックが必要です。継続的なテストへの投資は価値のある活動です。

## 👨‍🍳演習の説明

**💥 あなた自身の冒険を選んでください 💥**

`Quality Gates`の見出しの下でできることはたくさんあるので、自分で何をしたいかを決めてください。テーブル グループで、各演習のタイトルでかんばんを作成します。モブ/ペアとして、リストからタスクを取得して実行します。各セクションの終わりに、自分が達成したことを他のグループに再生します。次に、リストの次の優先項目を取得します。

![team-kanban](images/team-kanban.jpg)

## ビッグピクチャー

![big-picture-pipeline-complete](images/big-picture-pipeline-complete.jpg)

## 🔮 学習成果

- [ ] パイプラインにセキュリティ ゲートを追加できます
- [ ] パイプラインにテスト ゲートを追加できます
- [ ] 静的コード分析ゲートをパイプラインに追加できます
- [ ] イメージ署名をパイプラインに追加できます
- [ ] パイプラインに負荷テストを追加できます

## 🔨 この演習で使用されるツール

- <span style="color:blue;"><a href="https://www.sonarqube.org/">Sonar</a></span> - 静的コード分析をパイプラインに追加
- テストツール -
    <span style="color:blue;"><a href="https://jestjs.io/">Jest</a></span> 、
    <span style="color:blue;"><a href="https://github.com/allure-framework/allure2">Allure</a></span> 、
    <span style="color:blue;"><a href="https://quarkus.io/guides/getting-started-testing">RESTassured</a></span> - API とフロントエンドのテストを追加
- コードリンティング -
    <span style="color:blue;"><a href="https://www.npmjs.com/package/lint">npm lint</a></span> 、
    <span style="color:blue;"><a href="https://checkstyle.sourceforge.io">checkstyle</a></span> - テストの静的コード リンターとカバレッジ レポート
- Kubeリンティング -
    <span style="color:blue;"><a href="https://github.com/stackrox/kube-linter">kubeinter</a></span> - ベスト プラクティスに照らして K8S yaml を検証
- <span style="color:blue;"><a href="https://owasp.org/www-project-zap">ZAP - OWASP</a></span> 一般的な攻撃パターンをチェックするための アプリケーション スキャン
- イメージセキュリティ -
    <span style="color:blue;"><a href="https://www.stackrox.com">StackRox</a></span> - StackRox を使用してイメージとホスト内の脆弱性を発見
- イメージ署名 -
    <span style="color:blue;"><a href="https://www.sigstore.dev">sigstore</a></span> - cosign でイメージに署名
- 負荷テスト -
    <span style="color:blue;"><a href="https://docs.locust.io/en/stable/index.html">イナゴ</a></span>- パイプラインでの自動負荷テスト
- システム テスト - 次の段階に進む前にシステムをテストします。
