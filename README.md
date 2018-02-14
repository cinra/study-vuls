# Vuls

**課題**

- 脆弱性をなんとかしたい
  - 脆弱性情報を監視し続けるのはコストがかかる
  - サーバーにインストールされているソフトウェアは膨大
  - 該当箇所がよくわからない
- 手動で管理するの、ほぼ無理…

**Vuls**

- 脆弱性の検出〜監視を行ってくれるツール
  - 対象のサーバーに関係する脆弱性のみ教えてくれる
  - 自動でスキャンするので漏れがない
  - インターフェイスが充実

## Setup

### Docker Imageを用意

```sh
$ docker run --rm vuls/go-cve-dictionary -v
$ docker run --rm vuls/goval-dictionary -v
$ docker run --rm vuls/vuls -v
```

- **vuls/go-cve-dictionary**
- **vuls/goval-dictionary**

脆弱性のリストを管理するためのDocker Image

- **vuls/vuls**

Vuls本体

### Fetch NVD

最新の脆弱性情報を取得します。時間がかかるので、今回は2017〜2018年の脆弱性情報を取得。

> NVD（National Vulnerability Database）
アメリカ国立標準技術研究所（National Institute of Standards and Technology、通称NIST) が管理している脆弱性情報データベース


```sh
for i in `seq 2017 $(date +"%Y")`; do \
  docker run --rm -it \
  -v $PWD:/vuls \
  -v $PWD/log/go-cve-dictionary:/var/log/vuls \
  vuls/go-cve-dictionary fetchnvd -years $i; \
done
```

スキャン結果を日本語で出力したい場合は、JVNも取得しておきます。

> JVN（Japan Vulnerability Notes）
JPCERT/CCと情報処理推進機構（IPA）が共同で管理している脆弱性情報データベース

```
for i in `seq 2017 $(date +"%Y")`; do \
  docker run --rm -it \
  -v $PWD:/vuls \
  -v $PWD/log/go-cve-dictionary:/var/log/vuls \
  vuls/go-cve-dictionary fetchjvn -years $i; \
done
```

あと、Oval。今回はDebianのものをFetch。

> OVAL（Open Vulnerability and Assessment Language）

```sh
docker run --rm -it \
    -v $PWD:/vuls \
    -v $PWD/log/oval:/var/log/vuls \
    vuls/goval-dictionary  fetch-debian 8
```

### ターゲットコンテナ作成

脆弱性を診断する対象になるサーバー／Dockerコンテナを準備します。今回は、Dockerコンテナ。ちょっと古めのPHP入りApacheのDocker Imageを使います

```sh
$ docker-compose build
$ docker-compose up -d
```

### ログイン

```sh
$ docker exec -it vuls bash
```

## Usage

[ドキュメント](https://vuls.io/docs/ja/abstract.html)

### Scan

```sh
$ vuls scan
```

### Report

```sh
$ vuls report -format-full-text -lang=ja
```

### GUI

- https://github.com/usiusi360/vulsrepo
- https://usiusi360.github.io/vulsrepo/

```sh
docker run -dt \
    -v $PWD:/vuls \
    -v $PWD/log:/var/log \
    -p 5111:5111 \
    --name vulsrepo \
    vuls/vulsrepo
```

- http://localhost:5111/