# tech-exercise-docs-ja INSTALLATION

日本語翻訳版tech-exerciseをローカルで実行する方法。

## クローン
```
git clone -b 1.0.11-ja https://github.com/tl500-ja/tech-exercise.git
```
 
## イメージビルド
```
cd tech-exercise/docs-ja
podman build . -t tech-exercise-docs-ja
```

## ローカルでのコンテナー起動
```
podman run --name tech-exercise-docs-ja -p 8081:8080 --rm localhost/tech-exercise-docs-ja
```

## ブラウザ上で翻訳を開く
localhost:8081 で翻訳ドキュメントが表示されます。
