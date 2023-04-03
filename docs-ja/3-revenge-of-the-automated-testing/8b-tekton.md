## イメージ署名によるTektonパイプラインの拡張

1. ビルドしたイメージに署名するタスクをコードベースに追加します。

    ```bash
    cd /projects/tech-exercise
    cat <<'EOF' > tekton/templates/tasks/image-signing.yaml
    apiVersion: tekton.dev/v1beta1
    kind: Task
    metadata:
      name: image-signing
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
        - name: VERSION
          description: Version of the application
          type: string
        - name: COSIGN_VERSION
          type: string
          description: Version of cosign CLI
          default: 1.7.2
        - name: WORK_DIRECTORY
          description: Directory to start build in (handle multiple branches)
          type: string
      steps:
        - name: image-signing
          image: quay.io/openshift/origin-cli:4.9
          workingDir: $(workspaces.output.path)/$(params.WORK_DIRECTORY)
          script: |
            #!/usr/bin/env bash
            curl -skL -o /tmp/cosign https://github.com/sigstore/cosign/releases/download/v$(params.COSIGN_VERSION)/cosign-linux-amd64
            chmod -R 775 /tmp/cosign

            oc registry login
            /tmp/cosign sign -key k8s://$(params.TEAM_NAME)-ci-cd/$(params.TEAM_NAME)-cosign `oc registry info`/$(params.TEAM_NAME)-test/$(params.APPLICATION_NAME):$(params.VERSION) --allow-insecure-registry
    EOF
    ```

2. このタスクをパイプラインに追加しましょう。 `tekton/templates/pipelines/maven-pipeline.yaml`を編集し、プレースホルダーがある yaml の下にコピーします。

    ```yaml
        # Cosign Image Sign
        - name: image-signing
          runAfter:
          - verify-deployment
          taskRef:
            name: image-signing
          workspaces:
            - name: output
              workspace: shared-workspace
          params:
            - name: APPLICATION_NAME
              value: "$(params.APPLICATION_NAME)"
            - name: TEAM_NAME
              value: "$(params.TEAM_NAME)"
            - name: VERSION
              value: "$(tasks.maven.results.VERSION)"
            - name: WORK_DIRECTORY
              value: "$(params.APPLICATION_NAME)/$(params.GIT_BRANCH)"
    ```

3. git にないと本物じゃないですよね？

    ```bash
    # git add, commit, push your changes..
    cd /projects/tech-exercise
    git add .
    git commit -m  "👨‍🎤 ADD - image-signing-task 👨‍🎤"
    git push
    ```

4. イメージを確認したい人のために、公開鍵を`pet-battle-api`リポジトリに保存します。このプッシュは、パイプラインもトリガーします。

    ```bash
    cp /tmp/cosign.pub /projects/pet-battle-api/
    cd /projects/pet-battle-api
    git add cosign.pub
    git commit -m  "🪑 ADD - cosign public key for image verification 🪑"
    git push
    ```

    🪄 **pet-battle-api**パイプラインで**image-sign**タスクが実行されている様子の観察します。

    タスクが正常に終了したら、OpenShift UI &gt; Builds &gt; ImageStreams に移動し、 `pet-battle-api`を選択します。これがイメージ署名されていることを示す`.sig`で終わるタグが表示されます。

    ![cosign-image-signing](images/cosign-image-signing.png)

5. 署名されたイメージを公開鍵で検証しましょう。画像に適切な`APP VERSION`を使用していることを確認してください。 (この場合は`1.3.1` )

    ```bash
    cd /projects/pet-battle-api
    oc registry login $(oc registry info) --insecure=true
    cosign verify --key k8s://<TEAM_NAME>-ci-cd/<TEAM_NAME>-cosign default-route-openshift-image-registry.<CLUSTER_DOMAIN>/<TEAM_NAME>-test/pet-battle-api:1.3.1 --allow-insecure-registry
    ```

    出力は次のようになります。

     <div class="highlight" style="background: #f7f7f7">
     <pre><code class="language-bash">
        Verification for default-route-openshift-image-registry.&lt;CLUSTER_DOMAIN&gt;/&lt;TEAM_NAME&gt;-test/pet-battle-api:1.3.1 --
        The following checks were performed on each of these signatures:
          - The cosign claims were validated
          - The signatures were verified against the specified public key
          - Any certificates were verified against the Fulcio roots.
        {"critical":{"identity":{"docker-reference":"default-route-openshift-image-registry.&lt;CLUSTER_DOMAIN&gt;/&lt;TEAM_NAME&gt;-test/pet-battle-api"},"image":{"docker-manifest-digest":"sha256:1545e1d2cf0afe5df99fe5f1d39eef8429a2018c3734dd3bdfcac5a068189e39"},"type":"cosign container image signature"},"optional":null}
        </code></pre>
    </div>
    
