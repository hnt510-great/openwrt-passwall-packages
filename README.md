# Passwall 代理组件 feed

**简体中文** | [English](README.en.md)

本仓库是 [hnt510-great/openwrt](https://github.com/hnt510-great/openwrt) 定制主树使用的私有源码 feed。
项目面向 x86_64/glibc、APK、原生 systemd 和 GNOME 桌面；软件选择以主树
保存的完整配置为准。

| 项目 | 值 |
| --- | --- |
| Feed 名称 | `passwall_packages` |
| 本仓库分支 | `main` |
| 原始上游 | [https://github.com/Openwrt-Passwall/openwrt-passwall-packages](https://github.com/Openwrt-Passwall/openwrt-passwall-packages)（`main`） |

本 feed 提供 Passwall 使用的代理核心、DNS 辅助工具和规则数据。主要配方包括
`xray-core`、`sing-box`、`hysteria`、`shadowsocks-rust`、`chinadns-ng`、
`smartdns` 和 `v2ray-geodata`；LuCI 管理界面位于独立的 `passwall_luci` feed。

## 与主树的集成

- 主树保留 Passwall 作为默认代理方案；`daed` 是 `package/custom` 中默认停用的替代方案。
- `daed-geoip`、`daed-geosite` 复用本 feed 的 `v2ray-geoip`、`v2ray-geosite` 数据。
- 原生 systemd 镜像通过主树的 procd 桥接保留 UCI 和 init 脚本接口；代理进程的
  cgroup、监控和停止行为应结合主树验证记录检查。
- 代理流量与内核直接转发流量使用不同路径。内核 full cone NAT 的结果不能直接
  当作代理出口的 NAT 行为。

## 获取与构建

先在构建机登录有权访问全部私有仓库的 GitHub 账号，再获取完整主树：

```sh
gh auth login --hostname github.com --git-protocol https --web
gh auth setup-git
git clone https://github.com/hnt510-great/openwrt.git
cd openwrt
./scripts/feeds update -a
./scripts/feeds install -a
```

主树 `feeds.conf.default` 已包含：

```text
src-git passwall_packages https://github.com/hnt510-great/openwrt-passwall-packages.git;main
```

在现有主树中只更新本 feed，可执行：

```sh
./scripts/feeds update passwall_packages
./scripts/feeds install -a -p passwall_packages
```

先在主树配置中选择所需包并运行 `make defconfig`。单包构建示例：

```sh
make package/feeds/passwall_packages/xray-core/compile V=s
```

以上命令均从主树根目录运行。保留已有 `.config`；新 checkout 的配置恢复
步骤见 [本地构建指南](https://github.com/hnt510-great/openwrt/blob/main/docs/local-build.md)。`install` 安装的是构建配方链接，
不会在设备上安装软件。完整 `update -a` 还会检查并可能修改主树的 custom 配方。

## 维护与验证

`origin` 指向本私有仓库。新 clone 需要合并原始上游时，先添加一次 `upstream`：

```sh
git -C feeds/passwall_packages remote add upstream https://github.com/Openwrt-Passwall/openwrt-passwall-packages.git
git -C feeds/passwall_packages fetch upstream
git -C feeds/passwall_packages merge upstream/main
# 审阅、解决冲突并完成相关验证后推送
git -C feeds/passwall_packages push origin main
```

已有 `upstream` 时跳过添加命令。`scripts/feeds update` 拉取本私有分支，
不会自动合并原始上游。本地集成修改与主树的提交分别管理，保留现有提交、
stash 和未提交文件。主树的 `patches/feeds/` 和功能清单是历史备份，
不要向已经包含修改的 feed 重复应用补丁。

涉及已发布 APK 内容的修改必须产生新包版本，不能覆盖同一包身份。源码
检查、包构建、VM 与物理设备验证应分别记录；本文汇总已有设计和验证记录，
不新增构建或设备验证结论。私有源码托管说明见
[GitHub 源码托管](https://github.com/hnt510-great/openwrt/blob/main/docs/github-source-hosting.md)。

## 设计与验证文档

- [代理方案与互斥关系](https://github.com/hnt510-great/openwrt/blob/main/docs/daed.md)
- [systemd cgroups 与 Passwall 验证](https://github.com/hnt510-great/openwrt/blob/main/docs/systemd-cgroups-validation.md)
- [DNS nftset 快路径](https://github.com/hnt510-great/openwrt/blob/main/docs/dnsmasq-nftset.md)
- [Full cone NAT](https://github.com/hnt510-great/openwrt/blob/main/docs/fullcone-nat.md)

## 许可与上游

保留上游版权和许可。各软件包源码的许可按其配方 `PKG_LICENSE` 及源码声明执行。
