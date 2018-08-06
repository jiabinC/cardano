# 如何从源码构建连到cardano主网的可执行文件 connect-to-mainnet

* cardano 源码github下载 [cardano-sl]( https://github.com/input-output-hk/cardano-sl.git)

  ```
  git clone https://github.com/input-output-hk/cardano-sl.git
  cd cardano-sl
  ```
* nix 安装   
  ```
  sudo apt-get install curl       //安装curl
  curl https://nixos.org/nix/install | sh    //使用curl安装nix
  ```
* 之后使用签名的IOHK二进制缓存,创建nix的配置文件
  ```
  sudo mkdir -p /etc/nix
  sudo gedit /etc/nix/nix.conf     //可以使用vi,vim 等编辑工具
  ```
* 向其中添加下面两行
  ```
  binary-caches            = https://cache.nixos.org https://hydra.iohk.io
  binary-cache-public-keys = hydra.iohk.io:f/Ea+s+dFdN+3Y/G+FDgSq+a5NEWhJGzdjvKNGv0/EQ=
  ```
* 使用`git checkout master`切换到master分支，然后构建可执行文件
  ```
  nix-build -A connectScripts.mainnetWallet -o connect-to-mainnet
  ```
* 最后在cardano-sl目录下，会有可执行文件connect-to-mainnet，通过`./connect-to-mainnet`连接到主网





## 相关资料：
* 在ubuntu下完整构建过程 ： [https://steemit.com/utopian-io/@sebastiengllmt/building-daedalus-cardano-sl-on-ubuntu](https://steemit.com/utopian-io/@sebastiengllmt/building-daedalus-cardano-sl-on-ubuntu)
* 更多详细的文档 ：[https://github.com/input-output-hk/cardano-sl/tree/master/docs/how-to](https://github.com/input-output-hk/cardano-sl/tree/master/docs/how-to)
