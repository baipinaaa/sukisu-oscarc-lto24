# manager-signing — BakaSU Manager APK 的签名密钥（**密钥不在本目录**）

本目录**只保留说明**。keystore 本体与密码都存在 GitHub Secrets：

| Secret 名 | 内容 |
|---|---|
| `MANAGER_KEYSTORE_BASE64` | `base64 -w0 bakasu-manager-v2.jks` 的输出 |
| `MANAGER_KEYSTORE_PASSWORD` | 该 keystore 的 storepass（= keypass） |

`.github/workflows/build-bakasu-manager.yml` 在「安装签名配置」步骤里
`printf '%s' "$KEYSTORE_BASE64" | base64 -d > bakasu-manager.jks` 还原，未配置则直接失败。

## 当前密钥参数

| 项 | 值 |
|---|---|
| 别名 alias | `bakasu` |
| 密钥 | RSA 4096 / SHA384withRSA |
| storetype | PKCS12 |
| 有效期 | 10000 天 |
| 证书主体 | `CN=BakaSU Manager (oscarc), OU=SelfBuild, O=oscarc-lto24, C=CN` |
| 证书 DER 字节数 | 1373 = `0x55d` |
| 证书 DER SHA-256 | `48480a1fc5145334059cfc0f6c767dd5b6fdf05d4ffe2a5af03e80478c909a7d` |

这两个值（size / hash）就是内核 `kernel/manager/manager_sign.h` 里
`EXPECTED_SIZE_BAKASU` / `EXPECTED_HASH_BAKASU` 的取值，两份 workflow 顶层 env 里各写一份。

## ⚠ 已弃用的第一代密钥

`8e2d4ac` ~ `45e33ba` 期间，仓库里直接放着 `bakasu-manager.jks`（证书 SHA-256
`71:f8:a8:1d:...:cc:20`）且密码明文写在 workflow 的 `env` 里。**该密钥已作废**：

- 它出现在公开仓库的 git 历史里，删除文件不能让任何人都取不到
- `45e33ba` 之后内核信任「该签名的 APK = manager」，而 manager 可改 allowlist 授权 root、
  装 system 模块 —— **谁拿到这把钥匙，谁就能造出一个被本内核认定为 manager 的 APK**
- 当前内核已改为只信任 13.3 轮换后的新证书（`0x55d` / `48480a1f...`）

若在旧设备/旧 boot.img 上见到 `71f8a81d...` 生效，说明还停留在旧内核，应立刻重刷。

## 为什么不用官方 key

BakaSU 官方 main 分支不发布 APK（最新 release 仍是 `v4.2.0-rc3`，UAPI v4，而内核侧已升到
main / UAPI v6），官方 CI 的 PR 构建用的是**一次性随机 keystore**，每次签名都不同。
自己固定一份密钥后：

- 同一包名 `com.resukisu.resukisu`，**同一密钥签出的新版本可直接覆盖安装**，不用卸载
- 与官方 `v4.2.0-rc3` 签名不同，所以**第一次从官方版切过来仍需卸载一次**

## 校验装上的 APK 是不是这份密钥签的

```bash
apksigner verify --print-certs BakaSU_v4.x.y_35xxx-arm64-v8a-release.apk | grep SHA-256
# 期望 48480a1fc5145334059cfc0f6c767dd5b6fdf05d4ffe2a5af03e80478c909a7d
```

或设备侧（连 APK 文件都不用拷出来）：
```bash
adb shell su -c '/data/adb/ksud debug get-sign $(pm path com.resukisu.resukisu | sed "s/package://")'
```
