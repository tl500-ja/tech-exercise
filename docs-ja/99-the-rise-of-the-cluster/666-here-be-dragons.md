## ドラゴンが来た！

![oh-look-a-dragon](../images/oh-look-dragons.png)

### あるクラスターから別のクラスターに移動します!

コードと構成はすべて git にあるため、継続的デリバリー スタック全体を別の OpenShift クラスターに簡単に移動できます。これは、この実行のコードを使用して、後の段階ですべての演習を試したい場合に便利です。

前提条件として、前のセクション[ツールのインストール を](99-the-rise-of-the-cluster/1-tooling-installation)使用して TL500 をセットアップする必要があります。クラスターとツールをインストールしたら、コードを実行するための手順を説明します。

コードを`cluster-a`から`cluster-b`に移してみましょう。

#### ガイド付き手順

> これを機能させるための短い一連の手順を次に示します。

1. このコースを受講した後、安全に保管するために`tech-exercise` 、 `pet-battle` 、 `pet-battle-api`リポジトリをラップトップに git clone する必要があります。

2. `vscode` IDE などを使用して、コード内の`apps.cluster-a.com -> apps.cluster-b.com`の出現箇所をすべて置き換えます。

3. `gitlab`にログインして ${TEAM_NAME} を作成します

4. 新しいクラスターでホストされている`gitlab`インスタンスにコードをプッシュしましょう。

    ```bash
    export GIT_SERVER=gitlab-ce.apps.cluster-b.com
    export TEAM_NAME=ateam
    ```

    コードはラップトップのこのフォルダーにローカルにあると想定しているので、各リポジトリについて調整してください:

    `Pet-Battle`

    ```bash
    cd ~/git/tl500/pet-battle
    git remote set-url origin https://${GIT_SERVER}/${TEAM_NAME}/pet-battle.git
    git push -u origin main
    ```

    `Pet-Battle-API`

    ```bash
    cd ~/git/tl500/pet-battle-api
    git remote set-url origin https://${GIT_SERVER}/${TEAM_NAME}/pet-battle-api.git
    git push -u origin main
    ```

    `Tech-Exercise`

    ```bash
    cd ~/git/tl500/tech-exercise
    git remote set-url origin https://${GIT_SERVER}/${TEAM_NAME}/tech-exercise.git
    git push -u origin main
    ```

5. `gitlab`にログインし、新しく作成したプロジェクトが**public**に設定されていることを確認します (デフォルトではprivateになります)。

6. この新しいクラスターの`sealed-secrets`を再生成します。これは、セットアップ時にシークレット マスター キーを新しいクラスターに移行しなかっ*たことを*前提としています (移行した場合は、この手順をスキップしてください!)。

    `git-auth`を設定します

    ```bash
    export GITLAB_USER=<user>
    export GITLAB_PAT=<pat token>

    cat << EOF > /tmp/git-auth.yaml
    kind: Secret
    apiVersion: v1
    data:
      username: "$(echo -n ${GITLAB_USER} | base64 -w0)"
      password: "$(echo -n ${GITLAB_PAT} | base64 -w0)"
    metadata:
      annotations:
        tekton.dev/git-0: https://${GIT_SERVER}
      labels:
        credential.sync.jenkins.openshift.io: "true"
      name: git-auth
    type: kubernetes.io/basic-auth
    EOF

    kubeseal < /tmp/git-auth.yaml > /tmp/sealed-git-auth.yaml \
        -n ${TEAM_NAME}-ci-cd \
        --controller-namespace tl500-shared \
        --controller-name sealed-secrets \
        -o yaml

    cat /tmp/sealed-git-auth.yaml| grep -E 'username|password'
    ```

    これをテンポラリに適用する必要があります

    ```bash
    oc apply -n ${TEAM_NAME} -f /tmp/git-auth.yaml
    ```

    `sonarqube-auth`設定します

    ```bash
    cat << EOF > /tmp/sonarqube-auth.yaml
    apiVersion: v1
    data:
      username: "$(echo -n admin | base64 -w0)"
      password: "$(echo -n admin123 | base64 -w0)"
      currentAdminPassword: "$(echo -n admin | base64 -w0)"
    kind: Secret
    metadata:
      labels:
        credential.sync.jenkins.openshift.io: "true"
      name: sonarqube-auth
    EOF

    kubeseal < /tmp/sonarqube-auth.yaml > /tmp/sealed-sonarqube-auth.yaml \
        -n ${TEAM_NAME}-ci-cd \
        --controller-namespace tl500-shared \
        --controller-name sealed-secrets \
        -o yaml

    cat /tmp/sealed-sonarqube-auth.yaml| grep -E 'username|password|currentAdminPassword'
    ```

    `allure-auth`を設定します

    ```bash
    cat << EOF > /tmp/allure-auth.yaml
    apiVersion: v1
    data:
      password: "$(echo -n password | base64 -w0)"
      username: "$(echo -n admin | base64 -w0)"
    kind: Secret
    metadata:
      name: allure-auth
    EOF

    kubeseal < /tmp/allure-auth.yaml > /tmp/sealed-allure-auth.yaml \
        -n ${TEAM_NAME}-ci-cd \
        --controller-namespace tl500-shared \
        --controller-name sealed-secrets \
        -o yaml

    cat /tmp/sealed-allure-auth.yaml| grep -E 'username|password'
    ```

    `rox-auth`を設定します

    ```bash
    export ROX_API_TOKEN=$(oc -n stackrox get secret rox-api-token-tl500 -o go-template='{{index .data "token" | base64decode}}')
    export ROX_ENDPOINT=central-stackrox.apps.cluster-b.com

    cat << EOF > /tmp/rox-auth.yaml
    apiVersion: v1
    data:
      password: "$(echo -n ${ROX_API_TOKEN} | base64 -w0)"
      username: "$(echo -n ${ROX_ENDPOINT} | base64 -w0)"
    kind: Secret
    metadata:
      labels:
        credential.sync.jenkins.openshift.io: "true"
      name: rox-auth
    EOF

    kubeseal < /tmp/rox-auth.yaml > /tmp/sealed-rox-auth.yaml \
        -n ${TEAM_NAME}-ci-cd \
        --controller-namespace tl500-shared \
        --controller-name sealed-secrets \
        -o yaml

    cat /tmp/sealed-rox-auth.yaml | grep -E 'username|password'
    ```

7. 基本を実行します

    ```bash
    export TEAM_NAME="ateam"
    export CLUSTER_DOMAIN="apps.cluster-b.com"
    export GIT_SERVER="gitlab-ce.apps.cluster-b.com"

    oc login --server=https://api.${CLUSTER_DOMAIN##apps.}:6443 -u <TEAM_NAME> -p <PASSWORD>
    ```

8. ArgoCDをインストール

    namespaceをオペレーターのenv.var に追加します。

    ```bash
    run()
    {
      NS=$(oc get subscriptions.operators.coreos.com/openshift-gitops-operator -n openshift-operators \
        -o jsonpath='{.spec.config.env[?(@.name=="ARGOCD_CLUSTER_CONFIG_NAMESPACES")].value}')
      if [ -z $NS ]; then
        NS="${TEAM_NAME}-ci-cd"
      elif [[ "$NS" =~ .*"${TEAM_NAME}-ci-cd".* ]]; then
        echo "${TEAM_NAME}-ci-cd already added."
        return
      else
        NS="${TEAM_NAME}-ci-cd,${NS}"
      fi
      oc -n openshift-operators patch subscriptions.operators.coreos.com/openshift-gitops-operator --type=json \
        -p '[{"op":"replace","path":"/spec/config/env/1","value":{"name": "ARGOCD_CLUSTER_CONFIG_NAMESPACES", "value":"'${NS}'"}}]'
      echo "EnvVar set to: $(oc get subscriptions.operators.coreos.com/openshift-gitops-operator -n openshift-operators \
        -o jsonpath='{.spec.config.env[?(@.name=="ARGOCD_CLUSTER_CONFIG_NAMESPACES")].value}')"
    }
    run
    ```

    Helm チャートをデプロイします

    ```bash
    oc new-project ${TEAM_NAME}-ci-cd
    helm repo add redhat-cop https://redhat-cop.github.io/helm-charts

    helm upgrade --install argocd \
    --namespace ${TEAM_NAME}-ci-cd \
    -f tech-exercise/argocd-values.yaml \
    redhat-cop/gitops-operator
    ```

9. UJをインストールします

    ```bash
    cd tech-exercise
    helm upgrade --install uj --namespace ${TEAM_NAME}-ci-cd .
    ```

10. `tech-exercise` 、 `pet-battle` 、 `pet-battle-api` git リポジトリの統合と Web フックを gitlab に追加します。

11. 新しい cosign 署名キーを作成します。

    ```bash
    cd /tmp
    cosign generate-key-pair k8s://${TEAM_NAME}-ci-cd/${TEAM_NAME}-cosign

    cd /projects/tech-exercise
    git add ubiquitous-journey/values-tooling.yaml
    git commit -m  "🔒 ADD - Cosign Jenkins Agent 🔒"
    git push

    cp /tmp/cosign.pub /projects/pet-battle-api/
    cd /projects/pet-battle-api
    git add cosign.pub
    git commit -m  "🪑 ADD - cosign public key for image verification 🪑"
    git push
    ```

12. ビルドを開始し、それらが機能することを確認し、Helmチャートのバージョンの不一致などを修正します。

    ```bash
    cd /projects/pet-battle-api; git commit -m "test" --allow-empty; git push
    cd /projects/pet-battle; git commit -m "test" --allow-empty; git push
    ```

13. 🎉🎉🎉 新しいクラスターへの移行の成功を祝いましょう 🎉🎉🎉
