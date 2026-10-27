# 百度网盘 Loon 去广告插件

根据手机百度网盘抓包编写的定向规则。当前样本共 6 条请求，仅覆盖其中已识别的广告接口，不保证去除开屏、信息流等全部广告；尚未在真机 Loon 上验证。

## 安装

在 Loon 的「插件」中添加下面的订阅地址：

```text
https://raw.githubusercontent.com/Liu-rz/loon-bdy/main/BaiduNetdisk.plugin
```

启用插件、URL 重写和 MITM，并安装、信任 Loon 的 CA 证书。插件仅为三个目标域名声明 MITM，不需要通配网盘域名。使用兼容的传统重写语法；JSON 响应替换要求 Loon Build 729 或更新版本。

更新后退出并重新打开百度网盘再验证；投屏配置若有本地缓存，可能需要等配置重新获取。若广告仍出现，可对出现广告的操作补充抓包，以继续定位。

## 处理范围

| 接口 | 处理 | 抓包依据 |
| --- | --- | --- |
| `sdk.1rtb.net/sdk/req_ad` | 拒绝请求 | 路径和参数标明广告请求，响应为二进制内容，不猜测其无广告响应格式 |
| `sdklog.zhangyuyidong.cn/sdk/lurl` | 拒绝请求 | 参数包含竞价价格、广告位、竞价失败原因等 |
| `conf.hpplay.cn/sender/conf` | 将 `data.csj_ad_switch.enabled` 改为数字 `0` | 原始 JSON 中该字段为数字 `1`，其他投屏配置保持原样 |

规则限定完整主机与接口路径，允许查询参数和显式端口，不整域封禁。插件按请求地址匹配；其他 App 若调用相同接口也会受到影响。

抓包中的 `pop.pcsnetwork.netdisk.baidu.com/pop/probe_v6_addr` 为连接探测，`180.76.76.200/v4/resolve` 为解析请求，均不处理。没有对会员、下载权限、速度或账号字段做修改。

## 验证与限制

- 离线核对：原始样本中 2 条广告请求命中拒绝规则、1 条投屏配置命中响应规则，其余 3 条不命中。
- 原始投屏配置确实存在目标字段，且其类型为数字；规则只替换这个字段。
- 拒绝广告请求后 App 是否重试、是否存在广告占位或等待时间，需要真机确认。若投屏异常，可先关闭插件排查。
- HTTPS 证书固定或未信任 MITM 证书时，相关规则可能无法正常执行。

原始 HAR 未提交：其中可能含有令牌、设备标识、签名等敏感数据。仓库仅保留去除敏感参数后的接口信息。

语法依据：[Loon 重写文档](https://nsloon.app/en/docs/Rewrite/)、[插件格式](https://nsloon.uk/tutorial/advanced/Plugin_Format.html)。
