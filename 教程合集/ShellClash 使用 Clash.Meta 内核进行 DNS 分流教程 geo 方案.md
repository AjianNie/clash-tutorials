# 解释：
此方案采用 GEOSITE 和 GEOIP 规则搭配 geosite.dat 和 geoip.dat（或 Country.mmdb） 路由规则文件  
DNS 分流简单来说就是**指定国内域名走阿里或腾讯 DNS**，主要是这个配置：
```
nameserver-policy:
  'geosite:cn': [https://dns.alidns.com/dns-query, https://doh.pub/dns-query]
```
---
# 一、 方法
注：
- 1. 搭配 [AdGuardHome](https://github.com/AdguardTeam/AdGuardHome) 时不要使用该方法
- 2. 须将 [Clash.Meta 内核](https://github.com/MetaCubeX/Clash.Meta)升级至 v1.14.3+版本才可兼容该方法

## 1. 安装并升级内核
方法请看[安装 Clash.Meta 内核](https://github.com/DustinWin/clash-tutorials/blob/main/%E6%95%99%E7%A8%8B%E5%90%88%E9%9B%86/ShellClash%20%E5%92%8C%20AdGuardHome%20%E5%BF%AB%E9%80%9F%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B.md#%E4%BA%8C-%E5%AE%89%E8%A3%85-clashmeta-%E5%86%85%E6%A0%B8)
## 2. 导入 user.yaml 文件
将 user.yaml 文件移动到 ShellClash 的指定目录（如 */data/clash/yamls*）  
或者使用快速导入方法：  
① 使用 fake-ip 模式不需要使用 DNS 分流，仅配置 `default-nameserver` 和 `nameserver` 这两个参数即可  
此处分享一配置，连接 SSH 后执行如下命令：
```
curl -o $clashdir/yamls/user.yaml -L https://cdn.jsdelivr.net/gh/DustinWin/clash-tutorials@main/dns-bypass/fake-ip-user.yaml && $clashdir/start.sh restart
```
② 使用 redir-host 模式，连接 SSH 后执行如下命令：
```
curl -o $clashdir/yamls/user.yaml -L https://cdn.jsdelivr.net/gh/DustinWin/clash-tutorials@main/dns-bypass/geo-mode/redir-host-user.yaml && $clashdir/start.sh restart
```
# 二、 诀窍
## 1. [白名单模式](https://cdn.jsdelivr.net/gh/DustinWin/clash-tutorials@main/rule-templates/geo-mode/template_whitelist.yaml)
若 ShellClash 规则选择的是白名单模式，需要将走直连的所有域名都设置为走国内 DNS 解析，比如我的白名单模式如下：
```
rules:
  - GEOSITE,category-ads-all,⛔️ 广告域名
  - GEOSITE,private,🏠 私有网络
  - GEOSITE,speedtest,📈 网络测速
  - GEOSITE,microsoft@cn,Ⓜ️ Microsoft 中国
  - GEOSITE,apple-cn,🍎 Apple 中国
  - GEOSITE,google-cn,🗽 Google 中国
  - GEOSITE,category-games@cn,🎮 国区游戏
  - GEOSITE,geolocation-!cn,🪜 代理域名
  - GEOSITE,cn,⚡ 直连域名
  - GEOIP,telegram,✈️ Telegram IP
  - GEOIP,private,🏠 私有网络,no-resolve
  - GEOIP,cn,🇨🇳 国内 IP
  - MATCH,🐟 漏网之鱼
```
那么需要增加 user.yaml 中的内容：
```
nameserver:
  - tls://dns.google
  - https://dns.cloudflare.com/dns-query
  - https://doh.opendns.com/dns-query

nameserver-policy:
  'geosite:speedtest,microsoft@cn,apple-cn,google-cn,category-games@cn': [https://doh.pub/dns-query, https://dns.alidns.com/dns-query]
  'geosite:cn,private': [https://doh.pub/dns-query, https://dns.alidns.com/dns-query]
```
## 2. [黑名单模式](https://cdn.jsdelivr.net/gh/DustinWin/clash-tutorials@main/rule-templates/geo-mode/template_blacklist.yaml)
若 ShellClash 规则选择的是黑名单模式，需要将走代理的所有域名都设置为走国外 DNS 解析，比如我的黑名单模式如下：
```
rules:
  - GEOSITE,category-ads-all,⛔️ 广告域名
  - GEOSITE,speedtest,📈 网络测速
  - GEOSITE,gfw,🧱 GFWList 域名
  - GEOIP,telegram,✈️ Telegram IP
  - MATCH,🐟 漏网之鱼
```
那么需要增加 user.yaml 中的内容：
```
nameserver:
  - https://doh.pub/dns-query
  - https://dns.alidns.com/dns-query

nameserver-policy:
  'geosite:gfw': [tls://dns.google, https://dns.cloudflare.com/dns-query, https://doh.opendns.com/dns-query]
```
