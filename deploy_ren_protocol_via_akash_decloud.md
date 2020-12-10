### 通过Akash DeCloud部署Ren Protocol

**Ren Protocol**

>“暗池交易的主要优势在于，进行大宗交易的机构投资者可以在寻找买家和卖家时不会曝光。可以防止影响到整个市场，避免贬值。
>
>Republic Protocol系统依靠精心设计的激励系统来保证玩家遵守规则。RenEx“Dark Nodes”运行撮合引擎来保证暗池的运行，同时多个暗池的存在将减轻交易对手的风险。为了防止恶意攻击，规定网络节点（Darknodes）必须支付100000个Ren，这个也就是Republic Protocol的Token。

**Akash DeCloud**

> 根据官网介绍：Akash正在建立一个虚拟市场，让个人和组织实现无摩擦的云算力买卖的Airbnb市场：Airbnb平台能让用户出租闲置的房子，Akash平台则让用户出租闲置的算力。双方的直接接触大大拓展了已有的市场，让闲置的资源得以利用——出租方获得收益，租户获得更简便的体验。
>
> 目前世界上有大约840万个数据中心，其中85%服务器容量未被充分利用，Akash的去中心化云计算市场将如何彻底改变和重新定义云市场。
>
> akash肯定有自己的优势：
>
> - **去中心化：**Akash提供可信的执行环境，网络启动后，即使团队停止开发，用户仍然能正常使用Askah。
> - **无需许可：**只需要按照标准接入市场，人人可自由出借算力获取收益，人人可按需借用算力。
> - **成本更低：**Akash用户直接出价，供应商竞价获取该任务。买卖直接受供需关系影响，能提供更合理的价格。根据测试数据，**Akash的云计算成本比现有市场提供商（AWS、谷歌云、微软Azure和阿里云）低10倍。**
>
> 简而言之就是Akash是一个去中心化的云服务商，可以帮助我们以低成本像在AWS上部署应用一样部署一个去中心化应用。

整个教程大概分为编译构建源码、创建上传docker镜像、akash部署环境准备、部署Ren Protocol几部分。

#### 编译构建源码

1. 我们从[Ren Protocol](https://github.com/renproject/bridge)拉取代码

   ```
   git clone https://github.com/renproject/bridge.git
   ```

   输出如下

   ```
   ➜  github git clone https://github.com.cnpmjs.org/renproject/bridge.git
   Cloning into 'bridge'...
   remote: Enumerating objects: 40, done.
   remote: Counting objects: 100% (40/40), done.
   remote: Compressing objects: 100% (26/26), done.
   remote: Total 1354 (delta 16), reused 26 (delta 13), pack-reused 1314
   Receiving objects: 100% (1354/1354), 85.64 MiB | 10.25 MiB/s, done.
   Resolving deltas: 100% (886/886), done.
   ```

2. 注册自己的[firebase](https://firebase.google.com/)账号并获取Key之后设置变量

   ```
   export REACT_APP_FB_KEY="YOUR_KEY_HERE"
   ```

3. 安装依赖

   ```
   cd bridge && yarn install 
   ```

4. 运行

   ```
   yarn start
   ```

   之后访问 `http://localhost:3000/`

5. 打包

   ```
   yarn build
   ```

   输出如下

   ```
   ➜  bridge git:(master) ✗ yarn build
   yarn run v1.22.10
   $ CI=false REACT_APP_VERSION=$npm_package_version react-scripts --max_old_space_size=4096 build
   Creating an optimized production build...
   Compiled with warnings.
   
   ./node_modules/web3-eth-accounts/dist/web3-eth-accounts.umd.js
   Module not found: Can't resolve 'scrypt' in '/Users/siru-6/work/akash/github/bridge/node_modules/web3-eth-accounts/dist'
   
   Search for the keywords to learn more about each warning.
   To ignore, add // eslint-disable-next-line to the line before.
   
   File sizes after gzip:
   
     2.05 MB   build/static/js/2.f68016a1.chunk.js
     25.52 KB  build/static/js/main.f611ed64.chunk.js
     774 B     build/static/js/runtime-main.6f7ddd06.js
     604 B     build/static/css/main.08499955.chunk.css
   
   The bundle size is significantly larger than recommended.
   Consider reducing it with code splitting: https://goo.gl/9VhYWB
   You can also analyze the project dependencies: https://goo.gl/LeUzfb
   
   The project was built assuming it is hosted at /.
   You can control this with the homepage field in your package.json.
   
   The build folder is ready to be deployed.
   You may serve it with a static server:
   
     yarn global add serve
     serve -s build
   
   Find out more about deployment here:
   
     bit.ly/CRA-deploy
   
   ✨  Done in 122.29s.
   ```

   

#### 创建docker镜像

1. 创建Dockerfile文件

   ```
   vi Dockerfile
   ```

   填入如下内容

   ```
   FROM nginx
   ENV REACT_APP_FB_KEY="YOUR_KEY_HERE"
   COPY build /usr/share/nginx/html
   EXPOSE 80
   CMD ["nginx", "-g", "daemon off;"]
   ```

2. 生成docker镜像

   ```
   docker build . -t fishflyhuang/renprotocol:latest
   ```

   输出如下

   ```
   ➜  bridge git:(master) ✗ docker build . -t fishflyhuang/renprotocol:latest
   Sending build context to Docker daemon  846.2MB
   Step 1/5 : FROM nginx
    ---> bc9a0695f571
   Step 2/5 : ENV REACT_APP_FB_KEY="AIzaSyCe46fj_bL9IjA_S9lDUrHYig-pOvJU0mY"
    ---> Running in 03f2e052fe01
   Removing intermediate container 03f2e052fe01
    ---> c79a8ea1c864
   Step 3/5 : COPY build /usr/share/nginx/html
    ---> 3f58d8002d2a
   Step 4/5 : EXPOSE 80
    ---> Running in 1d41daacd507
   Removing intermediate container 1d41daacd507
    ---> 8e961fdd6e34
   Step 5/5 : CMD ["nginx", "-g", "daemon off;"]
    ---> Running in 0b42afe3b017
   Removing intermediate container 0b42afe3b017
    ---> 8cb3b59e5b90
   Successfully built 8cb3b59e5b90
   Successfully tagged fishflyhuang/renprotocol:latest
   ```

3. 查看docker镜像

   ```
   docker images
   ```

4. 运行容器

   在3260端口后台运行

   ```
   docker run -d --name renprotocol -p 3260:80 --restart=unless-stopped fishflyhuang/renprotocol:latest
   ```

   通过 `http://localhost:3260/` 访问

5. 上传镜像

   ```
   docker push fishflyhuang/renprotocol:latest
   ```

   输出如下

   ```
   ➜  bridge git:(master) ✗ docker push fishflyhuang/renprotocol:latest
   The push refers to repository [docker.io/fishflyhuang/renprotocol]
   3a4c7ae0fc19: Pushed
   7e914612e366: Mounted from stephendefi/balancer
   f790aed835ee: Mounted from stephendefi/balancer
   850c2400ea4d: Mounted from stephendefi/balancer
   7ccabd267c9f: Mounted from stephendefi/balancer
   f5600c6330da: Mounted from stephendefi/balancer
   latest: digest: sha256:ecad3d644348320319d28d15a6224567045dc80cf71b82e965f9cf2ef314a273 size: 1573
   ```

   

#### akash部署环境准备(Edgenet测试网)

  **安装钱包**

  这里使用的环境是ubuntu18.04

1. 下载[akash钱包程序](https://github.com/ovrclk/akash/releases/download/v0.9.0-rc13/akash_0.9.0-rc13_linux_arm64.zip)

   ```
   wget https://github.com/ovrclk/akash/releases/download/v0.9.0-rc13/akash_0.9.0-rc13_linux_arm64.zip
   ```

2. 解压

   ``` 
   tar -xvf akash_0.9.0-rc13_linux_arm64.zip
   ```

3. 重命名目录

   ```
   mv akash_0.9.0-rc13_linux_arm64 akash
   ```

4. 配置环境变量

   `/work/akash/akash` 为我们刚刚解压的路径

   编辑 `/etc/profile` 文件在最后加入一下内容

   ```
   export PATH=$PATH:/work/akash/akash
   ```

   之后运行

   ```
   akash version
   ```

   输出版本号

   ```
   v0.9.0-rc13
   ```

5. 创建钱包地址

   这里需要预先准备一些变量， `KEY_NAME` 是自己设置的字符串，默认 `alice`

   ```
   export KEY_NAME=alice
   export KEYRING_BACKEND=local
   ```

   生成钱包地址

   ```
   akash --keyring-backend "$KEYRING_BACKEND" keys add "$KEY_NAME"
   ```

   输出如下

   ```
   - name: alice
     type: local
     address: akash1cz87pqkad72gggrv3t7y2x9z56h9gqghlnx3j3
     pubkey: akashpub1addwnpepqtnydvj056gy64uuquldq5yx7mr8ncmn3ut59wwl9p83d8h2v4rtg5xa3vn
     mnemonic: ""
     threshold: 0
     pubkeys: []
   
   
   **Important** write this mnemonic phrase in a safe place. It is the only way to recover your account if you ever forget your password.
   
   town wolf margin parrot strong disease dance eyebrow inflict meadow crunch version tube elite interest movie uphold column shift fox excuse humble nest call
   ```

   >address 就是我们生成的账户地址
   >
   >最后的那24个单词非常重要，是我们钱包的助记词，需要我们保存好，恢复导入钱包的时候需要

**领取测试币**

这里需要先设置几个常用的变量

```
export AKASH_NET="https://raw.githubusercontent.com/ovrclk/net/master/edgenet"
export AKASH_CHAIN_ID="$(curl -s "$AKASH_NET/chain-id.txt")"
exprot AKASH_NODE="$(curl -s "$AKASH_NET/rpc-nodes.txt" | shuf -n 1)"
export ACCOUNT_ADDRESS=akash1cz87pqkad72gggrv3t7y2x9z56h9gqghlnx3j3
```

> AKASH_NET: 您要连接的网络的基本URL
>
> AKASH_CHAIN_ID: 需要连接的链的ID
>
>  AKASH_NODE: 节点RPC地址
>
> ACCOUNT_ADDRESS: 前面生成的钱包地址

运行命令查看领取测试币的水龙头地址

```
curl "$AKASH_NET/faucet-url.txt"
```

输出如下

```
https://akash.vitwit.com/faucet
```

在浏览器打开这个地址通过人机认证之后填入钱包地址确定，等待一会之后账户收到测试币，可以通过命令查看余额

```
akash --node "$AKASH_NODE" query bank balances "$ACCOUNT_ADDRESS"
```

输出如下

```
balances:
- amount: "135230674"
  denom: uakt
pagination:
  next_key: null
  total: "0"
```

akt的精度是6，所以余额应该是 135230674 uakt = 135 akt

#### 部署Ren Protocol

1. 根据应用部署[教程](https://docs.akash.network/v/master/guides/deploy)配置 `deploy.yml`文件

   创建&编辑`deploy.yml`文件

   ```
   touch deploy.yml
   vi deploy.yml
   ```

   加入以下内容

   ```
   ---
   version: "2.0"
   
   services:
     web:
       image: fishflyhuang/renprotocol
       expose:
         - port: 80
           as: 80
           to:
             - global: true
   
   profiles:
     compute:
       web:
         resources:
           cpu:
             units: 0.5
           memory:
             size: 512Mi
           storage:
             size: 512Mi
     placement:
       westcoast:
         attributes:
           organization: ovrclk.com
         signedBy:
           anyOf:
             - "akash1vz375dkt0c60annyp6mkzeejfq0qpyevhseu05"
         pricing:
           web:
             denom: uakt
             amount: 1000
   
   deployment:
     web:
       westcoast:
         profile: web
         count: 1
   ```

2. 部署

   ```
   akash tx deployment create deploy.yml --from $KEY_NAME --node $AKASH_NODE --chain-id $AKASH_CHAIN_ID -y
   ```

   输出如下

   ```
   {
     "height": "183906",
     "txhash": "FC58F52F49E91808A4751E5A5603C34489B253F59E40F7058E1CC5E70F036CA1",
     "codespace": "",
     "code": 0,
     "data": "0A130A116372656174652D6465706C6F796D656E74",
     "raw_log": "[{\"events\":[{\"type\":\"akash.v1\",\"attributes\":[{\"key\":\"module\",\"value\":\"deployment\"},{\"key\":\"action\",\"value\":\"deployment-created\"},{\"key\":\"version\",\"value\":\"f2b5e760ca93c53548737d76494db89b08d811f7e9b533aa07fd934bf51d6707\"},{\"key\":\"owner\",\"value\":\"akash1j8s87w3fctz7nlcqtkl5clnc805r240443eksx\"},{\"key\":\"dseq\",\"value\":\"19553\"}]},{\"type\":\"message\",\"attributes\":[{\"key\":\"action\",\"value\":\"create-deployment\"}]}]}]",
     "logs": [
       {
         "msg_index": 0,
         "log": "",
         "events": [
           {
             "type": "akash.v1",
             "attributes": [
               {
                 "key": "module",
                 "value": "deployment"
               },
               {
                 "key": "action",
                 "value": "deployment-created"
               },
               {
                 "key": "version",
                 "value": "f2b5e760ca93c53548737d76494db89b08d811f7e9b533aa07fd934bf51d6707"
               },
               {
                 "key": "owner",
                 "value": "akash1j8s87w3fctz7nlcqtkl5clnc805r240443eksx"
               },
               {
                 "key": "dseq",
                 "value": "19553"
               }
             ]
           },
           {
             "type": "message",
             "attributes": [
               {
                 "key": "action",
                 "value": "create-deployment"
               }
             ]
           }
         ]
       }
     ],
     "info": "",
     "gas_wanted": "200000",
     "gas_used": "53806",
     "tx": null,
     "timestamp": ""
   }
   ```

3. 查看刚刚部署的状态

   ```
   akash query market lease list --owner $ACCOUNT_ADDRESS --node $AKASH_NODE --state active
   ```

   输出如下

   ```
   - lease_id:
       dseq: "183906"
       gseq: 1
       oseq: 1
       owner: akash15rrya4ayu8yewh3d2agg552phtttpkghpd0paf
       provider: akash174hxdpuxsuys9qkauaf57ym5j8dm4secnz6jd7
     price:
       amount: "331"
       denom: uakt
     state: active
   pagination:
     next_key: null
     total: "0"
   ```

   记录下前面的输出的几个变量备用

   ```
   export PROVIDER=akash174hxdpuxsuys9qkauaf57ym5j8dm4secnz6jd7
   export DSEQ=183906
   export OSEQ=1
   export GSEQ=1
   ```

4. 上传 `Manifest`

   ```
   akash provider send-manifest deploy.yml --node $AKASH_NODE --dseq $DSEQ --oseq $OSEQ --gseq $GSEQ --owner $ACCOUNT_ADDRESS --provider $PROVIDER
   ```

5. 查看部署的具体信息

   ```
   akash provider lease-status --node $AKASH_NODE --dseq $DSEQ --oseq $OSEQ --gseq $GSEQ --provider $PROVIDER --owner $ACCOUNT_ADDRESS
   ```

   输出如下

   ```
   {
     "services": {
       "web": {
         "name": "web",
         "available": 0,
         "total": 1,
         "uris": [
           "c5fh0i0ietclnc6ush378isr5s.provider2.akashdev.net"
         ],
         "observed-generation": 0,
         "replicas": 0,
         "updated-replicas": 0,
         "ready-replicas": 0,
         "available-replicas": 0
       }
     },
     "forwarded-ports": {}
   }
   ```

6. 访问部署的应用

   `http://c5fh0i0ietclnc6ush378isr5s.provider2.akashdev.net/`

7. 关闭部署

   ```
   akash tx deployment close --node $AKASH_NODE --chain-id $AKASH_CHAIN_ID --dseq $DSEQ --owner $ACCOUNT_ADDRESS --from $KEY_NAME -y
   ```



到这里部署全部完成。

