---
layout: default
title: 1.各種インストール & ビルド
---
### 1.各種インストール & ビルド

#### 1.Caravelのインストール
基本的に、forkしたCaravel_user_projectのリポジトリ[^11]上で全てを進めていく
```shell
git clone git@github.com:efabless/caravel_user_project.git
cd caravel_user_project
make install
```

#### 2.PDKのビルド
5分くらい掛かる、PDK_ROOTはお好みで設定してよい。
```shell
mkdir ~/pdks
export PDK_ROOT=~/pdks
cd caravel_user_project
make pdk
```

#### 3.OpenLANEのインストール
OpenLNAEのDockerイメージが存在しているのでインストールする、Dockerは事前にインストールしてデーモンを起動しておく。OPENLANE_ROOTはお好みで設定してよい。

```shell
mkdir ~/openlane
export OPENLANE_ROOT=~/openlane
cd caravel_user_project
make openlane
```

以上で必要なツールは揃いました。
