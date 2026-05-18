# Server & Proxy Manager

> 统一版 · System Guardian v10.1.0 + Proxy Manager v13.2

一键部署和管理代理服务、防火墙、流量转发、系统安全的综合运维脚本，适用于 Debian/Ubuntu 系统。

---

## 功能概览

| 分类 | 功能 |
|------|------|
| 系统管理 | 一键初始化、XanMod 内核、BBR v3、网络参数优化 |
| 安全防护 | Fail2Ban、SSH 端口修改、iptables 防火墙、CN IP 封禁 |
| 代理服务 | Snell v5、Shadowsocks-Rust、SOCKS5 |
| 流量转发 | Realm 智能转发、链式中转、存活检测 |
| 监控告警 | Telegram 推送、SSH 登录通知、Nezha 探针丢包监测（TCPing）、流量配额告警 |

---

## 使用方法

### 一键启动

```bash
bash <(curl -sL https://raw.githubusercontent.com/Bud668/ipt_prxy/main/ipt_prxy.sh)
```

### 本地运行

```bash
# 下载脚本
wget -O ipt_prxy.sh https://raw.githubusercontent.com/Bud668/ipt_prxy/main/ipt_prxy.sh
chmod +x ipt_prxy.sh

# 以 root 运行
sudo bash ipt_prxy.sh
```

### 系统要求

- OS：Debian 10+ / Ubuntu 20.04+
- 权限：root
- 依赖：`curl`、`jq`（首次初始化时自动安装）

---

## 主菜单选项详解

```
================================================================
    Server & Proxy Manager  & Deploy Tool  v13.2
================================================================
:: 服务器信息 ::   :: 服务状态 ::   :: 防火墙 & 内核 ::
================================================================
 [ 系统管理 ]
 ★ 1. 一键初始化              4. 切换测试模式
    2. Fail2Ban               5. 防火墙规则
    3. TG 推送配置             6. 系统维护
----------------------------------------------------------------
 [ 代理服务 ]                  [ 规则与转发 ]
  7. 安装/管理 Snell           11. 添加转发规则
  8. 安装/管理 Realm           12. 重启 Realm
  9. 安装/管理 SS-Rust         13. 检测并删除失效规则
 10. 安装/管理 SOCKS5          14. 流量配额与到期管理
                               15. 查看运行状态日志
----------------------------------------------------------------
 [ 进阶控制 ]
 16. 启停服务
 17. 更新服务 (Snell/SS/Realm/SOCKS5)
 18. 卸载服务 (Snell/SS/Realm/SOCKS5)
================================================================
```

---

## 功能详解

### 1. 一键初始化

按顺序执行以下步骤，完成服务器基础配置：

1. **禁用 IPv6** — 写入 sysctl 配置永久生效
2. **系统更新 & 依赖安装** — apt upgrade + 安装 curl、jq、iptables、fail2ban、iperf3 等所有必要工具
3. **服务器名称** — 设置 Telegram 通知中显示的节点名称
4. **XanMod 内核** — 可选安装带 BBR v3 的高性能内核
5. **网络参数优化** — 根据带宽动态计算 rmem/wmem/BBR 参数，写入 sysctl
6. **防火墙** — 初始化 iptables DROP 策略，放行 SSH/HTTP/HTTPS 及代理端口
7. **TCPing 监控** — 部署 TCPing 监控服务，专用于监测 Nezha 探针入口的丢包情况，异常时通过 Telegram 告警

### 2. Fail2Ban & SSH 安全（选项 2）

- 修改 SSH 监听端口
- 配置 Fail2Ban 防暴力破解
- 设置 IP 白名单
- 部署 **SSH 登录 Telegram 通知**（有人登录时实时推送到 TG）

### 3. Telegram 推送配置（选项 3）

统一管理所有 TG 告警渠道：

| 子功能 | 说明 |
|--------|------|
| Bot Token / Chat ID | 配置机器人和接收频道 |
| SSH 登录通知 | 实时推送 SSH 登录事件（含 IP、用户名、时间） |
| 中转监控告警 | Realm 转发规则失效时自动告警 |
| 每日状态报告 | 定时汇报服务状态摘要 |
| 流量配额告警 | 用户超量或到期时 TG 通知 |

### 4. 测试模式（选项 4）

临时开放 ICMP（Ping）和 iperf3（端口 5201），方便测速：

- 开启后 **2 小时自动关闭**（通过 `at` 调度）
- 菜单实时显示测速命令，对端直接复制执行
- 再次按 4 可立即关闭

### 5. 防火墙管理（选项 5）

- 查看当前 iptables 规则
- 开放 / 封闭指定端口（TCP/UDP）
- 启用 **CN IP 封禁**（通过 ipset 加载 China IP 段，阻断大陆访问）
- 切换防火墙策略（DROP 安全模式 / ACCEPT 开放模式）
- 持久化规则（`iptables-persistent`）

### 6. 系统维护（选项 6）

- 安装 / 切换 XanMod 内核（BBR v3）
- 查看 / 修改 BBR 拥塞控制参数
- 系统清理（清除旧内核、apt 缓存）
- 调整 TCP 缓冲区大小（按实际带宽动态计算）
- TCPing 监控管理（安装、修改端口、卸载）— 专用于 Nezha 探针丢包检测

### 7. Snell 代理（选项 7）

[Snell v5](https://github.com/nicholasgasior/snell-server) 高性能代理：

- 一键安装最新版（自动检测 GitHub Release）
- 支持**多实例**（多端口并行运行，每实例独立 systemd 服务）
- 随机生成 PSK 或手动指定
- 菜单直接输出 Surge/Clash 格式配置行
- 支持一键升级（保留配置）

Surge 配置示例（菜单自动生成）：
```
🇯🇵jp_xxx = snell, 1.2.3.4, 12345, psk="abc123", version=5, reuse=true, tfo=true
```

### 8. Realm 流量转发（选项 8）

[Realm](https://github.com/zhboner/realm) 高性能端口转发：

- 安装 / 管理转发规则
- 支持**链式转发**（本机 → 中转 → 落地，自动带 Snell PSK 元数据）
- 菜单直接输出转发后的 Snell 配置行（显示目标国旗）
- 选项 13：**自动检测失效规则**（目标不可达时删除并 TG 告警）
- 选项 12：一键重启 Realm 服务

### 9. Shadowsocks-Rust（选项 9）

[shadowsocks-rust](https://github.com/shadowsocks/shadowsocks-rust) 代理服务：

- 一键安装，自动选择最新版
- 支持多端口多密码
- 内置 **ACL 域名黑名单**（阻断特定域名访问，如屏蔽国内服务）
- 可叠加 **CN IP 封禁**（配合 iptables ipset）

Surge 配置示例（菜单自动生成）：
```
🇯🇵Tokyo-JP-ss-8388 = ss, 1.2.3.4, 8388, encrypt-method=aes-128-gcm, password="xxx", tfo=true, udp-relay=true
```

### 10. SOCKS5 代理（选项 10）

基于 dante 的 SOCKS5 代理：

- 一键安装 dante-server
- 自动生成随机用户名/密码
- 菜单显示 `socks5://user:pass@ip:port` 连接串

### 11. 添加转发规则（选项 11）

高级 Realm 转发规则向导：

1. 输入目标落地机 IP 和 Snell 端口
2. 输入落地机 PSK（脚本自动存入元数据）
3. 自动分配本地随机端口，写入 Realm 配置
4. 支持链式：可将上游也是中转机的节点再次包装

### 14. 流量配额管理（选项 14）

基于 iptables 计数的流量配额系统：

- 为每个 Realm 转发端口设置月度流量上限
- 超量后自动封禁端口（DROP 规则）
- 每日定时通过 TG 推送流量使用报告
- 支持到期日管理（自动到期封禁）

### 15. 查看运行状态日志（选项 15）

- 查看 Snell / Shadowsocks-Rust / Realm 最近 50 条日志
- 实时跟踪日志（`journalctl -f`，按任意键退出）

### 17. 更新服务（选项 17）

- 检测 Snell、Shadowsocks-Rust、Realm、SOCKS5 是否有新版本
- 有更新时主菜单显示 `[有更新可用]` 提示
- 一键更新，保留配置不中断

### 18. 卸载服务（选项 18）

- 选择性卸载各组件
- 自动停止 systemd 服务、删除二进制和配置

---

## 守护进程模式（CLI 参数）

脚本支持以非交互方式运行特定子任务，用于 systemd 定时器：

```bash
# 运行中转监控守护进程（检测 Realm 转发存活）
bash ipt_prxy.sh daemon

# 发送每日状态报告到 Telegram
bash ipt_prxy.sh daily

# 检查所有端口流量配额
bash ipt_prxy.sh quota-check

# 发送配额每日报告
bash ipt_prxy.sh quota-daily
```

---

## 文件路径说明

| 路径 | 用途 |
|------|------|
| `/opt/proxy-manager/` | 脚本工作目录（缓存、配置） |
| `/etc/snell/` | Snell 配置目录 |
| `/etc/shadowsocks-rust/` | Shadowsocks-Rust 配置 |
| `/etc/realm/` | Realm 配置及元数据 |
| `/etc/socks5-monitor/` | SOCKS5 配置 |
| `/var/log/proxy-manager.log` | 脚本运行日志 |
| `/etc/ssh-tg-monitor.conf` | TG 推送配置 |
| `/etc/tcping-monitor.conf` | TCPing 监控配置 |

---

## 注意事项

- 脚本需要 **root 权限**运行
- 一键初始化安装 XanMod 内核后需**重启**才能启用 BBR v3
- 防火墙初始化后默认策略为 DROP，修改 SSH 端口前请确认新端口已放行
- 流量配额系统依赖 `iptables` 计数，重启后计数器清零（月度统计通过日志累加）
- 版本更新检测依赖 GitHub API，触发限流时自动使用保底版本

---

## License

MIT
