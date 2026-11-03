# CREDENTIALS.md — 你需要准备什么

> 一句话：**只有两个密钥**（模型密钥 + 邮件密钥），其余全部是可编辑的普通字段。
> **密钥永远不要写进节点里**——用 n8n 的 Credentials 面板保存，节点只引用名字。

---

## 一、必须准备（2 个）

| # | 名称 | 在哪拿 | 类型 | 用在哪 |
|---|---|---|---|---|
| 1 | **OpenAI API Key** | https://platform.openai.com/api-keys | n8n 凭证 → **Header Auth** | `01` 和 `02` 的 *Call model* 节点 |
| 2 | **Email API Key** | https://resend.com/api-keys（免费 3000 封/月） | n8n 凭证 → **Header Auth** | `03` 的 *Send reminder email* 节点 |

**Header Auth 凭证怎么填**（两个凭证都一样）：

| 字段 | 值 |
|---|---|
| Name | `Authorization` |
| Value | `Bearer sk-你的密钥` |

> 任何 OpenAI 兼容端点都行：改 `Config (edit me)` 里的 `baseUrl` 和 `model` 即可
> （自建 vLLM / Ollama / DeepSeek / 智谱 / 通义 / 本地模型都行）。

---

## 二、按需要准备（选填，填了才生效）

| 名称 | 类型 | 用在哪 | 不填会怎样 |
|---|---|---|---|
| `deliveryWebhookUrl` | URL | `01` 把合格线索推给你的 CRM / 表格 / Zapier | 走占位地址，分支仍会跑通 |
| `urgentWebhookUrl` | URL | `02` 紧急工单通知（Slack / 邮件 / 短信） | 同上，**但紧急工单不会被真正通知到** |
| `normalWebhookUrl` | URL | `02` 把草稿回复存进你的收件箱 | 同上 |
| `sheetId` | 表格 ID | `03` 读发票表 | 表读不到，会返回一行错误说明 |
| `sheetName` | 文本 | `03` 选工作表 | 默认 `Invoices` |
| `fromEmail` | 邮箱 | `03` 发件人 | 默认 `billing@example.com`（要改成你验证过的发件域） |
| `ownerBcc` | 邮箱 | `03` 每封抄送你自己（**证据链**） | 强烈建议填 |
| `paymentLinkBase` | URL 前缀 | `03` 拼接付款链接 | 默认 `https://pay.example.com/i/` |
| `bizName` | 文本 | 三个 workflow 都用（信里/提示词里显示公司名） | 默认 `Your Business` |

**发票表的列名**（`03` 认这些列名，大小写不敏感，多余列忽略）：

```
invoice_id | client_name | client_email | amount | currency | due_date | reminder_count
```

`due_date` 用 `2026-10-01` 这种格式；`reminder_count` 填已催次数（第一次催填 `0`）。
**表需要"发布到网页"**（Google Sheets：文件 → 分享 → 发布到网页），否则读不到。

---

## 三、默认值一览（都在 `Config (edit me)` 节点里）

| 字段 | 默认值 | 说明 |
|---|---|---|
| `model` | `gpt-4o-mini` | 任何 OpenAI 兼容模型可换 |
| `baseUrl` | `https://api.openai.com/v1/chat/completions` | 换端点就改这里 |
| `useJsonMode` | `true` | 端点不支持 `response_format` 时改成 `false` |
| `qualifyThreshold` | `70` | 线索合格分数线 |
| `reminderAfterDays` | `7` | 逾期几天开始催 |
| `maxReminders` | `3` | 催几次为止 |
| `currency` | `USD` | 发票币种兜底值 |

---

## 四、分享/导出前的清理清单 ⚠️

如果你要把自己的 workflow 导出给别人（或放进你的交付包），**官方要求**先清掉这些：

- [ ] 凭证的 **name 和 id**（节点里的凭证引用可能指向你的账号）
- [ ] 所有 API key / token（凭证本身不会被导出，但节点里硬写的值会）
- [ ] 你自己的邮箱、表格 ID、内部 URL

导出命令：

```bash
n8n export:workflow --id=<id> --separate --output=./workflows
n8n-cli package export --workflow-id=A --workflow-id=B --workflow-id=C \
  --include-variable-values=false --output=shipped-automations.n8np
```

> ⚠️ `--include-variable-values` **默认是 `true`**，会把变量值（含密钥）打进包里——**必须显式写 `false`**。
> ⚠️ 导入时 workflow 默认是**停用态**，要在 README 里提醒对方点一下 **Active**。
