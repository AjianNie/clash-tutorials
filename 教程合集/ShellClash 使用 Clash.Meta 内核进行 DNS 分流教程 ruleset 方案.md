# 解释：
此方案采用 `RULE-SET` 规则  
DNS 分流简单来说就是**指定国内域名走阿里或腾讯 DNS**，主要是这个配置：
```
nameserver-policy:
  'rule-set:cn': [https://dns.alidns.com/dns-query, https://doh.pub/dns-query]
```
---
# 一、 方法
注：
- 1. 搭配 [AdGuardHome](https://github.com/AdguardTeam/AdGuardHome) 时不要使用该方法
- 2. 须将 [Clash.Meta 内核](https://github.com/MetaCubeX/Clash.Meta)升级至 v1.14.4+版本才可兼容该方法

## 1. 安装并升级内核
方法请看[安装 Clash.Meta 内核](https://github.com/DustinWin/clash-tutorials/blob/main/%E6%95%99%E7%A8%8B%E5%90%88%E9%9B%86/ShellClash%20%E5%92%8C%20AdGuardHome%20%E5%BF%AB%E9%80%9F%E5%AE%89%E8%A3%85%E6%95%99%E7%A8%8B.md#%E4%BA%8C-%E5%AE%89%E8%A3%85-clashmeta-%E5%86%85%E6%A0%B8)
## 2. 导入 user.yaml 文件
将 user.yaml 文件移动到 ShellClash 安装目录（如 */data/clash*）  
或者使用快速导入方法：  
① 使用 fake-ip 模式不需要使用 DNS 分流，仅配置 default-nameserver 和 nameserver 这两个参数即可  
此处分享一配置，连接 SSH 后执行如下命令：
```
curl -o $clashdir/user.yaml -L https://cdn.jsdelivr.net/gh/DustinWin/clash-tutorials@main/dns-bypass/fake-ip-user.yaml && $clashdir/start.sh restart
```
② 使用 redir-host 模式，连接 SSH 后执行如下命令：
```
curl -o $clashdir/user.yaml -L https://cdn.jsdelivr.net/gh/DustinWin/clash-tutorials@main/dns-bypass/ruleset-mode/redir-host-user.yaml && $clashdir/start.sh restart
```
# 二、 诀窍
## 1. [白名单模式](https://cdn.jsdelivr.net/gh/DustinWin/clash-tutorials@main/rule-templates/ruleset-mode/template_whitelist.yaml)
若 ShellClash 规则选择的是白名单模式，需要将走直连的所有域名都设置为走国内 DNS 解析，比如我的白名单模式如下：
```
rules:
  - RULE-SET,reject,⛔️ 广告域名
  - RULE-SET,private,🏠 私有网络
  - RULE-SET,speedtest,📈 网络测速
  - RULE-SET,microsoft,Ⓜ️ Microsoft 中国
  - RULE-SET,apple,🍎 Apple 中国
  - RULE-SET,google,🗽 Google 中国
  - RULE-SET,games,🎮 国区游戏
  - RULE-SET,proxy,🪜 代理域名
  - RULE-SET,direct,⚡ 直连域名
  - RULE-SET,lancidr,🏠 私有网络,no-resolve
  - RULE-SET,cncidr,🇨🇳 国内 IP
  - RULE-SET,telegramcidr,✈️ Telegram IP
  - MATCH,🐟 漏网之鱼
```
那么需要增加 user.yaml 中的内容：
```
nameserver:
  - tls://dns.google
  - https://dns.cloudflare.com/dns-query
  - https://doh.opendns.com/dns-query

nameserver-policy:
  'rule-set:speedtest,microsoft,apple,google,games': [https://doh.pub/dns-query, https://dns.alidns.com/dns-query]
  'rule-set:cn,private': [https://doh.pub/dns-query, https://dns.alidns.com/dns-query]
```
## 2. [黑名单模式](https://cdn.jsdelivr.net/gh/DustinWin/clash-tutorials@main/rule-templates/ruleset-mode/template_blacklist.yaml)
若 ShellClash 规则选择的是黑名单模式，需要将走代理的所有域名都设置为走国外 DNS 解析，比如我的黑名单模式如下：
```
rules:
  - RULE-SET,reject,⛔️ 广告域名
  - RULE-SET,private,🏠 私有网络
  - RULE-SET,speedtest,📈 网络测速
  - RULE-SET,gfw,🧱 GFWList 域名
  - RULE-SET,telegramcidr,✈️ Telegram IP
  - MATCH,🐟 漏网之鱼
```
那么需要增加 user.yaml 中的内容：
```
nameserver:
  - https://doh.pub/dns-query
  - https://dns.alidns.com/dns-query

nameserver-policy:
  'rule-set:gfw': [tls://dns.google, https://dns.cloudflare.com/dns-query, https://doh.opendns.com/dns-query]
```
