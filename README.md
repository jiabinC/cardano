## 如何从源码构建连到cardano主网的可执行文件 connect-to-mainnet

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
* 构造可执行文件 cardano-node-master
  ```
  nix-build -A cardano-sl-static --cores 0 --max-jobs 2 --no-build-output --out-link master
  ```
* 使用`git checkout master`切换到master分支，然后构建可执行文件
  ```
  nix-build -A connectScripts.mainnetWallet -o connect-to-mainnet
  ```
* 最后在cardano-sl目录下，会有可执行文件connect-to-mainnet，通过`./connect-to-mainnet`连接到主网,如果运行出现yaml exception，说明配置文件路径不对，进入编辑connetc-to-mainnet可执行文件，按照下图修改配置文件路径即可

  ![](https://github.com/jiabinC/cardano/blob/master/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180807102135.png)
  
  
## 如何构建daedalus钱包

  1. 下载[daedalus](https://github.com/input-output-hk/daedalus.git)源码
     ```
     $ git clone https://github.com/input-output-hk/daedalus.git
     $ cd daedalus
     ```

  2. 首先安装nodejs 与 npm
     ```
     $ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash  //安装npm的版本管理器 
     $ sudo nvm install x.y.z                         // x,y,z 为node版本号，此命令来安装node与npm
     $ node -v                                        //测试node是否安装成功
     $ npm -v                                         //测试npm 是否安装成功
     ```
  3. daedalus 目录下有package.json，它是一个模块的配置文件，在项目目录下使用下面的命令来安装模块，安装的模块将位于node_modules目录下：
     ```
     $ npm install
     ```
  4. 在connect-to-mainnet脚本运行的前提下，通过下面的命令来运行钱包应用
     ```
     $ npm run dev
     ```
     上面的命令其实执行的是package.json里面设置的一些命令（scripts），在package.json中script被定义为dev，可通过上述命令运行
     
     ![](https://github.com/jiabinC/cardano/blob/master/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20180807104410.png)
     
  
  





## 相关资料：
* 在ubuntu下完整构建过程 ： [https://steemit.com/utopian-io/@sebastiengllmt/building-daedalus-cardano-sl-on-ubuntu](https://steemit.com/utopian-io/@sebastiengllmt/building-daedalus-cardano-sl-on-ubuntu)
* 更多详细的文档 ：[https://github.com/input-output-hk/cardano-sl/tree/master/docs/how-to](https://github.com/input-output-hk/cardano-sl/tree/master/docs/how-to)
