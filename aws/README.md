ECS を AWS CloudFormation を使って構築する
===

Author: Shigeki Shoji
Twitter: @takesection

---

# AWS CloudFormation

* [AWS CloudFormation](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html) を使ってリソースのプロビジョニングや設定を行うことができます。
* テンプレートは、JSON または YAML 形式のドキュメントです。

---

# ECS

[ECS](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html) は、Docker コンテナを簡単に実行、停止、管理できるコンテナ管理サービスです。

---

# ECS の構成

ECS では Docker イメージを EC2 インスタンスの論理グループであるクラスターにタスクを配置します。
クラスターは複数の EC2 インスタンスを持ち、各 EC2 インスタンスで Docker イメージを実行します。

---

# ECS Cluster を作成するテンプレート

```
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  ECSCluster:
    Type: AWS::ECS::Cluster
    Properties:
      ClusterName: "ECSCluster"
```

---

# ECS が使用する EC2 インスタンス

ECS が使用する EC2 インスタンスには、コンテナエージェントが必要です。AWS では、必要なものが事前設定され最適化された AMI があるのでそれを使用します。

アプリケーションロードバランサを使用する場合は、次のような記述となります。

```
  ECSInstances:
    Type: AWS::AutoScaling::LaunchConfiguration
    Properties:
      ImageId: ami-f63f6f91
      InstanceType: t2.micro
      IamInstanceProfile: !Ref 'EC2InstanceProfile'
      AssociatePublicIpAddress: true
      KeyName: !Ref 'KeyName'
      SecurityGroups:
      - !Ref 'EcsSecurityGroup'
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash -xe
          echo ECS_CLUSTER=ECSCluster >> /etc/ecs/ecs.config
          yum install -y aws-cfn-bootstrap
          /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --region ${AWS::Region}
```

---

## テンプレートの説明

ECS のインスタンスは、ECS クラスターにインターネット接続で登録できるようにする必要があります。
ここで説明しているテンプレートではインターネットゲートウェイを使用しています。

* ImageId で Amazon ECS に最適化された AMI(Tokyoリージョン)を指定しています。
* SSH でこのコンテナにアクセスするために KeyName と Public IP アドレスを設定(AssociatePublicIpAddress)。

---

# ALB の設定

インターネットからは、HTTPS を使用したいため、リスナーは HTTPS とします。

```
  ALBListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    DependsOn: ECSServiceRole
    Properties:
      DefaultActions:
      - Type: forward
        TargetGroupArn: !Ref 'ECSTG'
      LoadBalancerArn: !Ref 'ECSALB'
      Port: '443'
      Protocol: HTTPS
      Certificates:
      - CertificateArn: !Ref 'Certificate'
```

---

## テンプレートの説明

* HTTPS で着信するために Protocol を HTTPS、Port を 443 に設定します。
* HTTPS のためには SSL 証明書が必要になるため、AWS Certificate Manager に登録された証明書の ARN を設定します。

---

# テンプレート

* [ECS Cluster](https://github.com/takesection/docker-examples/aws/cloudformation/ecscluster.yaml)
* [ECS](https://github.com/takesection/docker-examples/aws/cloudformation/ecs.yaml)
