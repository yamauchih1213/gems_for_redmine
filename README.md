# gems_for_redmine
Redmineをオンプレミス環境で構築するために必要となるgem(s)をまとめたもの

## Usage
vendor/cacheをオンプレミスのRedmineのアプリケーションルートディレクトリに配置し以下を実行する  
(--localを指定することでvendor/cacheのgem(s)を参照させる)
```bash
bundle install --local
```

## Reference
### 1. How to make this
* rubygems.orgに接続できるオンライン環境で一度Redmineを構築する
* bundle package を実行して、必要となるgem(s)をvendor/cacheにまとめる

### 2. How to install Redmine on an "ONLINE environment"

#### ●構成概要
|種別|バージョン等|補足|
|:--|:--|:--|
|OS| Oracle Linux 8.7|WSL 2.3.26/Win11で実行|
|Redmine|5.1.5||
|Ruby|3.0.7p220||
|PostgreSQL|12.22||
|MySQL|8.0.36|実際には利用しない,gem(s)取得のためにライブラリのみ導入|

#### ●OSインストール
コマンド実行後、プロンプトに従って、ユーザ名とパスワードを設定する
```PowerShell
wsl --install -d OracleLinux_8_7
```

#### ●OS基本設定
```bash
sudo yum install -y git
sudo yum install -y bzip2 gcc openssl-devel readline-devel zlib-devel unzip
sudo yum groupinstall -y "Development Tools"
```

#### ●Ruby 3のインストール
```bash
sudo dnf module -y reset ruby
sudo dnf module -y enable ruby:3.0
sudo dnf module -y install ruby:3.0/common
sudo yum install -y ruby-devel
```

#### ●PostgreSQLのインストール/設定/起動
```bash
sudo dnf module -y reset postgresql
sudo dnf module -y enable postgresql:12
sudo dnf module -y install postgresql:12
sudo yum install -y libpq-devel
sudo  -i -u postgres
# ユーザpostgresでの操作ここから
initdb -D /var/lib/pgsql/data
pg_ctl start
psql
-- psql内の操作ここから
CREATE ROLE redmine LOGIN ENCRYPTED PASSWORD 'my_password' NOINHERIT VALID UNTIL 'infinity';
CREATE DATABASE redmine WITH ENCODING='UTF8' OWNER=redmine;
quit
# psql内の操作ここまで
exit
# ユーザpostgresでの操作ここまで
```

#### ●Redmineのインストール
```bash
cd
unzip /tmp/redmine-5.1.5.zip
cd redmine-5.1.5/config
cp -p database.yml.example database.yml
vi database.yml
```
database.ymlの記載内容
```yml
# PostgreSQL configuration example
production:
  adapter: postgresql
  database: redmine
  host: localhost
  username: postgres
  password: "postgres"
```

#### ●[Optional] MySQLライブラリのインストール
* 実際には使わないが、いざMySQLを使う際に必要となるgem(s)を持っておきたかったので実施した内容
* database.yml内にMySQL向けのエントリが存在している状態でbundle installする必要がある
* MySQLを使わないなら、bundle installの実施後にMySQL向けのエントリをdatabase.xmlから削除すること
```bash
sudo yum install -y mysql-devel
```

#### ●Redmineで利用するgem(s)のインストール
・準備
```bash
cd ~/redmine-5.1.5
gem install bundler
bundle config set --local path vendor/bundle
cat <<EOF > Gemfile.local
# Gemfile.local
gem 'puma'
EOF
```

・オンラインインストールの場合(rubygemsに接続可能)
```bash
bundle install
```
・オフラインインストールの場合(vendor/cacheに格納したgemファイルを使う)
```bash
bundle install --local
```

#### ●Redmineの設定/起動
[Redmine公式のインストール手順](https://www.redmine.org/projects/redmine/wiki/RedmineInstall)のStep 5以降を実施する


### 3. Additional Info
* 今回対象とした環境
  * [Redmine公式のインストール手順](https://www.redmine.org/projects/redmine/wiki/RedmineInstall)では
  	下記のようにdevelopment/testの環境を除外するが、より広範にgem(s)を収集するために下記コマンドを実施していない
```bash
bundle config set --local without 'development test'
```
* concurrent-ruby-1.3.5の不具合
  * 2025年1月26日現在の最新であるconcurrent-ruby-1.3.5には不具合があり、Redmine公式のインストール手順の「Step 5 - Session store secret generation」が失敗する 
	([参考リンク](https://qiita.com/Taira0222/items/89fe772eb8d752da4db7))
  * このため、今回のgem(s)収集に際しては、バージョンを1.3.4に指定する行をRedmineのGemfileに追加してbundle installを行った
```
gem 'concurrent-ruby', '1.3.4'
```

* wsl操作コマンドあれこれ

|操作|コマンド|補足|
|:--|:--|:--|
|利用可能ディストリビューションの一覧|wsl -l --online||
|インストール|wsl --install -d OracleLinux_8_7||
|停止|wsl -t OracleLinux_8_7||
|エクスポート|wsl --export  OracleLinux_8_7 c:/redmine/OracleLinux_8_7.tar|停止状態で実施する|
|削除|wsl --unregister OracleLinux_8_7||


### 4. Links
* [Installing Redmine](https://www.redmine.org/projects/redmine/wiki/RedmineInstall)
* [setup Oracle Linux 9.1 for wsl2 on windows11](https://end0tknr.hateblo.jp/entry/20240102/1704150817#install-Oracle-Linux-91)
* [Ruby 3.1 : インストール](https://www.server-world.info/query?os=CentOS_Stream_8&p=ruby&f=8)
* [bundlerでgemをプロジェクトごとに管理する](https://dev.classmethod.jp/articles/bundler-gem-management/)
