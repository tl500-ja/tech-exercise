# DevOps Culsture &amp; Practice (TL500)

![jenkins-crio-ocp-star-wars-kubes](./images/jenkins-crio-ocp-star-wars-kubes.png)

## スライドデッキ

技術演習と並行して、スライド デッキが公開されるようになりました。各技術演習の未加工の Markdown ファイルは、学習者とファシリテーターが使用する同じ monorepo にあります。新しいスライド デッキを追加するか、既存のものを更新するには、 `docs/slides/content`に移動して既存のファイルを編集するか、新しい`.md`ファイルを作成します。これにより、公開後にスライド デッキが自動生成されます。 docsify サーバーを実行して、テスト用に を表示または編集できます。詳細については、github リポジトリを参照してください。

👨‍🏫 👉 [The Published Slides Live Here](https://rht-labs.com/tech-exercise/slides/) 👈 🧑‍💻

## 🪄 インストラクションをカスタマイズする

ページ上部のボックスを使用すると、チームが使用する変数が事前に入力されたドキュメントを読み込むことができます。ページ上部のボックスに自分のチーム名とクラスターが使用しているドメインを入力し、 `save`をクリックするだけです。これにより、サイトのローカル ストレージに値が保持されます。間違いを犯した場合は、 `clear`押すと値がリセットされます。

- 私のチームが`biscuits`と呼ばれている場合は、最初のボックスに入れます。この値は、使用する名前空間などの一部にプレフィックスが付けられます。
- クラスター ドメインの場合、 `apps.*` の該当部分はOpenShift ドメインを追加します。たとえば、私のコンソール アドレスが<code class="language-yaml">https://console-openshift-console.apps.hivec.sandbox1243.opentlc.com/</code>にある場合、 `apps.hivec.sandbox1243.opentlc.com`とボックスに記入すれば、演習用の正しいアドレスが生成されます。
- git サーバーの場合、好みのアクセス可能な Git サーバー (GitHub、GitLab など) を使用できます。インストラクターはあなたにそれを提供することができます。たとえば、git サーバーが<code class="language-yaml">https://gitlab-ce.apps.hivec.sandbox1243.opentlc.com/</code>にある場合、ボックスに`gitlab-ce.apps.hivec.sandbox1243.opentlc.com`を入れるだけで正しい演習のアドレスが生成されます。

## 🦆 規則

演習を実行するとき、置換が必要な場所を指摘します。重要なのは、 `<>`内のすべてを置き換える必要があることです。たとえば、あなたのチームが`biscuits`と呼ばれている場合、手順で`\<TEAM_NAME\>`が表示されている場合は、次のように`biscuits`に置き換える必要があります。

<div class="highlight" style="background: #f7f7f7">
<pre><code class="language-bash">
    name: &lt;\TEAM_NAME\&gt;
    # ^ this becomes
    name: biscuits
    </code></pre>
</div>

コピーして貼り付けるコード ブロックが多数あります。コード ブロックにカーソルを移動すると、右側に小さな ✂️ アイコンが表示されます。

```bash
echo "like this one :)"
```

ただし、コピー✂️アイコンのない、コピーして貼り付けてはならないブロックもあります。つまり、指定されたブロックに対して出力の内容または yaml を検証する必要があります。
