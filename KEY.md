# 控制台密钥与发布凭据

## 一句话

**能控制全体用户的人，只需要持有这把控制台密钥。** 不再需要单独的 GitHub 写入权限或令牌。

## 密钥

- 形如 `zk-ctrl-` 加 16 位十六进制，例如由 `python3 -c "import secrets; print('zk-ctrl-' + secrets.token_hex(8))"` 生成；
- 明文存放位置：私有仓库 `PWHbut/zhike` 的 `CONSOLE_KEY.txt`、管理员本机 `/workspace/CONSOLE_KEY.txt`；
- 公开仓库 `zk-guard` 的 `key.json` 只放 `sha256(密钥)`，控制台用它做本地校验，不能反推明文。

密钥同时用于两件事：
1. 对 `guard.json` / `hot.json` 生成 `auth` / `authV2` 签名，客户端据此决定是否执行策略；
2. 派生 AES-256-GCM 密钥，在本机解开 `cred.json` 得到发布凭据。

## 发布凭据（cred.json）

`cred.json` 是密文：`{ v, k, s, i, c }`。派生方式：

```
key = SHA-256( UTF8(控制台密钥 + "|" + s) )
明文 = AES-256-GCM-Decrypt(key, iv=i, ciphertext=c)   // c 末尾含 16 字节 auth tag
```

控制台在浏览器里用 `crypto.subtle` 完成解密，凭据只存在于内存变量中，
不写入 DOM、localStorage、cookie，也不会随任何请求体上传到别的服务器——
它只作为 `Authorization` 头发给 `api.github.com` 用于写本仓库的配置文件。

密文不含可读令牌，GitHub 的密钥扫描不会识别；没有控制台密钥则无法解开。

## 轮换

- **换密钥**：生成新密钥 → 更新私有仓库 `CONSOLE_KEY.txt` → 重新计算 `key.json` 哈希 →
  用新密钥重新加密生成 `cred.json` → 更新客户端内置的 `GUARD_SECRET`。
  ⚠️ 会造成未升级的旧客户端暂时读不懂新签名，需配合发版。
- **换凭据**：管理员重新生成 `cred.json` 并推送，控制台页面无需改动，操作员无感。

## 验证清单

- [ ] 页面版本号为 `v5.10.0`
- [ ] 顶部显示「发布通道：密钥直连 CDN · 已就绪」
- [ ] 页面里搜不到「令牌 / GitHub 编辑 / 复制 JSON 粘贴」等旧流程字样
- [ ] 填密钥点「停止」后，点「验证控制效果」显示「客户端：不可用」
- [ ] 点「启动」后恢复「可正常使用」
