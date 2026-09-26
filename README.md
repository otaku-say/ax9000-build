# ax9000-build

Xiaomi AX9000 固件云端构建仓库 —— 基于 [VIKINGYFY/immortalwrt](https://github.com/VIKINGYFY/immortalwrt)（main 分支，满血 NSS 驱动）。

## 特性

- **源码**：VIKINGYFY/immortalwrt @ main（qualcommax / ipq807x，内核 6.18）
- **PassWall**：官方源（Openwrt-Passwall），含 Xray / Sing-Box / SS-Rust / Hysteria / ShadowTLS / obfs / 插件 / Geoview / HAProxy 全套及完整依赖
- **QCN9074 修复**：内置 vendor board-2.bin（160MHz），编译期覆盖 + 首次启动兜底，双保险
- **LED 修复**：dts 补丁为顶部大灯注册稳定设备名（`top:red` / `top:green` / `top:blue`）；`topled` 服务提供联网状态指示
- **默认配置**：
  - SSID：`AX9000` / `AX9000-5G` / `AX9000-5G-2`（无密码）
  - 后台免密登录（root 空密码）
  - LAN 地址 `192.168.31.1`
  - 网络优化：fq + BBR

## 构建

在仓库 Actions 页手动触发 `Build AX9000 Firmware`，或 push 到 main 自动触发。
产物自动发布到 Releases。

> 公开仓库的 GitHub Actions 免费且不限分钟数。

## 构建产物

| 文件 | 说明 |
|---|---|
| `*-xiaomi_ax9000-squashfs-sysupgrade.bin` | OpenWrt expand layout（`/proc/device-tree/model` = `Xiaomi AX9000`） |
| `*-xiaomi_ax9000-stock-squashfs-sysupgrade.bin` | stock layout（model 含 `(stock layout)`） |
| `*-initramfs-*.ubi` | initramfs 中转镜像（首次从原厂刷入时使用） |

设备布局判断：SSH 执行 `cat /proc/device-tree/model`，与当前系统匹配即可。

## 目录结构

```
├── .github/workflows/build.yml      # 云编译工作流
├── configs/seed.config              # 配置种子（defconfig 自动补全）
├── patches/0001-ax9000-led-labels.patch   # 顶部灯设备名补丁
├── files/                           # 固件默认化层（注入 rootfs）
│   ├── etc/uci-defaults/99-ax9000-defaults  # 首启默认配置
│   ├── etc/init.d/topled            # 顶部灯服务
│   ├── etc/sysctl.conf              # fq + BBR
│   ├── usr/bin/topled-daemon        # 联网状态指示守护
│   ├── etc/board-2.bin              # vendor board 备份
│   └── lib/firmware/ath11k/QCN9074/hw1.0/board-2.bin
└── README.md
```

## LED 分工（AX9000 三处灯位）

| 灯位 | 通道 | 职责 |
|---|---|---|
| 顶部大灯 | `top:red` / `top:green` / `top:blue` | `topled` 服务：启动蓝闪 → 在线绿 / 断网红闪 |
| 前面-下灯 | system-yellow / system-blue | 系统标准灯语（启动黄闪 → 就绪蓝常亮，不干预） |
| 前面-上灯 | network-blue / network-yellow | 默认灭，可在 LuCI → 系统 → LED 配置中自定义 |

说明：AX9000 LED 为 GPIO 直驱（无硬件 PWM），支持开/关/闪烁，不支持平滑调光。

## 致谢

- [VIKINGYFY/immortalwrt](https://github.com/VIKINGYFY/immortalwrt) — 源码基础（NSS 驱动）
- [bidhata/BidhataWRT-Xiaomi-AX9000](https://github.com/bidhata/BidhataWRT-Xiaomi-AX9000) — QCN9074 vendor board-2.bin
- [Openwrt-Passwall](https://github.com/Openwrt-Passwall) — PassWall
- [qambain/luci-app-router-led](https://github.com/qambain/luci-app-router-led) — 顶部灯控制思路参考

## 注意

- 首次云端构建约 3~5 小时（NSS 驱动编译量大），后续构建有 ccache 缓存加速
- 未包含 NaiveProxy（体积大）；需要可在 `configs/seed.config` 增加 `CONFIG_PACKAGE_luci-app-passwall_INCLUDE_NaiveProxy=y`
- 5GHz 高频段若要 160MHz，需在 LuCI 无线设置中手动选择对应信道（如 36/100 段）与带宽
