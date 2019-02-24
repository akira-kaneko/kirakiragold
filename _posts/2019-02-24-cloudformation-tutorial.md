---
layout: post
title: "AWS CloudFormationを試してみた"
description: "DevelopersIOの記事を参考にしながらCloudFormationに入門してみました"
comments: false
keywords: "aws, cloudformation,　入門"
---

## 背景

AWSはコンソールから利用できる全ての操作をAPIとして提供しており、それらをプログラム側で制御することでインフラ構築・運用を自動化することができる。
（[Infrastructure as Code](https://www.imagazine.co.jp/infrastructure-as-code%E3%81%AE%E7%95%99%E6%84%8F%E7%82%B9%E3%81%A8%E3%83%A1%E3%83%AA%E3%83%83%E3%83%88%E3%80%80%EF%BD%9E%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E6%9B%B4%E6%94%B9%E3%83%97%E3%83%AD/){:target="_blank"}の実現）CloudFormationはそんな環境を提供してくれるAWSサービスの１つ。

## 主な特徴

- AWSの構成をテンプレート化して再利用性を高められる
- テンプレートをGit管理することで変更履歴の追跡性を高められる
- [ほぼ全てのAWSリソースの操作が可能](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/cfn-supported-resources.html){:target="_blank"}

## どんな場面で使うのか

- 新規環境の構築する時
- 既存環境をテンプレート化して再利用したい時
- [AWSリソースの操作は可能な限りCloudFormationを使うと良い](https://qiita.com/datake914/items/14cb4d66570c1efcbc7d){:target="_blank"}

## はじめに知っておきたいテンプレートの構造
JSONとYAML形式が使えるが、可読性の良さやコメントが書けるのでYAMLがおすすめ

参考：[プログラマーのための YAML 入門 (初級編)](https://magazine.rubyist.net/articles/0009/0009-YAML.html){:target="_blank"}

``` yaml
AWSTemplateFormatVersion: "2010-09-09"

Description: ここにテンプレートの説明を書く
 
Parameters:
  # スタック作成時にパラメーターとして任意の値を渡せる
 
Mappings:
  # ハッシュテーブルのようにキーに応じて値を設定できる

Conditions:
  # 条件判断を行い条件に一致した場合のみ実行されるリソースを指定

Resources:
  # 作成するリソースを定義する
 
Outputs:
  # テンプレートで生成したものの結果を出力させる
```

参考：[テンプレートの分析 - AWS CloudFormation](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/template-anatomy.html){:target="_blank"}

## VPCだけを作成する単純なテンプレートを触ってみる

参考：[【CloudFormation入門1】5分と6行で始めるAWS CloudFormationテンプレートによるインフラ構築 ｜ DevelopersIO](https://dev.classmethod.jp/cloud/aws/cloudformation-beginner01/){:target="_blank"}

**今回作成するイメージ**

![simple_vpc_image](https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2017/09/100.png)

**テンプレートファイルの内容**

[リソース - AWS CloudFormation](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/resources-section-structure.html){:target="_blank"}

``` yaml
AWSTemplateFormatVersion: "2010-09-09"
Resources:
  VPC: # 任意の名前
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
```

上記の内容を `kaneko_vpc.yaml` という名前でファイルに保存 

1. CloudFormationを開く
2. 新しいスタックの作成
3. テンプレート選択でファイルの選択から `kaneko_vpc.yaml` をAmazon S3にアップロード
4. スタックの名前に `kaneko-vpc` と入力
5. オプションはスキップ
6. 確認画面で作成を押す

ダッシュボードで状況が `CREATE_COMPLETE` になれば完了

VPCのダッシュボードに遷移して作成されたVPCを選択しタグに切り替えると、どのスタックに所属しているかがわかる

|キー|値|
| --- | --- |
|aws:cloudformation:logical-id|VPC|
|aws:cloudformation:stack-id|arn:aws:cloudformation:ap-southeast-1:hogehoge|
|aws:cloudformation:stack-name|kaneko-vpc|

**スタックの削除方法**

ダッシュボードのアクションからスタックの削除からできる

## 単純なテンプレートを変更してみる

基本的にリファレンスを見ながら設定できる項目をいじったりする

[AWS::EC2::VPC - AWS CloudFormation](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-vpc.html){:target="_blank"}

**タグを設定してみるテンプレート**

``` yaml
AWSTemplateFormatVersion: "2010-09-09"
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      Tags: # Nameキーを指定するとVPCのName欄に名前が出る
      - Key: Name
        Value: Kaneko-VPC
```

先ほど作った値にタグが設定されていることがわかる

## VPCにサブネット/ルーティングテーブル/インターネットゲートウェイを構築してみる

**今回作成するイメージ**

![110.png](https://cdn-ssl-devio-img.classmethod.jp/wp-content/uploads/2017/09/110.png)

**テンプレート**

``` yaml
AWSTemplateFormatVersion: "2010-09-09"
Resources:
  VPC: # https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-vpc.html 
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      Tags:
      - Key: Name
        Value: Kaneko-VPC
  InternetGateway: # https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-internetgateway.html
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
      - Key: Name
        Value: Kaneko-IGW
  AttachGateway: # https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-vpc-gateway-attachment.html
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway
  RouteTable: # https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-route-table.html
    Type: AWS::EC2::RouteTable
    DependsOn: AttachGateway
    Properties:
      VpcId: !Ref VPC
      Tags:
      - Key: Name
        Value: Kaneko-Route
  Route: # https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-route.html
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref RouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
  Subnet: # https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnet.html
    Type: AWS::EC2::Subnet
    DependsOn: AttachGateway
    Properties:
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: "true"
      VpcId: !Ref VPC
      Tags:
      - Key: Name
        Value: Kaneko-Subnet
  SubnetRouteTableAssociation: # https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnet-route-table-assoc.html
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref Subnet
      RouteTableId: !Ref RouteTable
```

**!Ref（組み込み関数）**

指定したリソースの値を参照できる。例えば AttachGateway のところで VpcId や InternetGatewayId を指定しなければならないが、構築されるまではIDが分からないため、代わりに!Refを使って値を参照するようにできる（ほぼ必須）

[Ref - AWS CloudFormation](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-ref.html){:target="_blank"}

**DependsOn（リソース依存関係の定義）**

特定リソースが他のリソースに続けて作成されるように指定できる。例えば RouteTable のところで InternetGateway のアタッチが終わってから設定するようにさせる。VPCなどの一部の項目にはこの設定が必須のものがある。

[DependsOn 属性 - AWS CloudFormation](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-attribute-dependson.html){:target="_blank"}

## おまけ
今の手順をコンソールから手動で行ってみる

1. VPCの作成
2. インターネットゲートウェイの作成
3. インターネットゲートウェイをVPCにアタッチ
4. VPCにルートテーブルを作成
5. ルートテーブルにインターネットゲートウェイへのルートを追加
6. VPCにサブネットを作成し、パブリックIPの自動割り当てを有効にする
7. サブネットをルートテーブルに関連づける

**時間があったらCloudFormationデザイナーで遊んでみると面白そう**
