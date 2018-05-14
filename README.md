# 比特股私有链环境搭建

## 编译安装
### Ubuntu 16.04 LTS 下的准备

> 建议使用Ubuntu 16.04进行

``` 
apt-get update

apt-get install libboost-all-dev cmake make libbz2-dev libdb++-dev libdb-dev libssl-dev openssl libreadline-dev autoconf libtool git ntp libcurl4-openssl-dev g++ libcurl4-openssl-dev

```

### 编译
```
mkdir /etc/insur

cat >> /etc/insur/build.sh << EOF
git clone https://github.com/bitshares/bitshares-core.git
cd bitshares-core
git submodule update --init --recursive
cmake -DBOOST_ROOT="$BOOST_ROOT" -
DCMAKE_BUILD_TYPE=Release .
make 
ln -s /etc/insur/bitshares-core/programs/witness_node/
witness_node  /usr/local/bin/witness_node
ln -s /etc/insur/bitshares-core/programs/cli_wallet/cli_wallet /usr/local/bin/cli_wallet
EOF

cd /etc/insur
nohup bash build.sh &
```

## 搭建私有网络

+ 创建一个名为`genesis.json`的新创世文件

> mkdir /etc/insur/run_test
> cd /etc/insur/run_test
> witness_node --create-genesis-json=genesis.json


修改这个文件可以更改网络的初始状态，它可以控制如下
1. 创世账户及公钥 
2. 资产及其初始分配 
3. 链参数的初始值
4. 初始证人的帐户签名密钥

+  执行如下命令生成`config.ini`文件

> witness_node --data-dir=data --genesis-json=genesis.json

会看到当前节点会默认选取公网默认节点进行同步，此时执行ctrl+c中断该操作。查看目录，已经生成数据目录`data`, 该目录包含数据库、配置文件等。

修改config.ini文件如下内容：
```
p2p-endpoint = 0.0.0.0:8090

seed-nodes = []

rpc-endpoint = 0.0.0.0:11011

genesis-json = genesis.json

enable-stale-production = true

required-participation = false

witness-id = "1.6.1"

private-key = ["BTS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV","5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3"]

bucket-size = [15,60,300,3600,86400]

history-per-size = 1000

[log.console_appender.stderr]
stream=std_error

[log.file_appender.p2p]
filename=logs/p2p/p2p.log

[logger.default]
level=info
appenders=stderr

[logger.p2p]
level=info
appenders=p2p
```

修改完config文件后，重新启动节点命令

> nohup witness_node --data-dir=data --genesis-json=genesis.json &

会看到如下显示
```
2081032ms th_a       witness.cpp:88                plugin_initialize    ] witness plugin:  plugin_initialize() begin
2081032ms th_a       witness.cpp:99                plugin_initialize    ] Public Key: BTS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
2081032ms th_a       witness.cpp:117               plugin_initialize    ] witness plugin:  plugin_initialize() end
2081032ms th_a       application.cpp:441           startup              ] Replaying blockchain due to: unclean shutdown detected
2081032ms th_a       application.cpp:330           operator()           ] Initializing database...
2081033ms th_a       db_management.cpp:51          reindex              ] reindexing blockchain
2081034ms th_a       db_management.cpp:104         wipe                 ] Wiping database
2081034ms th_a       db_management.cpp:170         close                ] Database close unexpected exception: {"code":10,"name":"assert_exception","message":"Assert Exception","stack":[{"context":{"level":"error","file":"index.hpp","line":111,"method":"get","hostname":"","thread_name":"th_a","timestamp":"2018-05-14T08:34:41"},"format":"maybe_found != nullptr: Unable to find Object","data":{"id":"2.1.0"}}]}
2081035ms th_a       object_database.cpp:87        wipe                 ] Wiping object database...
2081035ms th_a       object_database.cpp:89        wipe                 ] Done wiping object databse.
2081035ms th_a       object_database.cpp:94        open                 ] Opening object database from /etc/insur/run_test/data/blockchain ...
2081035ms th_a       object_database.cpp:100       open                 ] Done opening object database.
2081038ms th_a       db_debug.cpp:85               debug_dump           ] total_balances[asset_id_type()].value: 0 core_asset_data.current_supply.value: 1000000000000000
2081038ms th_a       db_management.cpp:58          reindex              ] !no last block
2081038ms th_a       db_management.cpp:59          reindex              ] last_block:
2081040ms th_a       application.cpp:204           reset_p2p_node       ] Configured p2p node to listen on 0.0.0.0:8090
2081041ms th_a       application.cpp:279           reset_websocket_serv ] Configured websocket rpc to listen on 0.0.0.0:11011
2081041ms th_a       witness.cpp:122               plugin_startup       ] witness plugin:  plugin_startup() begin
2081041ms th_a       witness.cpp:127               plugin_startup       ] Launching block production for 1 witnesses.

********************************
*                              *
*   ------- NEW CHAIN ------   *
*   - Welcome to Graphene! -   *
*   ------------------------   *
*                              *
********************************

Your genesis seems to have an old timestamp
Please consider using the --genesis-timestamp option to give your genesis a recent timestamp

2081041ms th_a       witness.cpp:138               plugin_startup       ] witness plugin:  plugin_startup() end
2081041ms th_a       main.cpp:179                  main                 ] Started witness node on a chain with 0 blocks.
2081041ms th_a       main.cpp:180                  main                 ] Chain ID is fd7e8d9e32ccbc7aea83d7786c9700e84b9fd97e8d01607c03fc15564823a107
2090000ms th_a       witness.cpp:184               block_production_loo ] Generated block #1 with timestamp 2018-05-14T08:34:50 at time 2018-05-14T08:34:50
2160000ms th_a       witness.cpp:184               block_production_loo ] Generated block #2 with timestamp 2018-05-14T08:36:00 at time 2018-05-14T08:36:00
2215000ms th_a       witness.cpp:184               block_production_loo ] Generated block #3 with timestamp 2018-05-14T08:36:55 at time 2018-05-14T08:36:55
```

> 开始产生块，说明节点创建成功。创世节点中 `seed-nodes = []` 为空，在后续创建其他节点时需要在此填入私有节点的`p2p-endpoint`地址。例如：
> seed-nodes = ["172.17.180.109:8090"]

## 客户端cli_wallet的使用

### 客户端连接
```
cd /etc/insur/run_test

cli_wallet --wallet-file=my-wallet.json --chain-id=fd7e8d9e32ccbc7aea83d7786c9700e84b9fd97e8d01607c03fc15564823a107 --server-rpc-endpoint=ws://127.0.0.1:8090
```

> chain-id 需要替换为上文中节点启动时日志中显示的chain-id, 或者不添加此参数启动，可以从报错信息中看到该提示

### 创建新钱包 
在上述步骤的命令行模式继续执行如下命令：
```
>>> set_password password
>>> unlock password 
```

### 获取创始利益 
```
unlock >>> import_key nathan 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```
> nathan恰好是在创建文件中定义的帐户名称。如果您在创建genesies.json文件后编辑了自己的文件，可以在其中添加一个不同的名称。
另请注意，5KQwrPbwdL ... P79zkvFD3是在config.ini文件中定义的私钥。

```
>>> get_account nathan
>>> list_account_balances nathan
```


 


