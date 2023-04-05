# KubeリンティングによるJenkinsパイプラインの拡張

私たちのパイプラインには、 `"🏗️ Deploy - Helm Package"`というステージがあることを思い出してください。このステージでは、 `helm lint`を実行してから、helm チャートをパッケージ化して Nexus に保存します。しかし、 `helm lint` 、意図が間違っているかどうかなどの潜在的な問題についてのみチャートをチェックしますが、 `kube-linter`を使用してこの段階を拡張し、セキュリティの構成ミスと Kubernetes のベスト プラクティスもチェックしたいと考えています。

1. `/projects/pet-battle/`の下の Jenkinsfile のプレースホルダーに次のコード スニペットを追加します。アンダー`stage("🏗️ Deploy - Helm Package")`ステージです。

    ```groovy
    		// Kube-linter step
    		echo '### Kube Lint ###'
    		sh '''
    		  export default_option="do-not-auto-add-defaults"
    		  export includelist="no-extensions-v1beta,no-readiness-probe,no-liveness-probe,dangling-service,mismatching-selector,writable-host-mount"
    		  kube-linter lint chart/  --"${default_option}" --include "${includelist}"
    		'''
    ```

    *使用するチェックのセットは制限されていますが、最初に`kube-linter checks list`コマンドで説明したように、含めることができるチェックは他にもあります。*

2. 変更を git にチェックします。

    ```bash
    cd /projects/pet-battle
    # git add, commit, push your changes..
    git add Jenkinsfile
    git commit -m  "🐠 ADD - kube-linter step 🐠"
    git push
    ```

    このプッシュは、パイプラインもトリガーします。パイプラインを見て、失敗することを確認してください 🤯🤯

    *次のように表示されます。*

    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-yaml">
            chart/pet-battle/templates/deploymentconfig.yaml: (object: &lt;no namespace&gt;/test-release-pet-battle apps.openshift.io/v1,
            Kind=DeploymentConfig) container "pet-battle" does not specify a liveness probe
            (check: no-liveness-probe, remediation: Specify a liveness probe in your container.
            Refer to https://kubernetes.io/docs/tasks/configure-pod-container/
            configure-liveness-readiness-startup-probes/ for details.)

            Error: found 1 lint errors
    </code></pre></div>

    Readiness および Liveness プローブは、アプリケーションの正常性状態を追跡するための基本的なベスト プラクティスです。詳細については、 [こちらを](https://docs.openshift.com/container-platform/4.9/applications/application-health.html)参照してください。

3. それでは直しましょう！ `projects/pet-battle/chart/templates/deploymentconfig.yaml`ファイルを開きます。 46 行目あたりに、 `readinessProbe`定義が表示されます。そのブロックの直後に`livelinessProbe`定義を追加します (52 行目)。 `readinessProbe`に合わせる必要があることに注意してください。

    ```yaml
            livenessProbe:
              httpGet:
                  path: /
                  port: 8080
              timeoutSeconds: 1
              periodSeconds: 10
              successThreshold: 1
              failureThreshold: 3
    ```

    YAML ファイルは次のようになります。

    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-yaml">
    ...
                readinessProbe:
                  httpGet:
                      path: /
                      port: 8080
                  initialDelaySeconds: 10
                  timeoutSeconds: 1
                livenessProbe:
                  httpGet:
                      path: /
                      port: 8080
                  timeoutSeconds: 1
                  periodSeconds: 10
                  successThreshold: 1
                  failureThreshold: 3
            dnsPolicy: ClusterFirst
            restartPolicy: Always
            schedulerName: default-scheduler
    ...
    </code></pre>
    </div>

    チャートに変更を加えたので、 `Chart.yaml`でチャートのバージョンを上げる必要があります。

    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-yaml">
    	apiVersion: v2
    	name: pet-battle
    	description: Pet Battle Frontend
    	type: application
    	version: 1.0.6 &lt;- bump this
    	appVersion: 0.0.1
    </code></pre>
    </div>

    変更をプッシュする前に、この変更が役立つかどうかを確認しましょう。

    ```bash
    cd /projects/pet-battle
    kube-linter lint chart --do-not-auto-add-defaults --include no-extensions-v1beta,no-readiness-probe,no-liveness-probe,dangling-service,mismatching-selector,writable-host-mount
    ```

    このような出力が表示されるはずです💪💪

    <div class="highlight" style="background: #f7f7f7">
    <pre><code class="language-yaml">
        KubeLinter 0.2.6
        
        No lint errors found!
    </code></pre>
    </div>

4. 再度、変更をリポジトリにプッシュします。

    ```bash
    cd /projects/pet-battle
    git add .
    git commit -m  "🗻 ADD - Liveliness probe 🗻"
    git push
    ```

    これによりパイプラインが再びトリガーされますが、今回はこのステージの出力が成功するはずです 🔥🔥🔥
