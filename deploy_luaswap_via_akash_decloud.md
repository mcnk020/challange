### 通过Akash DeCloud部署LuaSwap

**Akash DeCloud**

Akash是一个去中心化的云服务商，可以帮助我们以低成本像在AWS上部署应用一样部署一个去中心化应用。

整个教程大概分为准备docker镜像、akash部署环境准备、部署LuaSwap几部分。

#### 准备docker镜像

我们从[LuaSwap](https://hub.docker.com/r/minatofund/luaswap)找到我们所需要的docker镜像

```
docker pull minatofund/luaswap
```

#### Akash部署环境准备(Edgenet测试网)

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

1. 先设置几个常用的变量

   ```
   export AKASH_NET="https://raw.githubusercontent.com/ovrclk/net/master/edgenet"
   export AKASH_CHAIN_ID="$(curl -s "$AKASH_NET/chain-id.txt")"
   exprot AKASH_NODE="$(curl -s "$AKASH_NET/rpc-nodes.txt" | shuf -n 1)"
   export ACCOUNT_ADDRESS=akash1cz87pqkad72gggrv3t7y2x9z56h9gqghlnx3j3
   ```

   >AKASH_NET: 您要连接的网络的基本URL
   >
   >AKASH_CHAIN_ID: 需要连接的链的ID
   >
   >AKASH_NODE: 节点RPC地址
   >
   >ACCOUNT_ADDRESS: 前面生成的钱包地址
2. 运行命令查看领取测试币的水龙头地址

   ```
   curl "$AKASH_NET/faucet-url.txt"
   ```

   输出如下

   ```
   https://akash.vitwit.com/faucet
   ```

3. 在浏览器打开这个地址通过人机认证之后填入钱包地址确定，等待一会之后账户收到测试币，可以通过命令查看余额

   ```
   akash --node "$AKASH_NODE" query bank balances "$ACCOUNT_ADDRESS"
   ```

   输出如下

   ```
   balances:
   - amount: "100000000"
     denom: uakt
   pagination:
     next_key: null
     total: "0"
   ```

   akt的精度是6，所以余额应该是 100000000 uakt = 100 akt

#### 部署LuaSwap

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
     mysql:
       image: minatofund/luaswap 
       expose:
         - port: 3306 
           to:
             - global: true
         - port: 8080
           as: 80
           to:
             - global: true
   profiles:
     compute:
       mysql:
         resources:
           cpu:
             units: 0.1 
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
           mysql: 
             denom: uakt
             amount: 3000
   deployment:
     mysql:
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
       "height": "206495",
       "txhash": "829D7AEE522B01A66E03B8C9CC5B81DAC7BD8FC70BC633C4909592F285C772A5",
       "codespace": "",
       "code": 0,
       "data": "0A130A116372656174652D6465706C6F796D656E74",
       "raw_log": "[{\"events\":[{\"type\":\"akash.v1\",\"attributes\":[{\"key\":\"module\",\"value\":\"deployment\"},{\"key\":\"action\",\"value\":\"deployment-created\"},{\"key\":\"version\",\"value\":\"48a23fa0aeedb85e4a26e2aa4f0d0abf6f89a25dd42d3e1bf27f889617dce742\"},{\"key\":\"owner\",\"value\":\"akash19uwxt0ln3p9fenpdwtsm5rlmqsekc3g9ym6xqz\"},{\"key\":\"dseq\",\"value\":\"206494\"}]},{\"type\":\"message\",\"attributes\":[{\"key\":\"action\",\"value\":\"create-deployment\"}]}]}]",
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
                               "value": "48a23fa0aeedb85e4a26e2aa4f0d0abf6f89a25dd42d3e1bf27f889617dce742"
                           },
                           {
                               "key": "owner",
                               "value": "akash19uwxt0ln3p9fenpdwtsm5rlmqsekc3g9ym6xqz"
                           },
                           {
                               "key": "dseq",
                               "value": "206494"
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
       "gas_used": "56891",
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
   leases:
   - lease_id:
       dseq: "206494"
       gseq: 1
       oseq: 1
       owner: akash19uwxt0ln3p9fenpdwtsm5rlmqsekc3g9ym6xqz
       provider: akash1ds82uk3pzawavlasc2qd88luewzye2qrcwky7t
     price:
       amount: "102"
       denom: uakt
     state: active
   pagination:
     next_key: null
     total: "0"
   ```

   记录下前面的输出的几个变量备用

   ```
   export PROVIDER=akash1ds82uk3pzawavlasc2qd88luewzye2qrcwky7t
   export DSEQ=206494
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
           "02113q2tkdf1r64buuanq03da4.provider5.akashdev.net"
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

   `http://02113q2tkdf1r64buuanq03da4.provider5.akashdev.net/`

7. 关闭部署

   ```
   akash tx deployment close --node $AKASH_NODE --chain-id $AKASH_CHAIN_ID --dseq $DSEQ --owner $ACCOUNT_ADDRESS --from $KEY_NAME -y
   ```



到这里部署全部完成。


