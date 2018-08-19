# 如何部署本地的节点测试网络，使钱包连接到本地网络，并进行转账测试

### cardano网络可以分为三类节点，核心节点、中继节点、钱包节点（客户端节点）

* 首先一个节点网络需要一个启动时间，定义启动时间为 `system_start=$((`date +%s` + 15))`，并创建demo-cluster的目录
  ```
  # Remove previous state
  rm -rf ./state-demo
  mkdir -p ./state-demo/logs
  ```

* 利用cardano-sl提供的静态工具`cardano-keygen`,这个命令将生成创始区块的secrets并转存到目录中
  ```
  cardano-keygen --system-start 0 generate-keys-by-spec 
  --genesis-out-dir ./state-demo/genesis-keys 
  --configuration-file    $config_files/configuration.yaml 
  --configuration-key default
  ```
  
 ### 核心节点与中继节点的启动

* `cardano`提供了cordano-node可执行文件，cordano-node-simple可执行文件，后者用于启动一个wallet node





* 之前通过`nix-build`通过构建default.nix表达式，构建出来的位于/master/bin的可执行文件cardano-sl-simple，配置相关的参数可以发起一个core节点，具体的参数配置说明如下：

  ```
  --db-path ./state-demo/core-db$i                  //指定节点数据库的生成路径
  --genesis-secret $i                               //生成创世块secret数据
  --rebuild-db --genesis-secret $i 
  --listen 127.0.0.1:$((3000 + i))                  //节点启动的IP和port
  --json-log ./state-demo/logs/core$i.json          //json格式的日志文件
  --logs-prefix ./state-demo/logs 
  --system-start $system_start                      //将系统启动时间指定为unix时间戳，以秒为单位。 如果配置中没有它，则必须提供，否则不得提供。
  --metrics +RTS -N2 -qg -A1m -I0 -T -RTS 
  --node-id core$i 
  --topology /nix/store/bipfcjmkq0pp90fd9dng4ayygq92c19k-topology.yaml   //配置cardano-sl的网络拓扑配置文件
  --configuration-file $config_files/configuration.yaml     //配置文件的路径，默认为项目目录下的 /lib/configuration.yaml
  --configuration-key default                               //specifies key in this configuration. Default value is default.
  ```
  * 其中网络拓扑配置文件内容如下,格式为json文件，定义了网络的4个core节点和1个relay节点，确定了网络节点的关系。
  ```
    {"nodes":
    {"core1":{"addr":"127.0.0.1","port":3001,"region":"undefined","static-routes":[["core2"],["core3"],["core4"],                                ["relay1"]],"type":"core"},
     "core2":{"addr":"127.0.0.1","port":3002,"region":"undefined","static-routes":[["core1"],["core3"],["core4"],                                ["relay1"]] ,"type":"core"},
     "core3":{"addr":"127.0.0.1","port":3003,"region":"undefined","static-routes":[["core1"],["core2"],["core4"],                                ["relay1"]],"type":"core"},
     "core4":{"addr":"127.0.0.1","port":3004,"region":"undefined","static-routes":[["core1"],["core2"],["core3"],                                ["relay1"]],"type":"core"},
     "relay1":{"addr":"127.0.0.1","port":3101,"region":"undefined","static-routes":[["core1"],["core2"],["core3"],                                ["core4"]],"type":"relay"}}}
  ```
  * relay节点与core节点启动用到的可执行文件是相同的，都为cardano-node-simple，用到的配置文件也可以相同，缺少` --genesis-secret $i  `参数
  
  ### wallet node的启动，并连接到上述网络,使用的是可执行文件cardano-node
    ```
      exec /nix/store/hwhrw5q3gx2a132az89gq17yypzrdznw-cardano-sl-wallet-new-static-1.3.0/bin/cardano-node                                     \
      --configuration-file /nix/store/ygdlx8j6jbpf0cp7v6y1a0fb8rnqjqm9-cardano-sl-config/lib/configuration.yaml --configuration-key dev                                           \
     --tlscert ./state-demo/tls/server/server.crt     \                       //wallet node 的tls公钥证书
     --tlskey ./state-demo/tls/server/server.key      \                       //wallet node 的私钥
     --tlsca ./state-demo/tls/server/ca.crt           \                       //ca的公钥证书
     --log-config /nix/store/ygdlx8j6jbpf0cp7v6y1a0fb8rnqjqm9-cardano-sl-config/log-configs/connect-to-cluster.yaml \
     --topology "/nix/store/n7wkmk8y11n9q40903a1kj7xqr4rsd74-wallet-topology.yaml" \     //此配置文件定义wallet node连接到哪个 relay node
     --logs-prefix "./state-demo/logs"                               \
     --db-path "./state-demo/db"                       \
     --wallet-db-path './state-demo/wallet-db'        \
                                       \
     --no-client-auth                     \
     --keyfile ./state-demo/secret.key                               \
     --wallet-address localhost:8090               \
     --wallet-doc-address localhost:8091        \
     --ekg-server localhost:8000 --metrics                             \
      +RTS -N2 -qg -A1m -I0 -T -RTS   
    ```

 ### import HD keys/wallet 
  
   * 在core节点启动时，生成了genesis-secret，里面初始化了一些钱包的地址及余额等信息，位于genesis-keys目录中，我们可以通过curl http请求将其导入wallet node中

      ```
       curl https://localhost:8090/api/wallets/keys \ 
        --cacert ./state-demo/tls/client/ca.crt \                         //提供ca的公钥证书
        --cert ./state-demo/tls/client/client.pem \                       //tls双向验证，提供客户端的证书和私钥
        -X POST \                                                         // http method 为POST
        -H 'cache-control: no-cache' \                    
        -H 'content-type: application/json' \         
        -d "\"./state-demo/genesis-keys/generated-keys/poor/key$i.sk\"" | jq .       // 指定创始区块的.sk文件
      ```
 
### 启动wallet ui客户端
     
  * 在daedalus项目根目录下，在安装了yarn和npm、npm的基础上，运行yarn install，进行客户端相关js模块的安装
     
  * 进行tls配置
     
      ```
       export CARDANO_TLS_PATH="path-tls-file"   //指定tls的路径，这里为path-to-cardano/state-demo/tls/server,指定连接到的wallet的地址
       export NETWORK="   "                      //可为mainnet，testnet，默认为dev，这里为默认
      ```
  * 启动钱包客户端
     
     ```
       yarn run dev(start)
     ```
    
  
