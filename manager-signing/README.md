# manager-signing — BakaSU Manager APK 的自用签名密钥

`bakasu-manager.jks` 是 **本仓库专用** 的 APK 签名密钥，由
`.github/workflows/build-bakasu-manager.yml` 在构建 BakaSU Manager 时使用。

## 参数

| 项 | 值 |
|---|---|
| 文件 | `bakasu-manager.jks`（PKCS12） |
| 别名 alias | `bakasu` |
| 有效期 | 10000 天（约 27 年） |
| 密钥 | RSA 4096 / SHA384withRSA |
| 证书 | `CN=BakaSU Manager (sukisu-oscarc-lto24), OU=SelfBuild, O=baipinaaa, L=NA, ST=NA, C=CN` |
| 证书 SHA-256 | `71:F8:A8:1D:8B:3F:48:05:4D:0E:6F:B8:F2:5C:9F:43:39:7B:B0:48:B3:A0:9B:C0:5B:F8:29:D8:08:BC:CC:20` |
| 密码 | 见下方「为什么密码是明文」 |

## 为什么签名要自己搞一份

BakaSU 官方 main 分支没有发布 APK（最新 release 仍是 `v4.2.0-rc3`，UAPI v4，
而内核侧已经升到 main / UAPI v6）。官方 CI 只在 PR 上跑 `Build Manager`，而且
PR 构建用的是 **一次性随机 keystore** —— 每次签名都不同，装一次就要卸载一次。

自己固定一份密钥后：

- 同一个包名 `com.resukisu.resukisu`，**可以直接覆盖安装**，不用卸载
- 与官方 `v4.2.0-rc3` 的签名不同，所以 **第一次从官方版切过来仍需卸载一次**
- 之后每次 CI 重新构建都能无缝升级

## 为什么密码是明文

密码写在 `.github/workflows/build-bakasu-manager.yml` 的 `env:` 里，同时也在
这里是公开可见的。这是**有意为之**：

- 本仓库是个人自用的公开仓库，密钥泄漏的最坏后果只是「别人也能签出同签名的
  BakaSU Manager」，没有其它资产受影响
- 用 GitHub Secrets 更安全，但那样构建就无法完全复现，且这个 workflow 的
  定位就是「一条命令出 APK」
- 如果你要把这套东西给别人用，请**自己重新生成密钥**并改用 Secrets

重新生成的方法：

```bash
keytool -genkeypair -v \
  -keystore bakasu-manager.jks -alias bakasu \
  -keyalg RSA -keysize 4096 -validity 10000 -storetype PKCS12 \
  -storepass '<新密码>' -keypass '<新密码>' \
  -dname "CN=BakaSU Manager, OU=SelfBuild, O=you, C=CN"
```

生成后替换本目录文件、改 workflow 里的 `KEYSTORE_PASSWORD` 即可。

## 指纹核对

装完 APK 后可以这样确认它确实是用这份密钥签的：

```bash
apksigner verify --print-certs BakaSU_v4.3.0_35xxx-arm64-v8a-release.apk | grep SHA-256
# 应输出 71f8a81d8b3f48054d0e6fb8f25c9f43397bb048b3a09bc05bf829d808bccc20
```
