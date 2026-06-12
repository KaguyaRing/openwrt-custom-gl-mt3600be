![OpenWrt logo](include/logo.png)

<div align="center">

<a href="README.md">English</a> ｜
<a href="README_CN.md">中文</a>

</div>

# openwrt-custom-gl-mt3600be

这是一个基于官方 OpenWrt Snapshot 的定制分支，专为 `GL-MT3600BE` 设备适配，在主线基础上集成了常用软件包与开箱即用的设备默认配置。

## 当前状态

- 目标设备: `GL.iNet GL-MT3600BE`
- 目标平台: `mediatek/filogic`
- 固件产物: `openwrt-mediatek-filogic-glinet_gl-mt3600be-squashfs-sysupgrade.bin`
- 管理地址: `http://192.168.3.1`
- 已验证: 启动、双频 Wi-Fi、LuCI、风扇、AdGuardHome、OpenClash

## 当前构建固件集成包

- LuCI + Argon 主题 `2.4.3`
- `opkg` 与 `apk`
- OpenClash `v0.47.055` (Meta/Mihomo `v1.19.27` – 内核已预置，开箱即用)
- AdGuardHome(默认运行在 `http://192.168.3.1:3000`,具体以实际路由器IP地址为准)

- 常用中文包:
  - `luci-i18n-base-zh-cn`
  - `luci-i18n-package-manager-zh-cn`
  - `luci-i18n-firewall-zh-cn`

## 默认配置 (首次启动)

- **管理地址**: `http://192.168.3.1`
- **唯一可登录用户** — 用户名: `admin` / 密码: `admin`（等效 root 权限，uid 0）
- **`root` 账号** — 已锁定（无法登录）；请使用 `admin` 用户登录 LuCI 和 SSH
  - 如需解锁 root：以 `admin` SSH 登录后执行 `passwd -u root && passwd root` 设置密码即可
- **WAN 侧已放通端口**:
  - OpenClash: `7874`, `7890-7893`, `7895`, `9090` (TCP)
  - AdGuardHome: `3000` (TCP)
- **OpenClash 面板**: `http://192.168.3.1:9090` (密钥由 OpenClash 首次运行时自动生成)
- 2.4G SSID: `GL-MT3600BE-<MAC末4位>`
- 5G SSID: `GL-MT3600BE-<MAC末4位>-5G`
- Wi-Fi 密码: `88888888`
- 访客网段预留: `192.168.4.1/24`（默认关闭）
- 访客 SSID 预留: `GL-MT3600BE-<MAC末4位>-Guest` / `GL-MT3600BE-<MAC末4位>-Guest-5G`（默认关闭）
- `wan2` 预置: `DHCP`，`metric=30`，默认不绑定设备（可在 LuCI 中手动选择 USB 网卡/USB 共享接口）
- `wan2_6` 预置: `proto=none`（未配置）

### 可启用的配置

- 访客 Wi-Fi（2.4G/5G）
- `wan2` 备用上行（USB 共享网络/USB 网卡）

## 构建

如果你想自行构建？很简单，按照以下步骤进行。

首先，安装编译所需的依赖包（以 Debian/Ubuntu 为例）：

```bash
sudo apt update
sudo apt install build-essential libncurses5-dev gawk git libssl-dev zlib1g-dev file python3 python3-distutils rsync subversion unzip wget
```

然后，克隆仓库并开始编译：
```bash
git clone https://github.com/ChuranNeko/openwrt-custom-gl-mt3600be
cd openwrt-custom-gl-mt3600be
./scripts/feeds update -a
./scripts/feeds install -a
make defconfig
make -j$(nproc)
```

> 如你使用 `Root` 用户进行编译，建议使用 `export FORCE_UNSAFE_CONFIGURE=1; make -j$(nproc)` 进行编译。

编译完成后，固件位于 `bin/targets/mediatek/filogic/` 目录。

> 如你需要在编译时加入其他可选包，请先运行 `make menuconfig` 配置后再编译。

[已构建固件下载](https://github.com/ChuranNeko/openwrt-custom-gl-mt3600be/releases/latest)

## 刷机方法

从 `U-Boot` 自行上传

[GL-INet 官方教程](https://docs.gl-inet.cn/router/4/features/uboot/#2)

或者自行从 `管理后台` 自行上传,确保备份数据，不建议保留数据升级，因为版本差异较大。

[GL-INet 官方教程](https://docs.gl-inet.cn/router/4/features/upgrade/#_3)

再或者使用 `SCP` 上传至 `/tmp` 目录，随后在 `SSH` 运行以下命令:

```bash
scp bin/targets/mediatek/filogic/openwrt-mediatek-filogic-glinet_gl-mt3600be-squashfs-sysupgrade.bin root@<路由器ip>:/tmp/

# 先做镜像完整性校验（推荐）
ssh root@<路由器ip> 'sysupgrade -T /tmp/openwrt-mediatek-filogic-glinet_gl-mt3600be-squashfs-sysupgrade.bin'

# 跨体系/大版本升级建议不保留配置（推荐）
ssh root@<路由器ip> 'sysupgrade -n /tmp/openwrt-mediatek-filogic-glinet_gl-mt3600be-squashfs-sysupgrade.bin'

# 如需保留配置，可改用（风险自担）
# ssh root@<路由器ip> 'sysupgrade /tmp/openwrt-mediatek-filogic-glinet_gl-mt3600be-squashfs-sysupgrade.bin'
```

> `-T` 仅用于校验镜像，不会执行刷写。
>
> `-n` 表示不保留配置。对于从原厂/其它分支迁移到本固件，建议使用 `-n`。

## 说明

- 本仓库是设备定制分支，不是官方发布仓库。
- 如需与官方主线对齐，请以 `openwrt/openwrt` 为上游进行 rebase/merge。

## 适配要点

- MT3600BE 设备树、镜像定义、网络映射、LED 与 USB 供电脚本已适配。
- 已加入 `turn_on_usb_power` 开机脚本，启动后会拉高 `usb_power` GPIO(参考原厂固件后添加)。
- Wi-Fi 已验证可正常拉起双频 AP。
- **风扇曲线优化**：重新映射了 PWM 风扇转速，新曲线为 0/96/160/224/255（对应 0%/38%/63%/88%/100%），相比原厂 0/64/128/192/255（0%/25%/50%/75%/100%）起步风量更大、过渡更平滑，兼顾散热与静音。
- 已集成 USB 共享网络常见协议驱动与工具包（RNDIS/ECM/NCM/MBIM/QMI 等）。

## 包管理说明

- 当前 Snapshot 体系以 `apk` 为主。
- 镜像中同时包含 `opkg`，用于兼容部分场景。
- 第三方插件若强依赖纯 `opkg` 元数据，可能出现依赖识别不一致。

## 已知事项

- OpenClash Meta（mihomo）内核已预置；GeoIP/GeoSite/Country 数据库由 OpenClash 首次运行时自动更新（通过 jsDelivr CDN，国内通常可访问）。
- USB 设备不枚举时，优先检查电源规格与线材，建议使用足功率适配器。
- 本Openwrt版本不支持 `iStore` 的安装，因为 `iStore` 的最高支持是在 Openwrt `24.10`。

## 许可

本项目基于 OpenWrt 进行二次定制，整体遵循上游开源许可。

- 上游 OpenWrt: `GPL-2.0`
- 本仓库新增脚本/配置默认同样按 `GPL-2.0` 方式发布

第三方组件（如 LuCI、AdGuardHome、OpenClash、Argon 等）遵循各自上游许可证，具体以对应源码与包元数据为准。

## 感谢

- 感谢 OpenWrt 社区与维护者提供稳定的主线基础。
- 感谢 GL.iNet 社区与相关资料贡献者，帮助完成 MT3600BE 适配排障。
- 感谢 x-wrt 相关实现提供参考路径（如设备适配与驱动迁移思路）。
- 感谢我自己:)。

> 本人自己其实想适配 24.10的 可惜屡屡碰壁,干脆就直接拿 最新的 `Openwrt` 来做的适配。

## 赞助

[请我喝杯咖啡](https://blog.crneko.top/sponsor/)
