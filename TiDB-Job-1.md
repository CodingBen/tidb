[toc]
## TiDB High Performance第一期作业


1. [x] 编译部署TiDB
2. [x] 编译部署PD
3. [x] 编译部署TiKV
4. [x] 改编源码，在TiDB事务启动时候输出日志**hello transaction**



### 搭建TiDB,PD对应需要的go环境

> 本机环境是 Mac OS 10.14.6
TiDB，TiKV，PD均是从Pingcap Github仓库fork并使用Master分支编译



因为TiDB是go项目，并且采用Go Modules来管理项目的依赖。我们先到go mod下查看其中go语言的版本号。

![go-version](https://ben-space-1252588607.cos.ap-shanghai.myqcloud.com/img/tidb-go-version.png)

然后我们便安装这个version的go就好了

```shell
brew install go@1.13
```


### 编译TiDB


因为Tidb使用了MakeFile,所以只需要在项目文件夹下make可以开始执行.

```shell
make
```

当编译完成后可以在TiDB项目的bin文件夹下看到编译后的文件

![tidb-compile](https://ben-space-1252588607.cos.ap-shanghai.myqcloud.com/img/tidb%20compile.png)


### 编译PD

编译PD的过程和之前，一样。但是PD包含一个DashBorad，需要编译过程中下载。
我在编译的时候就遇到了这个下载资源很慢的情况，于是我便直接下载了这个资源文件放在对应的cache目录下，跳过了这一步。

直接通过这个URL下载

![pd-dashborad-url](https://ben-space-1252588607.cos.ap-shanghai.myqcloud.com/img/pd-dashborad-url.png)

放到对应的目录下

![pd-dashborad](https://ben-space-1252588607.cos.ap-shanghai.myqcloud.com/img/pd-dashborad.png)

很快看到编译好的文件就出来了

![pd-server](https://ben-space-1252588607.cos.ap-shanghai.myqcloud.com/img/pd-server.png)

### 编译TiKV
Tikv和TiDB和PD不一样。TiKV是采用Rust编写的，采用cargo 进行项目管理，编译的时候如果在本地可以直接cargo编译。

```shell
cargo build --release
```

然后可以在target目录下看到我们的编译结果
![](https://ben-space-1252588607.cos.ap-shanghai.myqcloud.com/img/tikv-compile.png)


### 通过Tiup 启动自己编译的集群

下载并安装Tiup

```shell
curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
```

将刚才编译后TiDB，TiKV，PD的绝对路径复制到对应的binpath参数后
```shell
tiup playground \
 --db 1 --db.binpath /Users/zhuqi/go/src/github.com/pingcap/tidb/bin/tidb-server \
 --kv 1 --kv.binpath /Users/zhuqi/rust/tikv/target/release/tikv-server \
 --pd 1 --pd.binpath /Users/zhuqi/go/src/github.com/pingcap/pd/bin/pd-server \
 --monitor
```

### 修改源码，在Tidb打开事务的时候输出日志 hello transaction

修改的位置
session/session.go:1494

![hello_transaction_code](https://ben-space-1252588607.cos.ap-shanghai.myqcloud.com/img/hello_transaction_code.png)


登陆TiDB
```sql
mysql -u root  -h 127.0.0.1 -P 4000
```


建个表吧，也能打开事务
```sql
show datbases;

use test;

# classic student table
CREATE TABLE IF NOT EXISTS `student`(
   `sid` INT UNSIGNED AUTO_INCREMENT,
   `name` VARCHAR(100) NOT NULL DEFAULT '',
   `gender` VARCHAR(40) NOT NULL DEFAULT '',
   PRIMARY KEY ( `sid` )
);
```

监听日志看到对应日志输出

![hello transaction](https://ben-space-1252588607.cos.ap-shanghai.myqcloud.com/img/hello-tranaction-log.png)

