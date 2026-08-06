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
