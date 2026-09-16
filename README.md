# Clash / OpenClash 配置维护说明

这个目录维护 Clash/Mihomo/OpenClash 配置模板，以及一份带真实订阅链接的本地私有配置。

## 目录约定

- 项目根目录：只放可提交、可公开的配置模板。
- `with_subscription/`：只放本地私有配置，可以包含真实机场订阅链接。
- `.gitignore` 已忽略 `with_subscription/`，不要移除这条规则。

如果用户口头说 `with-subscription`，通常也是指当前实际目录 `with_subscription/`。

## 订阅链接安全规则

真实机场订阅链接只能出现在：

```text
with_subscription/
```

订阅链接绝不能出现在项目根目录下的任何 YAML、README、脚本、提交说明或临时文件里。

根目录 YAML 中的 `proxy-providers.*.url` 必须保持为占位说明，不能填真实订阅。例如：

```yaml
url: "机场订阅填到这里，两端引号不要去掉，不要填写到下方链接里去！！"
```

修改、复制、对比文件时，必须避免把 `with_subscription/` 里的真实 URL 写回根目录。

## 同版本同步规则

修改规则时，项目根目录和 `with_subscription/` 里的同版本 YAML 应保持内容规则同步，区别只应是订阅链接和 provider 名称/前缀等私有信息。

常见对应关系：

```text
configdns.yaml              <-> with_subscription/configdns.yaml
configdns_v2.yaml           <-> with_subscription/configdns_v2.yaml
configdns_v3.yaml           <-> with_subscription/configdns_v3.yaml
configdns_v4.yaml           <-> with_subscription/configdns_v4.yaml
configdns_v5.yaml           <-> with_subscription/configdns_v5.yaml
configdns_v6.yaml           <-> with_subscription/configdns_v6.yaml
configdns_v7.yaml           <-> with_subscription/configdns_v7.yaml
configdns_v8.yaml           <-> with_subscription/configdns_v8.yaml
configdns_v8_Claude.yaml    <-> with_subscription/configdns_v8_Claude.yaml
```

如果某个版本只存在于一侧，不要擅自创建或删除另一侧文件，除非用户明确要求。

## 修改 rules 的基本流程

修改分流规则时，通常需要同时检查三处：

1. `proxy-groups`
2. `rules`
3. `rule-providers`

如果新增一个服务分组，例如 Claude，需要：

- 在 `proxy-groups` 新增策略组。
- 在 `rules` 中添加对应 `RULE-SET`，并放在更泛化规则之前。
- 在 `rule-providers` 中添加对应规则集来源。

规则匹配是从上到下，命中即停止。因此更具体的规则要放在更泛化规则前面。例如：

```yaml
- RULE-SET,claude_domain,🧠 Claude
- RULE-SET,ai,🤖 ChatGPT
```

不要把专用服务规则放在 `ai`、`geolocation-!cn`、`MATCH` 这类泛规则之后。

### 强制代理域名列表

无法被现有规则识别、但必须通过代理访问的域名统一维护在：

```text
force_proxy.list
```

该文件使用 Clash classical text 格式。需要同时匹配主域名及其所有子域名时，使用：

```text
DOMAIN-SUFFIX,example.com
```

`configdns_v6.yaml` 及其私有对应版本通过 `force_proxy` rule-provider 加载该列表，并交给 `🚀 默认代理`。

### 游戏下载补充列表

第三方游戏下载规则未覆盖、但需要交给 `🎮 游戏下载` 策略的下载清单接口和 CDN 域名统一维护在：

```text
game_download.list
```

该文件使用 Clash classical text 格式。`configdns_v8.yaml` 及其私有对应版本通过
`game_download_custom` rule-provider 加载该列表；其规则应放在 `geolocation-!cn` 等泛规则之前。

### Claude 静态 ISP 分支

`configdns_v8_Claude.yaml` 是从 V8 派生的 Claude 静态 ISP 配置分支，原 V8 保持不变。
该分支使用三个策略组：

- `🧠 Claude`：选择静态 ISP、普通机场策略或直连。
- `🛫 Claude 中转`：选择连接静态 ISP 时使用的机场节点，或选择 `➡️ 直连落地`绕过机场。
- `🏠 Claude 落地`：在三个静态 ISP 出口之间切换，默认顺序为 ISP-2、ISP-1、ISP-3。

`🛫 Claude 中转`首次使用 `REJECT`，必须手动选择机场节点或 `➡️ 直连落地`；选择结果会由
`store-selected` 保存。Claude/Anthropic 核心域名和 IP 检测站补充规则维护在：

```text
claude_custom.list
```

第四机场的真实订阅和静态 ISP 的服务器、端口、用户名、密码只能写入
`with_subscription/configdns_v8_Claude.yaml`，公开版本必须使用占位值。

具体的节点选择顺序、常用路径和故障排查见
[Claude 节点设置说明](CLAUDE节点设置说明.md)。

## 修改 filter 的注意事项

地区节点组依赖节点名正则过滤。修改时要避免过宽的单字匹配。

例如新加坡不应只用单字 `坡`，因为会误匹配：

```text
马来西亚-吉隆坡
```

更稳的写法是使用明确关键词：

```yaml
新加坡|狮城|SG|Singapore
```

如果新增 `其他节点` 这类反向过滤组，要确认它不会把已有港/日/新/美等主地区重复收进去。

## 版本变更记录

### V8_Claude（2026-09-16）

- 从 V8 派生独立配置，不修改原 V8。
- 新增第四机场、三个静态 ISP，以及 Claude 中转和落地选择。
- 支持机场加静态 ISP、静态 ISP 直连、普通机场直出和完全直连四种路径。
- 新增 `claude_custom.list`，补充 Claude 核心域名和 IP 检测站。

### V8（2026-09-13）

- 基于 V7，新增 `game_download.list`，覆盖 Epic 的下载清单接口、Fastly、腾讯云、Akamai 和 `epicgamescdn.com` 下载域名。
- V8 公私配置均通过 `game_download_custom` 将该列表交给 `🎮 游戏下载` 策略，并置于 `geolocation-!cn` 等泛规则之前。
- Epic 下载清单通过中国出口请求，可获得腾讯云国内分发点；登录和账号认证域名仍按原规则走代理。

### V7（2026-08-23）

- 基于 V6，分流规则和 DNS 逻辑保持不变。
- `keep-alive-idle` 保持为 600 秒：连接连续空闲 600 秒后才开始 TCP Keepalive 探测；期间有流量会重新计算空闲时间。
- `keep-alive-interval` 从 15 秒调整为 1800 秒，降低 OpenClash/Mihomo 接管 iOS 长连接后，频繁探测造成的待机唤醒和耗电。
- 仍保留 TCP Keepalive；取舍是对无响应连接进行后续探测的间隔变长，纯空闲死连接可能更晚被清理，正常业务流量的 TCP 重传与超时不受该参数替代。

## 修改前后的检查清单

提交或交付前至少检查：

- 根目录 YAML 没有真实订阅链接。
- `with_subscription/` 仍被 `.gitignore` 忽略。
- 同版本根目录 YAML 和 `with_subscription/` YAML 的 rules/proxy-groups/rule-providers 逻辑同步。
- 新增的策略组名称和 `rules` 中引用名称完全一致。
- 新增的 `RULE-SET` 有对应 `rule-providers` 定义。
- 更具体的规则位于更泛化规则之前。

可以用：

```powershell
git status --short --ignored
```

确认 `with_subscription/` 显示为 ignored，而不是待提交文件。

## 不要做的事

- 不要把真实订阅链接复制到项目根目录。
- 不要批量删除配置文件或私有目录。
- 不要只改根目录模板而忘记同步同版本私有配置。
- 不要只改私有配置而忘记同步同版本根目录模板。
- 不要随意调整 `proxy-providers.*.url`，除非用户明确要求。
