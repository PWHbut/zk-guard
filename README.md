# zk-guard · 知课全局开关配置仓库

这是知课客户端读取的**公开配置仓库**。客户端从这里并发读取 `guard.json` / `hot.json`，
用内置的对称密钥校验签名后决定「放行 / 停用 / 提示升级」。

## 控制台（唯一入口）

打开 **https://pwhbut.github.io/zk-guard/** ，填入**控制台密钥**，直接点
「停止所有用户服务」或「启动所有用户服务」即可。

- **不需要 GitHub 令牌**，页面上没有任何令牌输入框；
- **不需要后端**，纯静态页面自己完成发布；
- **不需要填地址、不需要复制 JSON、不需要去 GitHub 手动编辑**；
- 点「验证控制效果」可立即确认客户端当前是否处于不可用状态。

控制台密钥明文只放在私有仓库 `PWHbut/zhike` 的 `CONSOLE_KEY.txt`（仅仓库所有者可见）。
本公开仓库的 `key.json` 只有 SHA-256 哈希，`cred.json` 是用该密钥加密后的发布凭据密文，
没有密钥无法解开，也不含任何可读的令牌明文。

## 文件说明

| 文件 | 用途 | 谁读 |
| --- | --- | --- |
| `index.html` | 控制台页面（GitHub Pages，只需密钥） | 操作员（浏览器） |
| `cred.json` | AES-256-GCM 加密的发布凭据（密文，公开存放） | 控制台页面 |
| `key.json` | 控制台密钥的 SHA-256 哈希（单向校验） | 控制台页面 |
| `guard.json` | 全局开关与按版本控制策略 | 所有客户端 |
| `hot.json` | 界面热更新（css / js / flags） | v5.7+ 客户端 |
| `GUIDE.md` / `KEY.md` / `POLICY.md` | 使用说明、密钥管理、协议说明 | 管理员 |

## 发布链路

控制台用密钥在本机通过 WebCrypto 解开 `cred.json`，得到发布凭据后直接跨域调用
GitHub Contents API 写入配置文件，刷新 jsDelivr CDN，并**回读校验 rev 一致**才算成功。
凭据与密钥都只在浏览器内存中短暂存在，不上传、不落盘、不写入 localStorage。

## 回档

每一次发布都是本仓库的一次 commit，可直接用历史 commit 还原 `guard.json`，
或把默认策略改回 `allow` 一键恢复。客户端安装包版本另见 `PWHbut/zhike` 的 Release。
