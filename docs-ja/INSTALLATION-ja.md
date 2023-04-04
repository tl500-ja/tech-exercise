# tech-exercise-docs-ja INSTALLATION

## ビルド
podman build . -t tech-exercise-docs-ja

## ローカルでの起動
podman run --name tech-exercise-docs-ja -p 8081:8080 --rm localhost/tech-exercise-docs-ja
