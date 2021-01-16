Golang Cheat Sheet

<!-- TOC -->

- [Fundarmental](#fundarmental)
  - [Golan Installation](#golan-installation)
  - [GET GOROOT](#get-goroot)
  - [GOPATH](#gopath)
  - [Basic knowledge](#basic-knowledge)
- [Tools](#tools)
  - [go get](#go-get)
  - [vgo (a.k.a Modules)](#vgo-aka-modules)
    - [depからの移行](#depからの移行)
    - [ライブラリのMINORバージョン又はPATCHバージョンを更新](#ライブラリのminorバージョン又はpatchバージョンを更新)
    - [全ライブラリを最新versionに更新する手順](#全ライブラリを最新versionに更新する手順)
    - [使わなくなったライブラリを削除する手順](#使わなくなったライブラリを削除する手順)
    - [go.modの内容をvendorに反映させる](#gomodの内容をvendorに反映させる)
  - [dep](#dep)
  - [gore - Go REPL](#gore---go-repl)
  - [ghq](#ghq)
- [Go Language Rules](#go-language-rules)
  - [Debug Print](#debug-print)
  - [Variables](#variables)
  - [String](#string)
  - [Array and Slice](#array-and-slice)
  - [Map](#map)
  - [Function](#function)
  - [Closure](#closure)
  - [Struct](#struct)
  - [Pointer](#pointer)
  - [Pointer receiver and Value receiver](#pointer-receiver-and-value-receiver)
  - [Interface](#interface)
  - [Goroutine](#goroutine)
  - [Channel](#channel)
- [golang with VSCode](#golang-with-vscode)
  - [Install/Update Tools](#installupdate-tools)
  - [Go config in settings.json](#go-config-in-settingsjson)
- [LINKS](#links)

<!-- /TOC -->

## Fundarmental
- No classes, but structs with methods
- Interfaces
- No implementation inheritance. There's type embedding, though.
- Functions are first class citizens
- Functions can return multiple values
- Has closures
- Pointers, but not pointer arithmetic
- Built-in concurrency primitives: Goroutines and Channels

### Golan Installation
Linux
```
# download go
https://github.com/golang/go/releases
### STANDARD WAY
sudo tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
# set PATH
# Add /usr/local/go/bin to PATH with /etc/profile or $HOME/.profile
export GO_HOME=/usr/local/go
export GOROOT=/usr/local/go
export PATH=$PATH:$GO_HOME/bin
```
MacOS
```
# (1) install go by brew
brew install go
# or
brew upgrade go

# (2) Env variables
# /etc/profile or $HOME/.profile
export GOROOT=/usr/local/opt/go/libexec
export GOPATH=$HOME
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

### GET GOROOT
```sh
$ go env GOROOT

/usr/local/opt/go/libexec
```
### GOPATH
```sh
export GOPATH=$HOME/dev/go
export PATH=$PATH:$GOPATH/bin
```
check $GOPATH
```sh
$ ls $GOPATH

bin pkg src
```

### Basic knowledge
3 dot
```
go [command] ./...
Here ./ tells to start from the current folder, ... tells to go down recursively.
```
> example
```
# 再起的test実行
go test ./...
# 再起的go list
go list ./... 
```

## Tools
### go get
Install go package directory with `go get`. 
```
go get go.opencensus.io/examples/http/...
# [NOTE]: triple dot means install all sub packages as well
# See this for more detail: https://golang.org/cmd/go/
```

Useful packages:
```
# go-outline
go get -v github.com/ramya-rao-a/go-outline
# gocode
go get -v github.com/stamblerre/gocode
# gopkgs
go get -v github.com/uudashr/gopkgs/cmd/gopkgs
# go-symbols
go get -v github.com/acroca/go-symbols
# guru
go get -v golang.org/x/tools/cmd/guru
# gorename
go get -v golang.org/x/tools/cmd/gorename
# golint
go get -v golang.org/x/lint/golint
# godef
go get -v github.com/rogpeppe/godef
# goimports
go get -v golang.org/x/tools/cmd/goimports
```

### vgo (a.k.a Modules)

> `Go 1.11`から(Versioned) Go Modules (a.k.a その実装である `vgo`)が利用できるようになっている。 `vgo` が有効化された go を使うと、 go build のタイミングで勝手に必要なモジュールバージョンをダウンロードしてくれる。 dep 等だと vendor に該当モジュールをダウンロードする前提だったけど、 vgo の場合はデフォルトでは vendor を利用しない。$GOPATH/go/mod 以下にバージョニングされた状態でダウンロードされて、vgo を利用する全プロジェクトはそこを参照するようになる
> Go 1.11なら `GO111MODULES=on go build` のように go コマンドを実行すると利用できる。Go 1.12以降はデフォルトで `GO111MODULES=auto` の扱いとなっている。auto の場合は 後述する GOPATH 以下のディレクトリの場合は vgo が無効化され、 GOPATH の外なら vgo が有効化された状態になる

```go
# Go 1.11 - 
export GO111MODULES=on
go build

# Go 1.12 - by default
export GO111MODULES=auto # auto の場合は 後述する GOPATH 以下のディレクトリの場合は vgo が無効化され、 GOPATH の外なら vgo が有効化された状態
```
(Reference: [他言語から来た人がGoを使い始めてすぐハマったこととその答え](https://qiita.com/mumoshu/items/0d2f2a13c6e9fc8da2a4#vgo))

moduleを使うときは、`go.mod`ファイルをGoプログラムのルートディレクトリに置きます。 go.modファイルではモジュール名の宣言と、依存モジュールを記述することができます。go buildなどモジュールのimportが必要になれば`$GOPATH/pkg/mod/`以下にバージョン別に整理されてダウンロードされ、参照できるようになります。 自分のプログラム内でimportするときもGOPATHの下で作業をしなくてもモジュール名を使ってimportができます。 
(see [Go modulesの話](https://techblog.asahi-net.co.jp/entry/2019/03/11/122341))

`newmod`という名前のmoduleでプロジェクトを開始する
```bash
mkdir newmod
cd newmod
# go mod init <moduleName>
go mod init newmod
# -> go.mod ファイルが作られる

# xxx.goファイルを編集してBuildする
go build newmod
# -> go.modの中にrequireが必要なモジュールが記入される
```
例） サンプルで作ったmain.goをmodulesでやってみる
```bash
ls -1 
# -> main.go
go mod init main
# -> go.modが作られる
go build main
# -> go.modにrequireなパッケージが記入される
cat go.mod

-------------------------
module main

go 1.13

require (
        cloud.google.com/go/datastore v1.0.0
        google.golang.org/api v0.13.0
)
-------------------------
```

#### depからの移行

`go.mod`ファイルを生成する`go mod init`コマンドではdepで使われる`Gopkg.lock`ファイルがあればそれを解釈して`go.mod`に反映させる機能がある

すでに`Gopkg.lock`がある状態で
```
# GO111MODULE=on go mod init <modulename>
GO111MODULE=on go mod init mymod

Gopkg.lock
Gopkg.toml
README.md
go.mod      <<<<<<新しく作られた
main.go
vendor
```
中身を覗いてみると
```
module main

go 1.12

require (
        cloud.google.com/go v0.47.0
        github.com/BurntSushi/toml v0.3.1
        github.com/golang/groupcache v0.0.0-20191027212112-611e8accdfc9
        github.com/golang/protobuf v1.3.2
        github.com/googleapis/gax-go v1.0.3
        github.com/jstemmer/go-junit-report v0.9.1
        go.opencensus.io v0.22.1
        golang.org/x/exp v0.0.0-20191030013958-a1ab85dbe136
        golang.org/x/lint v0.0.0-20190930215403-16217165b5de
        golang.org/x/net v0.0.0-20191101175033-0deb6923b6d9
        golang.org/x/oauth2 v0.0.0-20190604053449-0f29369cfe45
        golang.org/x/sys v0.0.0-20191029155521-f43be2a4598c
        golang.org/x/text v0.3.2
        golang.org/x/tools v0.0.0-20191101200257-8dbcdeb83d3f
        google.golang.org/api v0.13.0
        google.golang.org/appengine v1.6.5
        google.golang.org/genproto v0.0.0-20191028173616-919d9bdd9fe6
        google.golang.org/grpc v1.24.0
        honnef.co/go/tools v0.0.1-2019.2.3
)
```
(see [Go modulesの話](https://techblog.asahi-net.co.jp/entry/2019/03/11/122341))


#### ライブラリのMINORバージョン又はPATCHバージョンを更新
From [Go: module modeでのgo.mod, go.sumファイルを用いた依存ライブラリ管理、及びcontributionの方法＆手順](https://horizoon.jp/post/2019/04/18/contributing_with_gomodules/)

```bash
# v1.19.12 という最新バージョンが存在している事が分かる
$ go list -u -m github.com/aws/aws-sdk-go
github.com/aws/aws-sdk-go v1.17.14 [v1.19.12]

# v1.18.x にマッチする最新バージョン、に更新してみる
$ go get github.com/aws/aws-sdk-go@'<v1.19'
go: finding github.com/aws/aws-sdk-go v1.18.6
go: downloading github.com/aws/aws-sdk-go v1.18.6
go: extracting github.com/aws/aws-sdk-go v1.18.6

# go.modの更新内容を確認
$ cat go.mod | grep aws-sdk-go
        github.com/aws/aws-sdk-go v1.18.6
```
> [補足] go get PACKAGE@VERSION - 指定バージョンを取得
> - `go get foo@latest` - 現在取得可能な最新バージョン
> - `go get foo@v1.6.2` - 指定したバージョン
> - `go get foo@e3702bed2` - 指定したcommit hash（のリビジョン）
> - `go get foo@'<v1.6.2'` - 指定した条件に合致する最新バージョン
> - `go get foo@master` - masterブランチの内容

#### 全ライブラリを最新versionに更新する手順
From [Go: module modeでのgo.mod, go.sumファイルを用いた依存ライブラリ管理、及びcontributionの方法＆手順](https://horizoon.jp/post/2019/04/18/contributing_with_gomodules/)

- `go list -u -m all` - 全依存ライブラリに更新可能バージョン（があればそれ）を付けて一覧表示・確認
- `go get -u`
- `go build` or `go run` or `go test` を実行する（これでgo.mod, go.sumも最新化される）

#### 使わなくなったライブラリを削除する手順
- コードを編集・import文を削除
- `go mod tidy`
- `go build` or `go run` or `go test` （これでgo.mod, go.sumも最新化される）

#### go.modの内容をvendorに反映させる
- go mod vendor

go.mod, go.sumがあるmodulesプロジェクトでgo mod vendor実行で依存パッケージが`vendor`にコピーされる

> 例: go.modの内容をvendorに反映させて、go testでvendorを利用する
```bash
ls -la
  go.mod go.sum ..
# 依存をvendorにコピー
go mod vendor
# vendorの内容でテストする
go test ./... -mod=vendor
```

### dep
Goでパッケージの依存関係を自動で解決して管理するソフトウェア. Goでは<パッケージのルート>/vendor/をimport pathとみなすことができるので、 vendor/ にパッケージを保存して特定のバージョンを固定して使うことができます(vendoring)。
プロジェクト固有のライブラリは $GOPATH/src/プロジェクト/vendor の下に入れても良いことになっている。そうすると go コマンドは vendor 以下を優先して見てくれる。面白い事に、vendor ディレクトリの管理は govendor 等色々なサードパーティツールを使う (see [Go modulesの話](https://techblog.asahi-net.co.jp/entry/2019/03/11/122341))
> NOTE: 
> Go 1.12 からこのやり方を改め、$GOPATH/src や vendor は廃止となり Go Modules (vgo) という物を採用する事になった。Go 1.11の段階では移行期だが、環境変数 GO111MODULE (`export GO111MODULE=on `)により Go Modules を試す事が出来る

Install `dep` using `go get`
```sh
go get -u github.com/golang/dep/cmd/dep
```
Create new project
```
mkdir $GOPATH/src/github.com/trydep
cd $GOPATH/src/github.com/trydep
# Manifest, Gopkg.toml and lock file, Gopkg.lock, and vendor directory will be created
dep init
```
- `Gopkg.toml`: application's dependancy list
- `Gopng.lock`
- `vendor`: all packages are installed in this dir

How to use:
```sh
# inital kick to generate `Gopkg.toml` and `Gopkg.lock`
dep init

# install project's dependancies under `vender` directory
dep ensure

# add a dependancy to the project
dep ensure -add github.com/foo/bar

# update the locked versions of all dependencies
dep ensure -update
# update specified package
dep ensure -update github.com/foo/bar

# show current status
# dep status command give you info on projects dependencies and which packages are missing, etcx
dep status
```

Typical Project build using dep is like this:
> kedacore/keda
```bash
go get -v github.com/kedacore/keda
cd  $GOPATH/src/github.com/kedacore/keda
# install project's dependancies under `vender` directory
dep ensure
make build
```

LINKS
- https://golang.github.io/dep/


### gore - Go REPL
`REPL` is `Real-event-print loop`
```sh
$ go get github.com/motemen/gore
$ gore -autoimport
gore version 0.3.0  :help for help

gore> fmt.Println("Hello World")
HelloHello WorldHello World
12
<nil>
```

source: https://github.com/motemen/gore 


### ghq
`ghq` is a very helpful tool in putting go lang source codes under `$GOPATH/src`. 
source: https://github.com/motemen/ghq.

`ghq` gets source code with the same rule as `go get` and allows to show a list of all the repository under `$GOPATH/src` with `ghq list`.

```sh
# install
$ brew install ghq
# set ghq.root
$ git config --global ghq.root $GOPATH/src
```

Then, you're ready to use
```sh
$ ghq get owner/repo
$ ghq get https://github.com/owner/repo
$ ghq get git@git.example.com:path/to/repo.git
```

## Go Language Rules

### Debug Print

- `fmt.Printf("%v", val)`: the value in a default format when printing structs, the plus flag (%+v) adds field names
- `fmt.Printf("%#v", val)`: a Go-syntax representation of the value
- `fmt.Printf("%T", val)`: a Go-syntax representation of the type of the value

for more detail, see: https://golang.org/pkg/fmt/

`fmt.Printf("%#v", val)`でk8s node構造体をダンプした結果例
```yaml
Node dump: v1.Node{TypeMeta:v1.TypeMeta{Kind:"", APIVersion:""}, ObjectMeta:v1.ObjectMeta{Name:"ip-192-168-10-234.ap-northeast-1.compute.internal", GenerateName:"", Namespace:"", SelfLink:"/api/v1/nodes/ip-192-168-10-234.ap-northeast-1.compute.internal", UID:"f87d728b-3855-4347-b3fd-9446bc6a02d4", ResourceVersion:"865186", Generation:0, CreationTimestamp:v1.Time{Time:time.Time{wall:0x0, ext:63745067050, loc:(*time.Location)(0x2bec460)}}, DeletionTimestamp:(*v1.Time)(nil), DeletionGracePeriodSeconds:(*int64)(nil), Labels:map[string]string{"alpha.eksctl.io/cluster-name":"eks-test01", "alpha.eksctl.io/nodegroup-name":"standard-workers", "beta.kubernetes.io/arch":"amd64", "beta.kubernetes.io/instance-type":"m5.large", "beta.kubernetes.io/os":"linux", "eks.amazonaws.com/capacityType":"ON_DEMAND", "eks.amazonaws.com/nodegroup":"standard-workers", "eks.amazonaws.com/nodegroup-image":"ami-05d934c4ed7221f07", "eks.amazonaws.com/sourceLaunchTemplateId":"lt-062c3ffa7ac0825fb", "eks.amazonaws.com/sourceLaunchTemplateVersion":"1", "failure-domain.beta.kubernetes.io/region":"ap-northeast-1", "failure-domain.beta.kubernetes.io/zone":"ap-northeast-1c", "kubernetes.io/arch":"amd64", "kubernetes.io/hostname":"ip-192-168-10-234.ap-northeast-1.compute.internal", "kubernetes.io/os":"linux", "node.kubernetes.io/instance-type":"m5.large", "topology.kubernetes.io/region":"ap-northeast-1", "topology.kubernetes.io/zone":"ap-northeast-1c"}, Annotations:map[string]string{"node.alpha.kubernetes.io/ttl":"0", "volumes.kubernetes.io/controller-managed-attach-detach":"true"}, OwnerReferences:[]v1.OwnerReference(nil), Finalizers:[]string(nil), ClusterName:"", ManagedFields:[]v1.ManagedFieldsEntry{v1.ManagedFieldsEntry{Manager:"kube-controller-manager", Operation:"Update", APIVersion:"v1", Time:(*v1.Time)(0xc0002ea800), FieldsType:"FieldsV1", FieldsV1:(*v1.FieldsV1)(0xc0002ea820)}, v1.ManagedFieldsEntry{Manager:"kubelet", Operation:"Update", APIVersion:"v1", Time:(*v1.Time)(0xc0002ea8a0), FieldsType:"FieldsV1", FieldsV1:(*v1.FieldsV1)(0xc0002ea8c0)}}}, Spec:v1.NodeSpec{PodCIDR:"", PodCIDRs:[]string(nil), ProviderID:"aws:///ap-northeast-1c/i-009af4df315f8acbb", Unschedulable:false, Taints:[]v1.Taint(nil), ConfigSource:(*v1.NodeConfigSource)(nil), DoNotUseExternalID:""}, Status:v1.NodeStatus{Capacity:v1.ResourceList{"attachable-volumes-aws-ebs":resource.Quantity{i:resource.int64Amount{value:25, scale:0}, d:resource.infDecAmount{Dec:(*inf.Dec)(nil)}, s:"25", Format:"DecimalSI"}, "cpu":resource.Quantity{i:resource.int64Amount{value:2, scale:0}, d:resource.infDecAmount{Dec:(*inf.Dec)(nil)}, s:"2", Format:"DecimalSI"}, "ephemeral-storage":resource.Quantity{i:resource.int64Amount{value:85886742528, scale:0}, d:resource.infDecAmount{Dec:(*inf.Dec)(nil)}, s:"83873772Ki", Format:"BinarySI"}, "hugepages-1Gi":resource.Quantity{i:resource.int64Amount{value:0, scale:0}, d:resource.infDecAmount{Dec:(*inf.Dec)(nil)}, s:"0", Format:"DecimalSI"}, "hugepages-2Mi":resource.Quantity{i:resource.int64Amount{value:0, scale:0}, d:resource.infDecAmount{Dec:(*inf.Dec)(nil)}, s:"0", Format:"DecimalSI"}, "memory":resource.Quantity{i:resource.int64Amount{value:8141807616, scale:0}, d:resource.infDecAmount{Dec:(*inf.Dec)(nil)}, s:"", Format:"BinarySI"}, "pods":resource.Quantity{i:resource.int64Amount{value:29, scale:0}, d:resource.infDecAmount{Dec:(*inf.Dec)(nil)}, s:"29", Format:"DecimalSI"}}, Allocatable:v1.ResourceList{"attachable-volumes-aws-ebs":resource.Quantity{i:resource.int64Amount{value:25, scale:0}, d:resource.infDecAmount{Dec:(*inf.Dec)(nil)}, s:"25", Format:"DecimalSI"}, "cpu":resource.Quantity{i:resource.int64Amount{value:1930, scale:-3}, d:resource.infDecAmount{Dec:(*inf.Dec)(nil)}, s:"1930m", Format:"DecimalSI"}, "ephemeral-storage":resource.Quantity{i:resource.int64Amount{value:76224326324, scale:0}, d:resource.infDecAmount{Dec:(*inf.Dec)(nil)}, s:"76224326324", Format:"DecimalSI"}, "hugepages-1Gi":resource.Quantity{i:resource.int64Amount{value:0, scale:0}, d:resource.infDecAmount{Dec:(*inf.Dec)(nil)}, s:"0", Format:"DecimalSI"}, "hugepages-2Mi":resource.Quantity{i:resource.int64Amount{value:0, scale:0}, d:resource.infDecAmount{Dec:(*inf.Dec)(nil)}, s:"0", Format:"DecimalSI"}, "memory":resource.Quantity{i:resource.int64Amount{value:7435067392, scale:0}, d:resource.infDecAmount{Dec:(*inf.Dec)(nil)}, s:"", Format:"BinarySI"}, "pods":resource.Quantity{i:resource.int64Amount{value:29, scale:0}, d:resource.infDecAmount{Dec:(*inf.Dec)(nil)}, s:"29", Format:"DecimalSI"}}, Phase:"", Conditions:[]v1.NodeCondition{v1.NodeCondition{Type:"MemoryPressure", Status:"False", LastHeartbeatTime:v1.Time{Time:time.Time{wall:0x0, ext:63745322450, loc:(*time.Location)(0x2bec460)}}, LastTransitionTime:v1.Time{Time:time.Time{wall:0x0, ext:63745067048, loc:(*time.Location)(0x2bec460)}}, Reason:"KubeletHasSufficientMemory", Message:"kubelet has sufficient memory available"}, v1.NodeCondition{Type:"DiskPressure", Status:"False", LastHeartbeatTime:v1.Time{Time:time.Time{wall:0x0, ext:63745322450, loc:(*time.Location)(0x2bec460)}}, LastTransitionTime:v1.Time{Time:time.Time{wall:0x0, ext:63745067048, loc:(*time.Location)(0x2bec460)}}, Reason:"KubeletHasNoDiskPressure", Message:"kubelet has no disk pressure"}, v1.NodeCondition{Type:"PIDPressure", Status:"False", LastHeartbeatTime:v1.Time{Time:time.Time{wall:0x0, ext:63745322450, loc:(*time.Location)(0x2bec460)}}, LastTransitionTime:v1.Time{Time:time.Time{wall:0x0, ext:63745067048, loc:(*time.Location)(0x2bec460)}}, Reason:"KubeletHasSufficientPID", Message:"kubelet has sufficient PID available"}, v1.NodeCondition{Type:"Ready", Status:"True", LastHeartbeatTime:v1.Time{Time:time.Time{wall:0x0, ext:63745322450, loc:(*time.Location)(0x2bec460)}}, LastTransitionTime:v1.Time{Time:time.Time{wall:0x0, ext:63745067070, loc:(*time.Location)(0x2bec460)}}, Reason:"KubeletReady", Message:"kubelet is posting ready status"}}, Addresses:[]v1.NodeAddress{v1.NodeAddress{Type:"InternalIP", Address:"192.168.10.234"}, v1.NodeAddress{Type:"ExternalIP", Address:"3.112.60.73"}, v1.NodeAddress{Type:"Hostname", Address:"ip-192-168-10-234.ap-northeast-1.compute.internal"}, v1.NodeAddress{Type:"InternalDNS", Address:"ip-192-168-10-234.ap-northeast-1.compute.internal"}, v1.NodeAddress{Type:"ExternalDNS", Address:"ec2-3-112-60-73.ap-northeast-1.compute.amazonaws.com"}}, DaemonEndpoints:v1.NodeDaemonEndpoints{KubeletEndpoint:v1.DaemonEndpoint{Port:10250}}, NodeInfo:v1.NodeSystemInfo{MachineID:"ec2a306cc17f479e42954c0ed6d71f8d", SystemUUID:"EC2A306C-C17F-479E-4295-4C0ED6D71F8D", BootID:"677dff8e-7a9e-4fcf-b9a3-e1f0f7e2bfeb", KernelVersion:"4.14.209-160.335.amzn2.x86_64", OSImage:"Amazon Linux 2", ContainerRuntimeVersion:"docker://19.3.6", KubeletVersion:"v1.18.9-eks-d1db3c", KubeProxyVersion:"v1.18.9-eks-d1db3c", OperatingSystem:"linux", Architecture:"amd64"}, Images:[]v1.ContainerImage{v1.ContainerImage{Names:[]string{"docker.elastic.co/elasticsearch/elasticsearch@sha256:3ad224719013a016ab4931d1891fcf873fdf6b8c38d0970e30fc1c8b0c07f436", "docker.elastic.co/elasticsearch/elasticsearch:7.10.0"}, SizeBytes:773720806}, v1.ContainerImage{Names:[]string{"602401143452.dkr.ecr.ap-northeast-1.amazonaws.com/amazon-k8s-cni@sha256:f310c918ee2b4ebced76d2d64a2ec128dde3b364d1b495f0ae73011f489d474d", "602401143452.dkr.ecr.ap-northeast-1.amazonaws.com/amazon-k8s-cni:v1.7.5-eksbuild.1"}, SizeBytes:312076970}, v1.ContainerImage{Names:[]string{"602401143452.dkr.ecr.ap-northeast-1.amazonaws.com/amazon-k8s-cni-init@sha256:d96d712513464de6ce94e422634a25546565418f20d1b28d3bce399d578f3296", "602401143452.dkr.ecr.ap-northeast-1.amazonaws.com/amazon-k8s-cni-init:v1.7.5-eksbuild.1"}, SizeBytes:287782202}, v1.ContainerImage{Names:[]string{"602401143452.dkr.ecr.ap-northeast-1.amazonaws.com/eks/kube-proxy@sha256:71b15b05cdee85a1ff6bdb5bdf9c0788c5089cc72b2f0f41a666c22488670ea4", "602401143452.dkr.ecr.ap-northeast-1.amazonaws.com/eks/kube-proxy:v1.18.8-eksbuild.1"}, SizeBytes:131606723}, v1.ContainerImage{Names:[]string{"602401143452.dkr.ecr.ap-northeast-1.amazonaws.com/eks/coredns@sha256:edffc302f7adb1adc410d45ed2266d83c966bc195995191da63e035111e38afd", "602401143452.dkr.ecr.ap-northeast-1.amazonaws.com/eks/coredns:v1.7.0-eksbuild.1"}, SizeBytes:46269990}, v1.ContainerImage{Names:[]string{"602401143452.dkr.ecr.ap-northeast-1.amazonaws.com/eks/pause@sha256:1cb4ab85a3480446f9243178395e6bee7350f0d71296daeb6a9fdd221e23aea6", "602401143452.dkr.ecr.ap-northeast-1.amazonaws.com/eks/pause:3.1-eksbuild.1"}, SizeBytes:682696}}, VolumesInUse:[]v1.UniqueVolumeName{"kubernetes.io/aws-ebs/aws://ap-northeast-1c/vol-056f0c4c4eda394a9", "kubernetes.io/aws-ebs/aws://ap-northeast-1c/vol-09df3f95e9992955b"}, VolumesAttached:[]v1.AttachedVolume{v1.AttachedVolume{Name:"kubernetes.io/aws-ebs/aws://ap-northeast-1c/vol-09df3f95e9992955b", DevicePath:"/dev/xvdbj"}, v1.AttachedVolume{Name:"kubernetes.io/aws-ebs/aws://ap-northeast-1c/vol-056f0c4c4eda394a9", DevicePath:"/dev/xvdch"}}, Config:(*v1.NodeConfigStatus)(nil)}} 
```

[github.com/davecgh/go-spew/spew](https://github.com/davecgh/go-spew/) give you more human readable format

```go
import (
    "github.com/davecgh/go-spew/spew"
)

func main() {
    spew.Dump(node) 
}
```

サンプル出力例
```
```


### Variables

```go
var (
  i    int     = 1      // can omit int as it will be inferred from the light-hand value
  f64  float64 = 1.2    // can omit float64 as it will be inferred from the light-hand value
  s    string  = "test" // can omit string as it will be inferred from the light-hand value
  t, f bool    = true, false
)

xi := 1
xi = 2
xf64 := 1.2
var xf32 float32 = 1.2
xs := "test"
xt, xf := true, false
fmt.Println(xi, xf64, xs, xt, xf)
// %T show variable type
fmt.Printf("%T\n", xf64)
fmt.Printf("%T\n", xf32)
```


### String
```go
import (
  "fmt"
  "strings"
  "strconv"
)
fmt.Println("Hello " + "World")  // Hello World
fmt.Println(string("Hello World"[0])) //H
s := "Hello World"
s = strings.Replace(s, "H", "Y", 1) // Yello World
fmt.Println(strings.Contains(s, "Hello")) //trude
fmt.Println(`AAA
    xxxx     BBB
    m (_ _)m 
CCC`)
fmt.Println(`"`)
fmt.Println("\"")

// Convert
i := 8
x := fmt.Sprintf("k%ds", i)
fmt.Println(x) // k8s
// strconv.Atoi(string) int, error
i, _ = strconv.Atoi("100")
fmt.Println(i) // 100
// strconv.Itoa(int) string
fmt.Println(strconv.Itoa(100)) // 100
```

### Array and Slice
- Return `_` : declear I don't use this
- `make` and `cap` of slice

```go
// Array
var a[2]int   // You must give the SIZE to array
a[1]=10
a[2]=20
// var a = [2]int{10,20}
// a := [...]int{10, 20}   // elipsis -> Compiler figures out array length
var a[3] = [3]int{1,2,3}
// cannot append as you can NOT change the SIZE of array
// ar = append(ar, 4)

//Slice
//var s []int
//var s = []int {1, 2}
s := []int{1,2}  // This is how you define Slice
s[0]=10
s[1]=20
s = append(s, 3) // You CAN change the size of slice
fmt.Println(s)
// you can add slice in slice
var sb = [][]int{  
		[]int{0, 1, 2},
		[]int{3, 4, 5},
		[]int{6, 7, 8},
}	
// Slice basic
n := []int{1, 2, 3, 4, 5, 6}
fmt.Println(n)
fmt.Println(n[2])   // 2nd
fmt.Println(n[2:4]) // 2nd-> 4thまで、でも4thを含まない -> [3 4]
fmt.Println(n[:2])  // 2ndまで、でも2ndを含まない -> [1,2]
fmt.Println(n[2:])  // 2nd から全部 -> [3,4,5,6]
fmt.Println(n[:])   // all
// Slice make cap
n := make([]int, 3, 5)   //  type, len, capacity
fmt.Printf("len=%d cap=%d value=%v\n", len(n), cap(n), n) // -> len=3 cap=5 value=[0 0 0]
n = append(n, 0, 0) 
fmt.Printf("len=%d cap=%d value=%v\n", len(n), cap(n), n) // -> len=5 cap=5 value=[0 0 0 0 0]
n = append(n, 1, 2, 3, 4, 5)
fmt.Printf("len=%d cap=%d value=%v\n", len(n), cap(n), n) // -> len=10 cap=10 value=[0 0 0 0 0 1 2 3 4 5]
// なんとCapacityが柔軟に5ー>10 にあがった
// メモリ領域を気にするならばCapに収めること。ただしダイナミックアップすることが可能
```
Here is how you iterate arrays and slices
```go
a := [4]int{10, 20, 30, 40}

for i := 0; i < len(a); i++ {
  fmt.Printf("basic loop: index=%d val=%d\n", i, a[i])
}
// loop over an array/a slice
for i, e := range a {
  // i is the index, e the element
  fmt.Printf("index=%d val=%d\n", i, e)
}
// if you only need e:
for _, e := range a {
  // e is the element
  fmt.Printf("val=%d\n", e)
}
// ...and if you only need the index
for i := range a {
  fmt.Printf("index=%d\n", i)
}
// In Go pre-1.4, you'll get a compiler error if you're not using i and e.
// Go 1.4 introduced a variable-free form, so that you can do this
for range time.Tick(time.Second) {
  // do it once a sec
  fmt.Println("do it once a second")
}
```

### Map
```go
// Basic 
// var m map[string]int
m := map[string]int{"apple": 100, "banana": 200}
m["banana"] = 300
fmt.Println(m["banana"])  // -> 300
fmt.Println(m["nothing"]) // -> 0
// check if the value exists
v2, ok2 := m["nothing"]
fmt.Println(v2, ok2) // -> 0, false

// make: メモリ上に空のMapを作成する
m2 := make(map[string]int)
m2["pc"] = 200
fmt.Println(m2["pc"])

/*
	var m3 map[string]int
 	m3["pc"] = 5000
 	fmt.Println(m3)
 	->>>> panic ::: 宣言はしているけどメモリ上にMapがないのでNilなのでPanic発生

 // varで宣言したときはMapもSliceもNilなので注意！！
  var s []int
  if s == nil {
		fmt.Println("Nil")
	}
*/
// iterate over map
for k, v := range m {
  fmt.Printf("key=%s val=%d\n", k, v)
}

type Person struct {
  Name string
  Age  int
}
m3 := map[string]Person{
  "Taro": {"Go Taro", 35},
	"Jiro": {"Go Jiro", 30},
}
for k, v := range m3 {
  fmt.Printf("key=%s Person: name=%s age=%d\n", k, v.Name, v.Age)
}
```

### Function

```go
// You can return multiple values
func testfunc(x, y int) (int, int) {
	return x + y, x - y
}

// Named return value
func testfunc(a, b int) (result int) {
	result = a * b
	//return result  ## This is also OK
	return //  Naked Returns
}

// Inner func
f := testfunc(x int) {
  fmt.Println("Inner func", x)
}
f(1)

// You can directry execute the func instead of assigning it to f
func(x int) {
  fmt.Println("Inner func", x)
}(1)

// Variadic parameters
func varparamfunc(params ...int) {
	for _, p := range params {
		fmt.Println(p)
	}
}
varparamfunc(1, 2)
varparamfunc(1, 2, 3)
params := []int{1, 2, 3, 4}
varparamfunc(params...)
```

### Closure
```go
// Func as return value
func incrCounter() func() int {
	x := 0
	return func() int {
		fmt.Println("inner func: Increment")
		x++
		return x
	}
}
counter := incrCounter()
fmt.Println(counter())
fmt.Println(counter())
fmt.Println(counter())
```

### Struct

```go
// struct atrribute:
// CAPTIAL     -> Accessible from outside
// NOT CAPITAL -> NOT accessible from outside 
type Vertex struct {
  X, Y int
  Z string
}

// There is NO class in Golang but use Struct for OOP like
// Value vs Pointer?
//  - Value   -> When you refer to the value
//  - Pointer -> When you change the value

// Value receiver
func (v Vertex) UseValue() string {
  return fmt.Sprintf("X=%d Y=%d Z=%s", v.X, v.Y, v.Z)
}
// Pointer receiver
func (v *Vertex) ChangeValue(i int) {
  v.X = v.X * i
  v.Y = v.Y * i
  v.Z = fmt.Sprintf("%s_%d", v.Z, i)
}

func main(){
  v1 := Vertex{X: 1, Y: 2, Z:"hoge"}
  v2 := Vertex{X: 1}
  fmt.Println(v2)               // -> {1 0 }
  v3 := Vertex{1, 2, "hoge"}
  fmt.Println(v3)               // -> {1 2 hoge}
  v4 := Vertex{}
  var v5 Vertex
  fmt.Printf("%T %v\n", v5, v5) // -> main.Vertex {0 0 }
  v6 := new(Vertex)
  fmt.Printf("%T %v\n", v6, v6) // -> *main.Vertex &{0 0 }
  v7 := &Vertex{1, 2, "hoge"}
  fmt.Printf("%T %v\n", v7, v7)
  // [Key Point]
  // It's very common to use pointer for struct in dealing struct in your app
  // while it's recommended to use make for Slice / Map 
  v := Vertex{100, 200, "foo"}
  fmt.Println(v.UseValue()) // X=100 Y=200 Z=foo
  v.ChangeValue(100)
  fmt.Println(v.UseValue()) // X=10000 Y=20000 Z=foo_10
}
```

### Pointer
```go
func set_ten(x *int) {
  *x = 10
}
var n = 100
set_ten(&n)
fmt.Println(n)  // -> 10
fmt.Println(&n) // -> address
var p *int = &n
fmt.Println(p)   // -> address
fmt.Println(*p)  // -> 10
fmt.Println(&*p) // -> address
```

### Pointer receiver and Value receiver
Basically use pointer receiver, but use value receiver in case :
- you don't need to change the value of receiver
- struct is based on primitive data type as copy cost for primitive type is very low

```go
type Human struct{ Name string }

// Value receiver
func (h Human) Say(something string) {
	fmt.Printf("%s said %s.\n", h.Name, something)
}

// Value receiver
func (h Human) Change(name string) {
	h.Name = name
}

// Pointer receiver
func (h *Human) SayP(something string) {
	fmt.Printf("%s said %s.\n", h.Name, something)
}

// Pointer receiver
func (h *Human) ChangeP(name string) {
	h.Name = name
}

hp := &Human{Name: "Yoichi"}
hp.SayP("Hello")
hp.Say("Hello")        //コンパイラによるレシーバの暗黙的変換
hp.ChangeP("Kawasaki") //レシーバの値を変更
hp.SayP("Hello")

h := &Human{Name: "Yoichi"}
h.SayP("Hello")
h.Say("Hello")       //コンパイラによるレシーバの暗黙的変換
h.Change("Kawasaki") //レシーバの値が変更されない
h.Say("Hello")
```
See also [this](https://skatsuta.github.io/2015/12/29/value-receiver-pointer-receiver/), good summary!

### Interface

 Interfaceで宣言したメソッドを必ず実装してほしいときにInterfaceを使う
```go
// Interface definition
type Car interface {
	Run() string
}
type SuperCar struct {
	Name string
}
// Implimentation of method
func (s *SuperCar) Run() string {
	return s.Name + " is running very fast"
}

var car1 Car = &SuperCar{"Ferrari"}
fmt.Println(car1.Run())
```
Empty interfaceについて。 Empty Interfaceにはどんな型でも代入することが可能
```go
type Hoge interface{} //  空インターフェース定義
var hoge Hoge
// var hoge interface{} // 空インターフェース定義SHORTCUT
// You can give any value
hoge = 0
hoge = "hey"

//"value, ok = element.(T)"  => valueは変数の値が格納、okにはbool型
if value, ok := hoge.(int); ok {
  fmt.Printf("int value:%d", value)
} else if value, ok := hoge.(string); ok {
  fmt.Printf("string value:%s", value)
} else {
  fmt.Println("other")
}
```

### Goroutine
```go
/****************************** 
Goroutine keypoints
var wg sync.WaitGroup
go gorouine(xxx, &wg)  //goroutine(xx, wg *sync.WaitGroup)
wg.Add(N)              // incr N thread operation
wg.Done() x N          // Done when thread done (if it's in a scope call wg.Done() with defer)
wg.Wait()              // end of goroutine (goroutine wait until wg.done(). Without this, main func go end before goroutine)
******************************/

import ( "sync")

func goroutine(s string, wg *sync.WaitGroup) {
	defer wg.Done()
	// do something
	//	wg.Done()
}
func main() {
	var wg sync.WaitGroup
	wg.Add(1) // tell that I have 1 thread operation -> Wait 1 wg.Done() 
	go goroutine("world", &wg)
	wg.Wait() // Wait wg.Done()
}
```

### Channel
`Channels` are the `pipes` that connect concurrent `goroutines`. By default, channlels are `unbuffered`: accept 1 send(c<-) if a receive(<-c)

```go
 messages := make(chan string)
 go func() { messages <- "ping" }()
 msg := <-messages
```
`Buffered Channels` allow you to receive N send(c<-) without N receive(<-c)

```go
msg := make(chan string, 2) //buffered channel
msg <- "buf buf"   //send
msg <- "chan chan" //send
fmt.Println(<-msg) //receive
fmt.Println(<-msg) //receive

// Close Channel so no longer channel send & receive msg
ch := make(chan int, 2)
ch <- 100
ch <- 200
fmt.Println(len(ch))
close(ch)

// Without close channle, you'll get error in looping with Range as Range try to get 3rd one
for c := range ch {
	fmt.Println(c)
}
```

`Channel Syncronization` -  synchronize execution (block recieve to wait for a goroutine to finish) across goroutines

```go
func goroutine(done chan bool) {
    // Do something
    // Send a value to notify that we're done.
    done <- true
}

func main() {
    // Start a worker goroutine, giving it the channel to notify on.
    done := make(chan bool, 1)
    go goroutine(done)
    // Block until receive on the channel.
    <-done
}
```
Use `select` when you don't want to block receive (Channel block receive by default)

```go
select {
case v := <- ch:
    fmt.Println(v)
default:
    fmt.Println("no value")
}

// if channle is aloready closed, channle not block and so you'll get value in "case v := <-ch"
// you can identify if you can receive or channle closeed, do like this
select {
case v, ok := <- ch:
    if ok {
        fmt.Println(v)
    } else {
        fmt.Println("closed")
    }
default:
    fmt.Println("no value")
}
```


## golang with VSCode
### Install/Update Tools
Run `Go: Install/Update Tools` in Command Palette to update all tools

### Go config in settings.json
```json
"go.gopath": "/Users/yoichi.kawasaki/dev/go",
"go.autocompleteUnimportedPackages": true,
// ツール類を最新にするだけではパッケージが見つからない
"go.toolsEnvVars": {"GO111MODULE": "on"},
// go.formatToolのデフォルト設定goreturnsはModulesに対応していない様子
"go.formatTool": "goimports"

"[go]": {
  // Save時にGoフォーマットするかどうか
  "editor.formatOnSave": true
}
```

- https://qiita.com/msmsny/items/a8d4573d03774815a198

## LINKS
- Standard Go Project Layout
  - https://github.com/golang-standards/project-layout
- Golang cheat sheet
  - https://github.com/a8m/golang-cheat-sheet
