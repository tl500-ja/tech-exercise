# 演習 3 - 自動テストの復習

> 継続的なテスト - エンド ツー エンドのテストは良さそうに見えますが、すべてのバグをキャッチすることはできないため、常に悪いものです。私たちが本当に必要としているのは、継続的なテストです。

継続的デリバリーには、迅速で信頼できるフィードバックが必要です。継続的なテストへの投資は価値のある活動です。

## 👨‍🍳 Exercise Intro

**💥 Choose your own adventure 💥**

There are lots of things we can do under the heading of `Quality Gates`, so decide for yourselves what you'd like to do. In your table groups, create a Kanban with each of the exercise titles. Discuss among yourselves the order you'd like to do them in and as mobs / pairs, take tasks from the list and implement them. At the end of each section, play back to the other group what you've accomplished. Then grab the next priortized item on your list!

![team-kanban](images/team-kanban.jpg)

## 🖼️ Big Picture

![big-picture-pipeline-complete](images/big-picture-pipeline-complete.jpg)

## 🔮 Learning Outcomes

- [ ] Can add security gates to pipeline
- [ ] Can add testing gates to pipeline
- [ ] Can add static code analysis gates to pipeline
- [ ] Can add image signing to the pipeline
- [ ] Can add load testing to the pipeline

## 🔨 Tools used in this exercise!

- <span style="color:blue;"><a href="https://www.sonarqube.org/">Sonar</a></span> - Add static code analysis to the pipelines
- Testing Tools -
    <span style="color:blue;"><a href="https://jestjs.io/">Jest</a></span>,
    <span style="color:blue;"><a href="https://github.com/allure-framework/allure2">Allure</a></span>,
    <span style="color:blue;"><a href="https://quarkus.io/guides/getting-started-testing">RESTassured</a></span> - Add API and front end tests
- Code Linting -
    <span style="color:blue;"><a href="https://www.npmjs.com/package/lint">npm lint</a></span>,
    <span style="color:blue;"><a href="https://checkstyle.sourceforge.io">checkstyle</a></span> - Static code linter and coverage reports for our tests
- Kube Linting -
    <span style="color:blue;"><a href="https://github.com/stackrox/kube-linter">kubelinter</a></span>- Validate K8S yamls against best practices
- <span style="color:blue;"><a href="https://owasp.org/www-project-zap">ZAP - OWASP</a></span> application scanning to check for common attack patterns
- Image Security -
    <span style="color:blue;"><a href="https://www.stackrox.com">StackRox</a></span> - Finding vulnerabilities inside the images and hosts with StackRox
- Image Signing -
    <span style="color:blue;"><a href="https://www.sigstore.dev">sigstore</a></span> - Sign your images with cosign
- Load Testing -
    <span style="color:blue;"><a href="https://docs.locust.io/en/stable/index.html">locust</a></span> - Automated load tests in your pipeline
- System Test - test the system before promoting to the next stage
