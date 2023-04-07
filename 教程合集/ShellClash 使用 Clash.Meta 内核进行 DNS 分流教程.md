# 解释：
DNS 分流简单来说就是**指定国内域名走阿里或腾讯 DNS**，主要是这个配置：
```
nameserver-policy:
  "geosite:cn": [https://dns.alidns.com/dns-query, https://doh.pub/dns-query]
```
---
# 一、 方法
注：
- 1. 若更改过 DNS 监听端口，请进入 [ShellClash](https://github.com/juewuy/ShellClash) 配置->7->8->4，更改为默认的 1053
- 2. 搭配 [AdGuardHome](https://github.com/AdguardTeam/AdGuardHome) 时不要使用该方法

## 1. 安装并升级内核
安装 [Clash.Meta](https://github.com/MetaCubeX/Clash.Meta) 内核并升级到 v1.14.2+版本，方法请看[安装 Clash.Meta 内核](https://github.com/DustinWin/Clash-Tutorials/blob/main/%E6%95%99%E7%A8%8B%E5%90%88%E9%9B%86/ShellClash%20%E5%92%8C%20AdGuardHome%20%E5%BF%AB%E9%80%9F%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B.md#%E4%BA%8C-%E5%AE%89%E8%A3%85-clashmeta-%E5%86%85%E6%A0%B8)
## 2. 导入 user.yaml 文件
将 user.yaml 文件移动到 ShellClash 安装目录（如 */data/clash*）  
或者使用快速导入方法（使用此方法可略过第“三”步）：
- 注：若更改过 DNS 监听端口，请进入 ShellClash 配置->7->8->4，更改为默认的 1053

① 使用 fake-ip 模式，连接 SSH 后执行如下命令：
```
curl -o $clashdir/user.yaml -L https://cdn.jsdelivr.net/gh/DustinWin/Clash-Tutorials@main/DNS-Bypass/fake-ip-mode/user.yaml && $clashdir/start.sh restart
```
② 使用 redir-host 模式，连接 SSH 后执行如下命令：
```
curl -o $clashdir/user.yaml -L https://cdn.jsdelivr.net/gh/DustinWin/Clash-Tutorials@main/DNS-Bypass/redir-host-mode/user.yaml && $clashdir/start.sh restart
```
## 3. 重启 Clash 服务
连接 SSH 后，执行如下命令：
```
$clashdir/start.sh restart
```
# 二、 诀窍
若 ShellClash 规则选择的是[白名单模式](https://cdn.jsdelivr.net/gh/DustinWin/Clash-Tutorials@main/Rule-Templates/template_whitelist.yaml)，需要将走直连的所有域名都设置为走国内 DNS 解析，比如我的白名单模式如下：
```
rules:
  - GEOSITE,category-ads-all,⛔️ 广告域名
  - DOMAIN-SUFFIX,googleapis.cn,🚀 节点选择
  - DOMAIN-KEYWORD,test-ipv6,🌐 IPv6 测试
  - GEOSITE,ookla-speedtest,📈 网络测速
  - GEOSITE,private,🏠 私有网络
  - GEOSITE,microsoft@cn,Ⓜ️ 微软中国
  - GEOSITE,apple-cn,🍎 Apple 中国
  - GEOSITE,google-cn,🗽 Google 中国
  - GEOSITE,category-games@cn,🎮 国区游戏
  - GEOSITE,geolocation-!cn,🪜 国外域名
  - GEOSITE,cn,🚄 国内域名
  - GEOIP,telegram,✈️ Telegram IP 地址,no-resolve
  - GEOIP,private,🏠 私有网络,no-resolve
  - GEOIP,cn,🀄 国内 IP 地址
  - MATCH,🐟 漏网之鱼
```
那么需要增加 user.yaml 中的内容：
- 注：兼容 geosite 整合编写功能需要将 Clash.Meta 内核升级到 v1.14.3+版本

```
nameserver-policy:
  "geosite:ookla-speedtest,test-ipv6": [https://doh.pub/dns-query, https://dns.alidns.com/dns-query]
  "geosite:microsoft@cn,apple-cn,google-cn,category-games@cn": [https://doh.pub/dns-query, https://dns.alidns.com/dns-query]
  "geosite:cn,private": [https://doh.pub/dns-query, https://dns.alidns.com/dns-query]
```
