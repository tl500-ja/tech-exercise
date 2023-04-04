## 負荷テストによるTektonパイプラインの拡張

1. 負荷テストには、 <span style="color:blue;"><a href="https://docs.locust.io/en/stable/index.html"><code>locust</code></a></span>という Python ベースのオープン ソース ツールを使用します。 Locust は、シナリオ ベースの負荷テストを記述し、結果が期待と一致しない場合にパイプラインを失敗させるのに役立ちます (つまり、平均応答時間の比率が 200 ミリ秒を超える場合、パイプラインは失敗します)。

    テスト シナリオ用に`locustfile.py`を作成し、アプリケーション リポジトリに保存する必要があります。

    *ニーズに合わせてより複雑なテスト シナリオを作成する方法については<span style="color:blue;"><a href="https://docs.locust.io/en/stable/writing-a-locustfile.html">、Locust のドキュメントを</a></span>参照してください。*

    以下のシナリオでは`/cats`エンドポイントを呼び出し、次の場合にテストに失敗します。

    - 呼び出しの 1% は 200 ではない (OK)
    - `/cats`エンドポイントへの合計平均応答時間は 200 ミリ秒を超えています
    - 90 パーセンタイルの最大応答時間は 800 ミリ秒を超えています

    ```bash
    cat << EOF > /projects/pet-battle-api/locustfile.py

    import logging
    from locust import HttpUser, task, events

    class getCat(HttpUser):
        @task
        def cat(self):
            self.client.get("/cats", verify=False)

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

2. 負荷テストを実行するためのタスクを tekton パイプラインに追加します。

    ```bash
    cd /projects/tech-exercise
    cat <<'EOF' > tekton/templates/tasks/load-testing.yaml
    apiVersion: tekton.dev/v1beta1
    kind: Task
    metadata:
      name: load-testing
    spec:
      workspaces:
        - name: output
      params:
        - name: APPLICATION_NAME
          description: Name of the application
          type: string
        - name: TEAM_NAME
          description: Name of the team that doing this exercise :)
          type: string
        - name: WORK_DIRECTORY
          description: Directory to start build in (handle multiple branches)
          type: string
      steps:
        - name: load-testing
          image: quay.io/centos7/python-38-centos7:latest
          workingDir: $(workspaces.output.path)/$(params.WORK_DIRECTORY)
          script: |
            #!/usr/bin/env bash
            pip3 install locust
            locust --headless --users 10 --spawn-rate 1 -H https://$(params.APPLICATION_NAME)-$(params.TEAM_NAME)-test.{{ .Values.cluster_domain }} --run-time 1m --loglevel INFO --only-summary
    EOF
    ```

3. このタスクをパイプラインに追加しましょう。 `tekton/templates/pipelines/maven-pipeline.yaml`を編集し、プレースホルダーがある yaml の下にコピーします。

    ```yaml
        # Load Testing
        - name: load-testing
          runAfter:
            - verify-deployment
          taskRef:
            name: load-testing
          workspaces:
            - name: output
              workspace: shared-workspace
          params:
            - name: APPLICATION_NAME
              value: "$(params.APPLICATION_NAME)"
            - name: TEAM_NAME
              value: "$(params.TEAM_NAME)"
            - name: WORK_DIRECTORY
              value: "$(params.APPLICATION_NAME)/$(params.GIT_BRANCH)"
    ```

4. 覚えておいてください-gitにない場合、それは本物ではありません。

    ```bash
    cd /projects/tech-exercise/tekton
    git add .
    git commit -m  "🌀 ADD - load testing task 🌀"
    git push
    ```

5. 次に、 `locustfile.py`をプッシュして pet-battle-api パイプラインをトリガーし、負荷テスト タスクが期待どおりに機能するかどうかを確認します。

    ```bash
    cd /projects/pet-battle-api
    git add locustfile.py
    git commit -m  "🌀 ADD - locustfile for load testing 🌀"
    git push
    ```

    🪄**負荷テスト**タスクで実行されている**pet-battle-api**パイプラインを観察します。

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
