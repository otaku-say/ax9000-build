# ax9000-build

Xiaomi AX9000 固件云端构建仓库 —— 基于 [VIKINGYFY/immortalwrt](https://github.com/VIKINGYFY/immortalwrt)（main 分支，满血 NSS 驱动）。

## 特性

- **源码**：VIKINGYFY/immortalwrt @ main（qualcommax / ipq807x，内核 6.18）
- **PassWall**：官方源（Openwrt-Passwall），含 Xray / Sing-Box / SS-Rust / Hysteria / ShadowTLS / obfs / 插件 / Geoview / HAProxy 全套及完整依赖
- **QCN9074 修复**：内置 vendor board-2.bin（160MHz），编译期覆盖 + 首次启动兜底，双保险
- **硬件控制**：内置 `luci-app-ax900-hardware`（来源 [nixevol/Ax9000WRTBuild](https://github.com/nixevol/Ax9000WRTBuild)，PolyForm NC 1.0.0 非商业许可）——顶部 RGB 七色/轮播/闪烁、前灯灯效、联网指示、温度联动、风扇自动/手动、实时温度与转速
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
| `*-xiaomi_ax9000-squashfs-sysupgrade.bin` | 主固件（OpenWrt expand layout，`cat /proc/device-tree/model` 输出 `Xiaomi AX9000`） |
| `*-initramfs-*.ubi` | initramfs 中转镜像（首次从原厂刷入时使用） |

## 目录结构

```
├── .github/workflows/build.yml      # 云编译工作流
├── configs/seed.config              # 配置种子（defconfig 自动补全）
├── packages/luci-app-ax900-hardware # AX9000 硬件控制应用（LED/风扇/温度）
├── files/                           # 固件默认化层（注入 rootfs）
│   ├── etc/uci-defaults/99-ax9000-defaults  # 首启默认配置
│   ├── etc/sysctl.conf              # fq + BBR
│   ├── etc/board-2.bin              # vendor board 备份
│   └── lib/firmware/ath11k/QCN9074/hw1.0/board-2.bin
└── README.md
```

## LED 分工（AX9000 三处灯位）

| 灯位 | 通道 | 默认职责（可在 LuCI → AX9000 硬件 中调整） |
|---|---|---|
| 顶部大灯 | `red:` / `green:` / `blue:_2` | 固定蓝色常亮（可选：温度联动变色 / 七色轮播 / 闪烁 / 灭） |
| 前面-下灯 | `blue:` / `yellow:` | 系统标准灯语（启动黄闪 → 就绪蓝常亮） |
| 前面-上灯 | `blue:_1` / `yellow:_1` | 联网指示：在线绿 / 断网红（可改 WAN 活动灯等） |

说明：AX9000 LED 为 GPIO 直驱（无硬件 PWM），支持开/关/闪烁，不支持平滑调光。
风扇：EMC2301 由 `luci-app-ax900-hardware` 管理（默认自动温控：≤50°C 停转 / ≥50°C 25% / ≥60°C 50% / ≥70°C 100%）。

## 致谢

- [VIKINGYFY/immortalwrt](https://github.com/VIKINGYFY/immortalwrt) — 源码基础（NSS 驱动）
- [bidhata/BidhataWRT-Xiaomi-AX9000](https://github.com/bidhata/BidhataWRT-Xiaomi-AX9000) — QCN9074 vendor board-2.bin
- [Openwrt-Passwall](https://github.com/Openwrt-Passwall) — PassWall
- [nixevol/Ax9000WRTBuild](https://github.com/nixevol/Ax9000WRTBuild) — luci-app-ax900-hardware 硬件控制应用（PolyForm Noncommercial 1.0.0，仅限非商业使用）

## 注意

- 首次云端构建约 3~5 小时（NSS 驱动编译量大），后续构建有 ccache 缓存加速
- 未包含 NaiveProxy（体积大）；需要可在 `configs/seed.config` 增加 `CONFIG_PACKAGE_luci-app-passwall_INCLUDE_NaiveProxy=y`
- 5GHz 高频段若要 160MHz，需在 LuCI 无线设置中手动选择对应信道（如 36/100 段）与带宽
