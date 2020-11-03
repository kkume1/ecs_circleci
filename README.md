## CircleCi による、AWS ECSへのビルド/デプロイ実行コンフィグ

#### 概要
このリポジトリがコミットされると、CircleCIのビルド/デプロイを実行できます。

---
構成図
![aws構成図](https://user-images.githubusercontent.com/30540542/98032856-f8cb6780-1e57-11eb-8865-7a1fef3b8038.jpg)
---

## .circleci/config.ymlについて
- ジョブは以下の二つを設定しています。
  - aws-ecr:circleci/aws-ecr@6.10.0
  - aws-ecs:circleci/aws-ecs@1.2.0  
- 「aws-ecr:circleci/aws-ecr@6.10.0」が成功しないと、「aws-ecs:circleci/aws-ecs@1.2.0」は実行されません。  

```
  requires:
    - aws-ecr/build-and-push-image
```

- 「aws-ecs:circleci/aws-ecs@1.2.0」はmasterブランチでのみ実行可能です。

```
  - aws-ecs/deploy-service-update:
    ～～～～～～～～～
      filters:
        branches:
          only: master
```

## circleci側の設定について
- AWS認証情報やECRのリポジトリURLはcircleciにて変数として設定しています。  
  circleci内にても、展開された変数の内容は伏字にて表示されます。

## github側の設定について
github側に以下の設定をしております。
- Require pull request reviews before merging  
  (masterへのマージにはプルリクエストが必要）
- Require status checks to pass before merging  
  （CircleCIのジョブ実行結果チェック）  
 「ci/circleci: build-and-push-image」が成功しないと、masterブランチへのプルリクエストがapproveされない。

## 想定する作業フロー
以下の作業フローを想定しています。

- メンバーがmasterリポジトリをローカルにgit cloneする。
- ローカルにクローンしたリポジトリに作業用ブランチを切る。
- 作業用ブランチでスクリプトを修正し「git push origin <作業用ブランチ>」を実行する。
- 「aws-ecr:circleci/aws-ecr@6.10.0」ジョブがgithub上の作業用ブランチから、circlrciに投げられ実行される。
- 「aws-ecr:circleci/aws-ecr@6.10.0」が成功したら、メンバーはプルリクエストを発行する。  
（「aws-ecr:circleci/aws-ecr@6.10.0」が成功しないと、プルリクエストを発行してもマージが拒否される）
- masterブランチ管理者は、プルリクエストの内容に問題がなければ、masterブランチにマージする。
- masterブランチから、「aws-ecr:circleci/aws-ecr@6.10.0」と「aws-ecs:circleci/aws-ecs@1.2.0」が実行される。
- ビルドされたイメージがBule/greenデプロイメントで、デプロイされる。
- メンバーは、不要となった作業用ブランチを削除する。

