_在最近学习过程中，老是碰到TLS这个名词，为了加深印象，本人决定亲自“拆拆”这个协议。_

## 测试环境
一台默认安装wireshark的kali机子。
打开监听后使用
```
curl -i "https://tls12-strong-cipher-check.xglab.ariba.com"
```
命令访问。获取TLS1.2协议的过程Handshake包。
!(准备工作)[KALI2025.4训练版-2026-07-01-10-28-02.png]