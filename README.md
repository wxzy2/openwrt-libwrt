# 360 V6 / 京东云 AX6600 — LibWrt NSS 云编译

基于 [laipeng668/openwrt-ci-roc](https://github.com/laipeng668/openwrt-ci-roc) 精简，保留三个编译流程。

## 三个编译流程

| 流程 | 设备 | 源码 / 内核 / NSS | 内置插件 | Release 标签 |
| --- | --- | --- | --- | --- |
| `360v6-jd6600.yml` | 360 V6、京东云 AX6600 | `openwrt-6.x`（25.12-nss）/ 6.12 / NSS 11.4（含 mesh） | 都有 OpenClash（预置 clash_meta 内核）、DDNS-Go；360 V6 另有 zram；AX6600 另有 OpenList2、Aria2、athena-led | `360v6-jd6600` |
| `IPQ807X-LibWrt.yml` | Linksys MX4300 | `openwrt-6.x`（25.12-nss）/ 6.12 / NSS 11.4（含 mesh） | OpenClash（预置 clash_meta 内核）、DDNS-Go、OpenList2、Aria2（无 athena-led、无 zram） | `IPQ807X` |
| `JDCloud-ImmortalWrt.yml` | 京东云 AX6600 | `immortalwrt`（openwrt-25.12）/ 6.18 / NSS 12.5 | OpenClash（预置 clash_meta 内核）、DDNS-Go、OpenList2、Aria2、athena-led（无 zram） | `AX6600-6.18` |

- NSS 版本是整次编译全局的，同一个流程里的设备必须一致。NSS 12.5 只能配 6.18（ImmortalWrt 源码），在 6.12 上 `qca-nss-ecm` 会编译失败；反过来 11.4 对应 6.12。
- 默认管理地址：`10.1.1.1`，用户 `root`，默认密码为空，**首次登录后请立即设置密码**
- 设备名称（主机名）保持源码默认，不做修改

## 文件结构

| 路径 | 作用 |
| --- | --- |
| `.github/workflows/360v6-jd6600.yml` | 入口 1：360 V6 + AX6600（6.12） |
| `.github/workflows/JDCloud-ImmortalWrt.yml` | 入口 2：AX6600（6.18） |
| `.github/workflows/IPQ807X-LibWrt.yml` | 入口 3：Linksys MX4300（ipq807x，6.12） |
| `.github/workflows/Build-OpenWrt.yml` | 被三个入口调用的通用构建流程，不要删 |
| `configs/360v6-jd6600.config` | 入口 1 的设备与 NSS 配置 |
| `configs/JDCloud.config` | 入口 2 的设备与 NSS 配置 |
| `configs/IPQ807X.config` | 入口 3 的设备、NSS 与插件配置 |
| `configs/General.config` | 各流程共用的通用配置与插件 |
| `scripts/Roc-script.sh` | 默认 IP、拉取第三方源码、编译前检查 |

## 增减插件

1. 两个设备都要的插件：在 `configs/General.config` 里写 `CONFIG_PACKAGE_xxx=y`；不要的写成 `=n`，仅加 `#` 无效。
   只给某个设备用的插件：在 `configs/360v6-jd6600.config` 里写 `CONFIG_PACKAGE_xxx=m`，并把包名加进该设备的 `DEVICE_PACKAGES`。
2. 如果插件不在 feeds 里，需要在 `scripts/Roc-script.sh` 里用 `package_enabled` 加一段 `clone_repository`。
3. 到 `Actions` 运行 `360v6-jd6600-LibWrt`，固件在 `Releases` 的 `360v6-jd6600` 标签下载。
