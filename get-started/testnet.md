---
order: 3
---

# 加入测试网

## 最新版本

DipperNetwork 测试网的最新版本是[testnet-v4.0.1](https://github.com/Dipper-Labs/Dipper-Protocol/releases/tag/testnet-v4.0.1)

## 安装

```bash
git clone -b testnet https://github.com/Dipper-Labs/Dipper-Protocol.git
cd Dipper-Protocol && git checkout testnet-v4.0.1
make install
```

编译安装完成后，检查版本号

```bash
dipd version
testnet-v4.0.1-0

dipcli version
testnet-v4.0.1-0
```

## 运行全节点

```bash
# 初始化节点
dipd init --moniker=<your-custom-name> --chain-id=dip-testnet

# 下载主网公开的 config.toml 和 genesis.json
curl -o ~/.dipd/config/config.toml https://raw.githubusercontent.com/Dipper-Labs/testnet/master/config/config.toml
curl -o ~/.dipd/config/genesis.json https://raw.githubusercontent.com/Dipper-Labs/testnet/master/config/genesis.json

# 启动节点（也可使用 nohup 或 systemd 等方式后台运行）
dipd start
```

## 申领测试token

```bash
# 将下面命令的<address>替换为你的测试网地址
curl https://docs.dipperNetwork.com/dip/get_token?<address>
```  

## 测试网相关链接

- RPC：<http://rpc.testnet.dippernetwork.com>

- 区块浏览器：<https://explorer.testnet.dippernetwork.com>

- 水龙头：<https://docs.dipperNetwork.com/dip/get_token?your_address>