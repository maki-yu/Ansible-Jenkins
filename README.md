# Ansible/Jenkinsを用いたRuby on Railsの環境構築

## 構造
- ディレクトリ構造
```
.
├── README.md
└── ansible
    ├── ansible.cfg
    ├── hosts
    ├── playbook.yml
    └── roles
        ├── common
        │   └── tasks
        │       └── main.yml
        ├── mysql
        │   └── tasks
        │       └── main.yml
        ├── nginx
        │   └── tasks
        │       └── main.yml
        ├── nodejs
        │   └── tasks
        │       └── main.yml
        └── ruby
            ├── tasks
            │   └── main.yml
            └── templates
                └── rbenv_system.sh.j2
```

## 事前準備
- Jenkins実行用のサーバーを1個（今回はEC2を使用）
- 紐づいていないElasticIPを1個

## 実行手順
### 1. 用意したサーバーにJenkinsをインストール
```
$ sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
$ sudo yum upgrade
$ sudo yum install jenkins java-1.8.0-openjdk-devel.x86_64
$ sudo systemctl daemon-reload
```

### 2. Jenkins実行
```
$ sudo systemctl start jenkins

$ sudo systemctl status jenkins
# Active:active(running)になっていればOK
```

### 3. Jenkinsにアクセス
- 以下URLより、Jenkinsにアクセス
```
http://<jenkinsが起動しているサーバのIPv4アドレス>:8080
```
- 初期パスワードの入力、プラグインのインストール、初期設定、インスタンスURLの設定を実行

### 4. ジョブ実行ユーザーの変更
- 以下ファイルの"JENKINS_USER"を変更
```
$ sudo vi /etc/sysconfig/jenkins

# "jenkins"から"ec2-user"に変更する
JENKINS_USER="jenkins"
 　　　　　↓↓
JENKINS_USER="ec2-user"
```
- ジョブ実行に関するディレクトリやファイルの権限を変更する
```
$ sudo chown -R ec2-user: /var/lib/jenkins /var/log/jenkins /var/cache/jenkins
```
- Jenkinsのプロセスを再起動する
```
$ sudo systemctl restart jenkins.service
```

### 5. GitHubにSSH接続するための設定
**GitHubのリポジトリからcloneする際にSSH接続をするため、設定を行う**

- gitをインストール
```
$ sudo yum install -y git
```
- SSHキーの作成
```
$ ssh-keygen -t rsa -b 4096
```
- 公開鍵をGitHubに登録
- 接続を確認
```
$ ssh -T git@github.com
# Hi! name〜 と表示されたら完了
```

### 6. AWS CLIの設定
**JenkinsでCFnのスタックを作成するために、AWS CLIの設定を行う**

- Jenkins用にIAMユーザーを作成し、secret keyを作成
- Credentialの設定
```
$ aws configure
AWS Access Key ID [None]: 
AWS Secret Access Key [None]: 
Default region name [None]: 
Default output format [None]: 
```

### 7. ジョブ作成とビルド実行
- 下記のジョブを作成する
```
- CFnのテンプレートをリポジトリよりclone
- CFnのスタック作成
- Ansibleのテンプレートをリポジトリよりclone
- Ansible実行
```
- 上記のジョブを繋ぎ、ビルドを実行する
