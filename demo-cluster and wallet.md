## 如何部署本地的节点测试网络，使钱包连接到本地网络，并进行转账测试

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
* 





* 之前通过`nix-build`通过构建default.nix表达式，构建出来的位于/master/bin的可执行文件cardano-sl-simple，配置相关的参数可以发起一个普通的节点，具体的参数配置说明如下：

  ```
  --db-path ./state-demo/core-db$i                  //指定节点数据库的生成路径
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
  其中网络拓扑配置文件内容如下：

  
