# ドラゴンが来た!

![oh-look-another-dragon](../images/oh-look-dragons.png)

## Tektonプルーニング

Operator *TektonConfig*を設定することで、Tekton リソースのプルーニングをグローバルに設定できます。たとえば、次の構成を使用して、最後の 15 個の*PipelineRun*リソースを保持し、15 分ごとにプルーニングできます。

```yaml
  pruner:
    keep: 15
    resources:
      - pipelinerun
    schedule: '*/15 * * * *'
```

**cluster-admin**パッチとして、これを使用します。

```bash
oc patch tektonconfig config -p '{"spec":{"pruner":{"keep":15,"resources":["pipelinerun"],"schedule":"*/15 * * * *"}}}' --type=merge
```

これにより、次の*targetNamespace*に kubernetes *CronJob*が生成されます。

```bash
oc get cronjob resource-pruner -n openshift-pipelines -o yaml
```

?&gt; **GitOps** これは、クラスターの Tekton をデプロイするために使用されるグローバル チャート/構成に配置する必要があります。
