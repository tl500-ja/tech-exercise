## Blue/Greenデプロイメント

> Blue/Green デプロイメントでは、アプリケーションの 2 つのバージョンを同時に実行し、トラフィックを古いバージョンから新しいバージョンに移動します。 Blue/Green デプロイメントにより、2 つの異なるバージョン間の切り替えが非常に簡単になります。

<span style="color:blue;"><a href="https://docs.openshift.com/container-platform/4.9/applications/deployments/route-based-deployment-strategies.html#deployments-blue-green_route-based-deployment-strategies">OpenShift Docs は、</a></span>手動で Blue/Green デプロイを行う方法の例を示すのに非常に優れています。しかし、現実の世界では、何らかのテストやその他のメトリックに基づいて、このアクティブなルートの切り替えを自動化する必要があります。さらに、これは GITOPS です。では、このすべての自動化と新しい技術を使用してBlue/Greenを行うにはどうすればよいでしょうか。Pet Battle UI を見てみましょう!

![blue-green-diagram](images/blue-green-diagram.jpg)

1. ペットバトルのフロントエンド用に、ArgoCD リポジトリに 2 つの新しいデプロイメントを作成しましょう。 1 つをBlue、もう 1 つをGreenと呼びます。 `tech-exercise/pet-battle/test/values.yaml`に 2 つの新しいアプリケーションを追加します。作成したものと一致するように、 `source_ref` Helm チャートのバージョンと`image_version`を調整します。

    ```bash
    cat << EOF >> /projects/tech-exercise/pet-battle/test/values.yaml
      # Pet Battle UI Blue
      blue-pet-battle:
        name: blue-pet-battle
        enabled: true
        source: http://nexus:8081/repository/helm-charts
        chart_name: pet-battle
        source_ref: 1.0.6 # helm chart version - may need adjusting!
        values:
          image_version: latest # container image version - may need adjusting!
          fullnameOverride: blue-pet-battle
          blue_green: active
          # we controll the prod route via the "blue" chart for simplicity
          prod_route: true
          prod_route_svc_name: blue-pet-battle
          config_map: '{
            "catsUrl": "https://pet-battle-api-<TEAM_NAME>-test.<CLUSTER_DOMAIN>",
            "tournamentsUrl": "https://pet-battle-tournament-<TEAM_NAME>-test.<CLUSTER_DOMAIN>",
            "matomoUrl": "https://matomo-<TEAM_NAME>-ci-cd.<CLUSTER_DOMAIN>/",
            "keycloak": {
              "url": "https://keycloak-<TEAM_NAME>-test.<CLUSTER_DOMAIN>/auth/",
              "realm": "pbrealm",
              "clientId": "pbclient",
              "redirectUri": "http://localhost:4200/tournament",
              "enableLogging": true
            }
          }'

      # Pet Battle UI Green
      green-pet-battle:
        name: green-pet-battle
        enabled: true
        source: http://nexus:8081/repository/helm-charts
        chart_name: pet-battle
        source_ref: 1.0.6 # helm chart version - may need adjusting!
        values:
          image_version: latest # container image version - may need adjusting!
          fullnameOverride: green-pet-battle
          blue_green: inactive
          config_map: '{
            "catsUrl": "https://pet-battle-api-<TEAM_NAME>-test.<CLUSTER_DOMAIN>",
            "tournamentsUrl": "https://pet-battle-tournament-<TEAM_NAME>-test.<CLUSTER_DOMAIN>",
            "matomoUrl": "https://matomo-<TEAM_NAME>-ci-cd.<CLUSTER_DOMAIN>/",
             "keycloak": {
              "url": "https://keycloak-<TEAM_NAME>-test.<CLUSTER_DOMAIN>/auth/",
              "realm": "pbrealm",
              "clientId": "pbclient",
              "redirectUri": "http://localhost:4200/tournament",
              "enableLogging": true
            }
          }'
    EOF
    ```

2. Git で変更をコミットすると、OpenShift UI で 2 つの新しいデプロイが有効になっていることがわかります。

    ```bash
    cd /projects/tech-exercise
    git add pet-battle/test/values.yaml
    git commit -m  "🍔 ADD - blue & green environments 🍔"
    git push
    ```

3. 各サービスに正しいラベルが含まれていることを確認します。一方は`active`で、もう一方は`inactive`である必要があります。パイプラインは、ラベルを切り替える前に、新しいデプロイを非アクティブなデプロイにプッシュします。

    ```bash
    oc get svc -l blue_green=inactive --no-headers -n <TEAM_NAME>-test
    oc get svc -l blue_green=active --no-headers -n <TEAM_NAME>-test
    ```

4. 両方がデプロイされたら、 `Jenkinsfile`を更新して`inactive`ファイルのデプロイを行いましょう。 Jenkins は、現在ラベル付けされている`inactive`デプロイメントを上書きします。その後、Jenkins はいくつかのテスト (🪞💨) を実行し、問題なく動作することを確認します。最後に、トラフィックをそこに切り替え、ラベルを交換して、これがアクティブなサービスになるようにします。他のサービスは`inactive`とラベル付けされ、望ましくない結果が発生した場合に備えて、元に戻す準備が整うまで待機します。

    これを行うには、次のステージを適切なプレースホルダーに追加します。

    ```groovy
    // 💥🔨 BLUE / GREEN DEPLOYMENT GOES HERE
    stage("🔷✅ Blue Green Deploy") {
      agent {
        label "jenkins-agent-argocd"
      }
      options {
         skipDefaultCheckout(true)
      }
      steps {
        echo '### set env to test against ###'
        sh '''
          #🌻 1. Get the current active / inactive
          export INACTIVE=$(oc get svc -l blue_green=inactive --no-headers -n ${DESTINATION_NAMESPACE} | cut -d' ' -f 1)
          export ACTIVE=$(oc get svc -l blue_green=active --no-headers -n ${DESTINATION_NAMESPACE} | cut -d' ' -f 1)

          #🌻 2. Deploy the new changes to hte current `inactive`
          printenv
          git clone https://${GIT_CREDS}@${ARGOCD_CONFIG_REPO} config-repo
          cd config-repo
          git checkout ${ARGOCD_CONFIG_REPO_BRANCH} # master or main
          yq eval -i .applications.\\"${INACTIVE}\\".source_ref=\\"${CHART_VERSION}\\" "${ARGOCD_CONFIG_REPO_PATH}"
          yq eval -i .applications.\\"${INACTIVE}\\".values.image_version=\\"${VERSION}\\" "${ARGOCD_CONFIG_REPO_PATH}"
          # Commit the changes :P
          git config --global user.email "jenkins@rht-labs.bot.com"
          git config --global user.name "Jenkins"
          git config --global push.default simple
          git add ${ARGOCD_CONFIG_REPO_PATH}
          git commit -m "🚀 AUTOMATED COMMIT - Deployment of ${APP_NAME} at version ${VERSION} 🚀" || rc1=$?
          git remote set-url origin  https://${GIT_CREDS}@${ARGOCD_CONFIG_REPO}
          git push -u origin ${ARGOCD_CONFIG_REPO_BRANCH}

          #🌻 3. do some kind of verification of the deployment
          sleep 10
          echo "🪞💨 TODO - some kinda test to validate blue or green is working as expected ... 🪞💨"
          curl -k -L -f $(oc get route --no-headers ${INACTIVE//_/-} -n $DESTINATION_NAMESPACE | cut -d' ' -f 4)

          #🌻 4. If "tests" have passed swap inactive to active to and vice versa
          yq eval -i .applications.\\"${INACTIVE}\\".values.blue_green=\\"active\\" "${ARGOCD_CONFIG_REPO_PATH}"
          yq eval -i .applications.\\"${ACTIVE}\\".values.blue_green=\\"inactive\\" "${ARGOCD_CONFIG_REPO_PATH}"

          #🌻 5. update the 'prod' route to point to the new active svc
          export NEW_ACTIVE=${INACTIVE//_/-}
          echo "🐥 - ${NEW_ACTIVE}"
          yq eval -i .applications.blue-pet-battle.values.prod_route_svc_name=\\"${NEW_ACTIVE}\\" "${ARGOCD_CONFIG_REPO_PATH}"
          git add ${ARGOCD_CONFIG_REPO_PATH}
          git commit -m "🚀 AUTOMATED COMMIT - Deployment of ${APP_NAME} at version ${VERSION} 🚀" || rc1=$?
          git remote set-url origin  https://${GIT_CREDS}@${ARGOCD_CONFIG_REPO}
          git push -u origin ${ARGOCD_CONFIG_REPO_BRANCH}
        '''
      }
    }
    ```

5. `Jenkinsfile`への変更をコミットする前に、簡単なアプリケーションの変更を加えて、これをより視覚的にしましょう。フロントエンドでは、アプリの上部にあるバナーを変更します。 IDE で、 `pet-battle/src/app/shell/header/header.component.html`を開きます。 `<!-- PB - Purple -->`コメントの下にある`<nav>`コメントを外し、その上の行を削除して、次のようにします。

    ```html
    <header>
        <!-- PB - Purple -->
        <nav class="navbar  navbar-expand-lg navbar-dark" style="background-color: #563D7C;">
    ```

6. フロントエンドのリポジトリのルートにある`package.json`のバージョンを更新して、アプリケーションの`version`を上げて新しいリリースをトリガーします。

     <div class="highlight" style="background: #f7f7f7">
     <pre><code class="language-yaml">
        "name": "pet-battle",
        "version": "1.6.1",  &lt;- bump this
        "private": true,
        "scripts": ...
        </code></pre>
    </div>
    

7. これらすべての変更をコミットします。

    ```bash
    cd /projects/pet-battle
    git add .
    git commit -m "🔵 ADD - Blue / Green deployment to pipeline 🟩"
    git push
    ```

8. Jenkins が実行されると、進行状況が表示され、Blue/Greenデプロイが自動的に行われます。

    本番環境のバージョンは、最新の変更を加えて公開された新しい`1.6.1`です。本番環境のルート`prod-pet-battle`( `green`サービスにリンクされている)からアクセスできるアプリケーションのナビゲーション バーから確認できます:

    ![prod-pet-battle](images/bg-prod-pet-battle.png)

    現在`blue`として識別されている以前の`1.2.0`バージョンは、すでに blue ルート`blue-pet-battle`から入手できます。

    ![blue-pet-battle](images/bg-blue-pet-battle.png)

    `package.json`ファイルの`version`変数を変更するたびに、BlueとGreenのバージョンが切り替わります。アプリケーションの新しいバージョンを公開してみてください (例: `1.6.2` )。どれが本番環境にありますか？ `blue`はどれ？ `green`はどれ？

    これは、GitOps を使用してBlue/Greenデプロイを自動化する方法を示す簡単な例です。ただし、pet-battleの以前のデプロイメントは削除しませんでした。現実の世界ではこれを行うのです。
