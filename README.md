# Terraform入門

## 参考にしている記事

- https://dev.classmethod.jp/articles/terraform-getting-started-with-aws/
- https://qiita.com/minamijoyo/items/1f57c62bed781ab8f4d7
- https://qiita.com/litencatt/items/8da20374def6320a014b
- https://qiita.com/Chanmoro/items/55bf0da3aaf37dc26f73
- https://reboooot.net/post/what-is-terraform/

## 初期設定

以下を入れて適宜パスを通す。

- https://www.terraform.io/downloads.html
- https://github.com/terraform-linters/tflint

以下でチェック。

```
terraform -v
tflint -v
```

## VSCodeの設定

以下を入れる。

https://marketplace.visualstudio.com/items?itemName=mauve.terraform

```.vscode/settings.json
{
  "editor.formatOnSave": true,
}
```

以上で自動フォーマット、Lintingをしてくれる。(`terraform fmt`でもフォーマットしてくれる)

- https://qiita.com/pypypyo14/items/5520f3defa55119f3a1a
- https://qiita.com/garakutayama/items/0cdf43816bde2378f4f9

## 組み込み関数をためす

以下のコマンドで対話形式でTerraformの組み込み関数を試すことができる。

```
terraform console
```

## テンプレートファイルを作成する

テンプレートはHCL(HashiCorp Configuration Language)というHashiCorpの独自言語を使う。
`*.tf`という名前のファイルになる。(`.tf.json`という形式でもかけるらしい)

テンプレートにはまずプロバイダーの設定を記述する。
どのプロバイダーを使うかの宣言となる。

```
provider "aws" {
  version = "~> 2.0"
  access_key = "ACCESS_KEY_HERE"
  secret_key = "SECRET_KEY_HERE"
  region = "ap-northeast-1"
}
```

Providerをどのように設定するかは以下のようなドキュメントを参照する。(以下はAWS)

https://www.terraform.io/docs/providers/aws/index.html

## Terraformのバージョンについて

Terraformのバージョンを管理できる`tfenv`がある。(Nodeの`nvm`のポジションと同じ)

https://github.com/tfutils/tfenv

また以下のようにTerraformのバージョンを固定できる。

```main.tf
terraform {
  required_version = "= 0.10.7"
}
```

## テンプレートファイルの分割

`*.tf`ファイルを認識してくれるので、ファイルを分割することが可能。

## HCLのルール

以下のようなブロックの記述を複数記述するような構成になる。

```
{ブロック名} "{ブロックに対応する種類な名前など}" "{ブロックに対応する種類な名前など}" {
  {キー} = "{バリュー}"
  {キー} = "{バリュー}"
  {キー} {
    {キー} = "{バリュー}"
  }
  # ...
  # コメント
}
```

## キー情報などの管理/変数への値代入

プロバイダー(AWS等)へのキー情報は環境変数やTerraformの変数を用いる。(バージョン管理に混じると危険なため)

以下のように`variable`を用いて宣言することで、`var.*`で参照することができる。`${}`で展開できる。

```
variable "access_key" {}
variable "secret_key" {}
variable "region" {}

# 以下のようにデフォルト値を指定できる
# variable "region" {
#   default = "ap-northeast-1"
# }

provider "aws" {
  access_key = "${var.access_key}"
  secret_key = "${var.secret_key}"
  region     = "${var.region}"
}
```

### 変数への代入方法の3パターン

パターン1: コマンドオプション

```
terraform apply -var 'hoge_key=hoge_secret' -var 'piyo_key=piyo_secret'
```

パターン2: 環境変数`TF_VAR_*`を使う

```
export TF_VAR_hoge_key="hoge_secret"
export TF_VAR_piyo_key="piyo_secret"
```

パターン3: ファイルで値を渡す(公式推奨)(.gitignoreに追加する)

```terraform.tfvars
hoge_key = "hoge_secret"
piyo_key = "piyo_secret"
```

## アウトプット

以下のように`output`ブロックを使うとコマンド実行時に出力される。

```
output "説明" {
  value = "${var.hoge_key}"
}
```

## チェック、デプロイ、削除方法

ディレクトリ内でまず初期化する。
.tf ファイルで利用しているプラグインのダウンロード処理などが走り、`.terraform`直下に配置される。

```
terraform init
```

まずテンプレートに誤りがないか実行計画を確認する。
以下でリソースの作成/変更/削除の表示、構文エラー、ブロックに設定したパラメータ誤りなどチェックなどを行ってくれる。

また`terraform.tfstate`というリソースの状態を示すjsonファイルが出力される。

```
terraform plan
```

実際にテンプレートを利用してリソースを作成するには以下を実行する。
`terraform.tfstate`も更新される。(`terraform show`を用いるときれいに表示してくれる)

```
terraform apply
```

以下でリソースを削除できる。また削除計画も同様に`terraform plan -destroy`でチェックできる。

```
terraform destroy
```

## 条件分岐: production時とかで値をかえたい, リソースを作成する/しない

以下のように三項演算子を用いることができる。(if文はない)

```
resource "aws_instance" "web" {
  subnet = "${var.env == "production" ? var.prod_subnet : var.dev_subnet}"
}
```

リソースの`count`をいじって、作成する/しないを制御できる。

```
resource "aws_cloudwatch_log_group" "hoge" {
  count = "${var.create_log_group ? 1 : 0}"
  name  = "hoge"
}
```

## Terraformをデバッグする

https://qiita.com/minamijoyo/items/1f57c62bed781ab8f4d7#terraform%E3%82%92%E3%83%87%E3%83%90%E3%83%83%E3%82%B0%E3%81%99%E3%82%8B%E6%8A%80%E8%A1%93

## TODO

- AWSで実際につくってみる
- 組み込み関数を触ってみる
- よくあるパターンの構成を書いてみる
- モジュールを使った共通化
- lifecycleブロックを使う
- Terraform Module Resitoryを触ってみる https://registry.terraform.io/
- Remote Remote https://learn.hashicorp.com/terraform/getting-started/remote.html
