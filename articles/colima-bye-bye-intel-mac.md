---
title: "ColimaとDocker Desktopを併用してIntel Macへの未練を断ち切る"
emoji: "🐿"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["lima","colima","docker","m1","m2"]
published: false
---

# M1は速い、M2はもっと速い

AppleがM1チップを世に発表してから、およそ2年が経とうとしています。

ARMアーキテクチャを採用し、Macのために設計されたApple製のシリコンチップは、その「速さ」でMacの体験を大きく変えました。そのパフォーマンスに衝撃を受けて、**Intel Macにはもう戻れない**と思っている人も多いのでは無いでしょうか。

2022年発売のMacBookにはM1の画期的なパフォーマンスと能力をさらに高めたM2チップが搭載されています。M1 ProとM1 Maxはもっと速くて、2022年11月にはM2 ProとM2 Maxの発表も噂されています。

MacでApple製シリコンチップがスタンダードになる流れは止められないでしょう。

# 性能が出ても、M1にできないこともある

しかし、どれだけ速いM1,M2 Macにもできないことがあります。

それは、**x86アーキテクチャに依存しているシステムやアプリケーションを動かすこと**です。

大概のアプリケーションはM1,M2 Macに移行してもRosetta[^1]やDockerを使って動かすことができます。しかしながら、処理がより低いレイヤーな部分に依存していると、そのまま動かせないことがあります。

[^1]: Appleシリコン搭載Macでも、Intelプロセッサ搭載Mac用に開発されたアプリケーションを動かせるようにする互換性維持のための技術です。

その最たる例として、**Oracle Database本体のインストール**が上げられます。Intel Macであれば、Mac上にDockerを起動して、コンテナの中にOracle Databaseをインストールできます。しかし、M1,M2 Macの場合はARMアーキテクチャであるがゆえに、Oracleコンテナをつくれません。

たとえば、レガシーなオンプレミス環境をクラウド環境へ移行する案件に関わっている場合、最新のM1,M2 MacBookを使ってOracleに向き合わなければならないこともあるでしょう。せっかく速いMacを手に入れたのに、このためだけに開発端末を2台持ちしなければならないのは、とても悲しいことです。
<!-- textlint-disable -->
そんなピンチを救ってくれる（かもしれない）のが**Colima**です。
<!-- textlint-enable -->
# Colimaを使えば、Intel Macを部分的に再現できる

M1,M2 Macでx86の環境をつくるためには、Linuxの仮想化技術が必要です。

Mac OS上でLinux仮想マシンを立ち上げる「Lima」を使えば、Intel Macと同じ環境を再現します。

https://github.com/lima-vm/lima

Limaでつくった仮想マシンの上にDockerを入れてホスト側からアクセスすれば、**M1,M2 Mac上からいつも通りDockerを使っているのと同じ体験で、x86アーキテクチャの上にあるコンテナを扱うこと**ができます。

この**Lima + DockerをまとめてやってくれるのがColima**というわけです。

https://github.com/abiosoft/colima

参考までに、Colimaを使ってOracle Databaseをインストールする方法を以下の記事で詳しく書いています。Colimaのインストール手順を確認したい方はご覧ください。`x86_64` のアーキテクチャを指定して起動させる部分がポイントです。

https://qiita.com/waicode/items/d67782c33b7d40052245

# ColimaはDocker Desktopと併用できないという誤解

この記事で伝えたかった本題はここからです。

Colimaを使うなら「Docker Desktop（Docker for Mac）と共存できないからアンインストールせよ」という記事をよく見かけます。

実はこれは間違い（というか古い記述）で、**ColimaとDocker Desktopは共存可能**です。

Colimaを起動させるとDockerの向き先（context）がColima側で強制的に奪われてしまいますが、Docker Desktop側を使うように**コンテキストを切り替えれば、全く問題なく動作**します。

実際、公式のFAQにもv0.3.0から共存できると記載されています。

https://github.com/abiosoft/colima/blob/main/docs/FAQ.md#can-it-run-alongside-docker-for-mac

> Can it run alongside Docker for Mac?（Docker for Macと一緒に動作させることはできますか？）
>
> Yes, from version v0.3.0 Colima leverages Docker contexts and can thereby run alongside Docker for Mac. Colima makes itself the default Docker context on startup and should work straight away.（Colimaはバージョン0.3.0からDocker contextsを利用しており、Docker for Macと一緒に動作させることができます。Colimaは起動時にデフォルトのDockerコンテキストになり、すぐに動作するはずです。）

Dockerのコンテキストを確認すると `colima start` したあとは、たしかにcontextの向き先が `colima *` になっていることが分かります。

```console
# docker context ls
NAME                TYPE                DESCRIPTION                               DOCKER ENDPOINT                                      KUBERNETES ENDPOINT   ORCHESTRATOR
colima *            moby                colima                                    unix:///Users/xxxxx/.colima/default/docker.sock
default             moby                Current DOCKER_HOST based configuration   unix:///var/run/docker.sock                                                swarm
desktop-linux       moby                                                          unix:///Users/xxxxx/.docker/run/docker.sock
```

これを `docker context use` を使って `desktop-linux` または `default` に切り替えればDocker Desktop（Docker for Mac）が使えます。

```console
# docker context ls
NAME                TYPE                DESCRIPTION                               DOCKER ENDPOINT                                      KUBERNETES ENDPOINT   ORCHESTRATOR
colima *            moby                colima                                    unix:///Users/xxxxx/.colima/default/docker.sock
default             moby                Current DOCKER_HOST based configuration   unix:///var/run/docker.sock                                                swarm
desktop-linux       moby

# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# docker context use desktop-linux
desktop-linux

# docker ps
CONTAINER ID   IMAGE                           COMMAND                  CREATED       STATUS       PORTS                    NAMES
2a65602a3db7   zenndocc-zenndocc             "/bin/sh -c 'echo Co…"   13 days ago   Up 2 hours   0.0.0.0:8009->8000/tcp   zenndocc

# docker context ls
NAME                TYPE                DESCRIPTION                               DOCKER ENDPOINT                                      KUBERNETES ENDPOINT   ORCHESTRATOR
colima              moby                colima                                    unix:///Users/xxxxx/.colima/default/docker.sock
default             moby                Current DOCKER_HOST based configuration   unix:///var/run/docker.sock                                                swarm
desktop-linux *     moby
```

MacでDockerを扱うなら、**普段はGUIベースで様々な操作ができるDocker Desktop（Docker for Mac）を使うと最も開発生産性が高い**と（個人的には）感じています。

Oracle DatabaseはDocker Desktopが使えないので、（悪い言い方をすると、仕方なく）Colimaを使います。

# 普段はDocker Desktop、ときどきColima

今回はじめてColimaを使ってみて、その便利さに驚きました。とても実用性が高いのに、一部誤解されていそうだったので記事を書いてみました。

ColimaをはじめとするMac上の仮想化技術は、Docker Desktopの有料化が発表されたタイミングで、無料で継続して使うための「引っ越し先」として注目度が高まったように感じます。

しかしながら完全な引っ越しではなく「使い分け」をすることで、**利便性を損なうことなくIntel Macへの未練を断ち切ること**ができます。

M1 Macでの開発生産性を最大化するために、**普段はDocker Desktopを使い倒して、x86アーキテクチャに向き合う場合はColimaを活用する**のがオススメです。

:::message
**Colimaは決して「速く」はない**

M1,M2 Macでもx86アーキテクチャで動かせるColimaはとても便利ですが、本来はARMアーキテクチャで動かせないものを、仮想化技術で無理矢理動かしています。その上にさらにDockerを起動させているので、当然ながら動作は決して速くありません。

というより、体感でも分かるくらいに「遅い」です。

本格的に開発環境を整えるなら、M1,M2 MacでなくIntel MacやWindowsなどx86アーキテクチャの開発端末をもう1つ用意するのが無難です。

あくまで「使い分け」ができる用途に限った範囲で、という点を最後に補足いたします。
:::
