## 負荷テストによるJenkinsパイプラインの拡張

1. 負荷テストには、 <span style="color:blue;"><a href="https://docs.locust.io/en/stable/index.html"><code>locust</code></a></span>という Python ベースのオープン ソース ツールを使用します。 Locust は、シナリオ ベースの負荷テストを記述し、結果が期待と一致しない場合にパイプラインを失敗させるのに役立ちます (つまり、平均応答時間の比率が 200 ミリ秒を超える場合、パイプラインは失敗します)。

    *ニーズに合わせてより複雑なテスト シナリオを作成する方法については<span style="color:blue;"><a href="https://docs.locust.io/en/stable/writing-a-locustfile.html">、Locust のドキュメントを</a></span>参照してください。*

    `locust cli`を使用するには、python3 を含む Jenkins エージェントが必要です。 `tech-exercise/ubiquitous-journey/values-tooling.yaml`を開き、jenkins-agent リストを次のように拡張します。

    ```yaml
            - name: jenkins-agent-python
    ```

    次のようなリストが表示されます。

     <div class="highlight" style="background: #f7f7f7">
     <pre><code class="language-yaml">
                # Jenkins agents for running builds etc
                # default names, versions, repo and paths set on the template
                - name: jenkins-agent-npm
                - name: jenkins-agent-mvn
                - name: jenkins-agent-helm
                - name: jenkins-agent-argocd
                - name: jenkins-agent-python # add this
        </code></pre>
    </div>


    変更を Git リポジトリにコミットします。

    ```bash
    cd /projects/tech-exercise
    git add ubiquitous-journey/values-tooling.yaml
    git commit -m  "🐍 ADD - Python Jenkins Agent 🐍"
    git push
    ```

     <p class="warn">error <b>: failed to push some refs to..</b>のようなエラーが発生した場合は、 <b><i>git pull</i></b>を実行してから、上記のコマンドを実行して変更を再度プッシュしてください。</p>
    

2. テスト シナリオ用に`locustfile.py`を作成し、アプリケーション リポジトリに保存する必要があります。

    以下のシナリオは`/home`エンドポイントを呼び出し、次の場合にテストに失敗します。

    - /cats 呼び出しの 1% が 200 ではない (OK)
    - /cats エンドポイントへの合計平均応答時間は 200 ミリ秒を超えています
    - 90 パーセンタイルの最大応答時間は 800 ミリ秒を超えています

    ```bash
    cat << EOF > /projects/pet-battle/locustfile.py

    import logging
    from locust import HttpUser, task, events

    class getCat(HttpUser):
        @task
        def cat(self):
            self.client.get("/home", verify=False)

    @events.quitting.add_listener
    def _(environment, **kw):
        if environment.stats.total.fail_ratio > 0.01:
            logging.error("Test failed due to failure ratio > 1%")
            environment.process_exit_code = 1
        elif environment.stats.total.avg_response_time > 200:
            logging.error("Test failed due to average response time ratio > 200 ms")
            environment.process_exit_code = 1
        elif environment.stats.total.get_response_time_percentile(0.95) > 800:
            logging.error("Test failed due to 95th percentile response time > 800 ms")
            environment.process_exit_code = 1
        else:
            environment.process_exit_code = 0
    EOF
    ```

3. `jenkins-agent-python`エージェントを使用して負荷テストをトリガーするステージを作成します。以下のコードを`/project/pet-battle/Jenkinsfile`のプレースホルダーにコピーします。

    ```groovy
            // 🏋🏻‍♀️ LOAD TESTING EXAMPLE GOES HERE
            stage("🏋🏻‍♀️ Load Testing") {
                agent { label "jenkins-agent-python" }
                options {
                   skipDefaultCheckout(true)
                }
                steps {
                    sh '''
                    git clone ${GIT_URL} pet-battle && cd pet-battle
                    git checkout ${BRANCH_NAME}
                    '''
                    dir('pet-battle'){
                    script {
                        sh '''
                        pip3 install locust
                        locust --headless --users 10 --spawn-rate 1 -H https://${APP_NAME}-${DESTINATION_NAMESPACE}.<CLUSTER_DOMAIN> --run-time 1m --loglevel INFO --only-summary
                        '''
                       }
                    }
                }
            }
    ```

    上記のコマンドは locust cli をインストールし、1 分間同時に 10 ユーザーのリクエストを開始します。次に、失敗するか、パイプラインを続行します。

    Jenkinsfile を更新したので、パイプラインも開始する変更をプッシュする必要があります。

    ```bash
    cd /projects/pet-battle
    git add Jenkinsfile locustfile.py
    git commit -m  "🌀 ADD - load testing stage and locustfile 🌀"
    git push
    ```

    🪄**負荷テスト**ステージで実行されている**ペット バトル**パイプラインを観察します。

    設定したしきい値が原因でパイプラインが失敗した場合は、 `locustfile.py`をより高い値で更新することにより、いつでも調整できます。

    ```py
        if environment.stats.total.fail_ratio > 0.01:
            logging.error("Test failed due to failure ratio > 1%")
            environment.process_exit_code = 1
        elif environment.stats.total.avg_response_time > 200:
            logging.error("Test failed due to average response time ratio > 200 ms")
            environment.process_exit_code = 1
        elif environment.stats.total.get_response_time_percentile(0.95) > 800:
            logging.error("Test failed due to 95th percentile response time > 800 ms")
            environment.process_exit_code = 1
    ```
