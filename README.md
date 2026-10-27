# Karing：Claude 与 Gemini 远程分流规则

仅包含公开域名匹配规则、Karing 初始化分流组和使用说明。仓库不包含节点地址、节点密码、私人证书或 GitHub 令牌。

## 固定远程规则地址

https://raw.githubusercontent.com/innvictus/karing-rules/main/ai-domains.ruleset.json

这是 sing-box 源格式 JSON 规则集（version 1），供 **Karing iOS 的自定义分流组 → Rule Set** 使用。它不是节点订阅，不能单独添加到主页的节点配置链接中；规则自身不指定家宽节点。

## 推荐方式：一次导入分流组，以后远程更新域名

1. 先保留或导入自己的两条 HY2 节点：家宽出口和 VPS 直接出口。节点文件由用户在设备本地保存。
2. 下载本仓库的 [diversion_rules_remote.json](https://raw.githubusercontent.com/innvictus/karing-rules/main/diversion_rules_remote.json)。
3. 在 Karing 中进入「设置 → 分流 → 分流规则 → 编辑/铅笔 → 自定义分流组 → 更多/⋮ → 导入」，导入下载的文件，启用两项并保存。不同版本入口名称可能略有差异。
4. 回到「分流规则」，把 **AI-RESIDENTIAL-REMOTE** 从「拦截」改为你的**具体家宽节点**，例如 `US-Residential-HY2`。不要选择自动选择或当前选择，以免跟随主页切换到其他出口。
5. **AI-QUIC-BLOCK-REMOTE** 保持「拦截」，并排在住宅规则前面；最终规则指定 VPS 直接出口，例如 `VPS-HY2`。启用自定义分流组，使用规则/分流模式。
6. 用下面的规则探测示例确认结果，再停用旧的本地域名 AI 分流组，避免旧列表残留影响后续更新；保存并断开、重新连接。

初始化文件将住宅规则暂设为拦截，避免尚未绑定时误走其他出口。**未完成第 4 步前，Claude 和 Gemini 暂时无法访问。** 家宽节点失败时不应配置自动切换到 VPS。

两组都引用同一个远程规则地址。之后更新仓库中的 `ai-domains.ruleset.json`，由 Karing 的远程规则集更新机制拉取；具体更新间隔在应用的分流/规则集设置中查看。初始化 JSON 本身只需导入一次，不是被自动同步的手机设置。

## 也可以完全手动添加

- 住宅组：Rule Set 填上面的固定地址，动作选家宽节点。
- QUIC 组：选择 AND；Rule Set 填同一地址；Network 填 `udp`；Port 填 `443`；动作选拦截。此组排在住宅组前。
- 最终规则选 VPS 节点。家宽上游只转发 TCP 应用请求，因此 QUIC 组让支持回退的 AI 客户端改用 TCP；它不阻断手机到 VPS 的 HY2/UDP 隧道。

## 导入后验证

先在「分流规则探测」确认规则集下载成功。TCP/443 示例：

| 目标 | 预期 |
|---|---|
| claude.ai | 家宽节点 |
| api.anthropic.com | 家宽节点 |
| bridge.claudeusercontent.com | 家宽节点 |
| gemini.google.com | 家宽节点 |
| generativelanguage.googleapis.com | 家宽节点 |
| accounts.google.com | 家宽节点 |
| api.ipify.org | VPS 最终规则 |

AI 域名的 UDP/443 应先命中 QUIC 拦截组。随后实际打开 Claude/Gemini，查看 Karing「连接」中登录和模型请求的出站节点。

已检查 JSON 格式和域名列表；是否成功下载、生效，应以手机的规则探测和连接记录为准。GitHub Raw 在部分网络下可能无法直接访问，必要时使用 Karing 已有可用代理下载规则。远程文件下载失败不能视为家宽路由已生效；确认成功后再停用旧规则。

## 覆盖范围

包括 Claude 的主站、API、登录、内容/Artifacts、MCP 内容域名，以及 Gemini、AI Studio 和原有 Google AI 相关服务。部分共用字体、JS CDN、验证及 Google 静态资源域名也被纳入：其他网站访问这些同域资源时同样会命中住宅规则。

这是一份域名列表；不能识别 Claude 页面任意外链、第三方 MCP 或自建模型网关。服务新增域名时需维护列表。国内直连和其他无关规则由设备现有配置决定。

## 格式与域名来源

- [Karing 官方远程 Rule Set 说明](https://karing.app/en/app-manual/diversion-rule-edit)
- [Karing 官方自定义分流说明](https://karing.app/en/tutorial/custom-diversion)
- [sing-box 源格式规则集](https://sing-box.sagernet.org/configuration/rule-set/source-format/)
- [Claude 官方网络域名要求](https://code.claude.com/docs/en/desktop#network-access-requirements)
- [v2fly/domain-list-community：Anthropic](https://github.com/v2fly/domain-list-community/blob/master/data/anthropic)
- [v2fly/domain-list-community：Google DeepMind](https://github.com/v2fly/domain-list-community/blob/master/data/google-deepmind)
