# glm-codex-model-catalog

给 Codex CLI / Desktop 使用的 GLM 模型目录。一份 `models.json` 包含 `glm-5.3`、`glm-5-turbo` 和 `glm-5.3-flash`。

参考 [grok-codex-model-catalog](https://github.com/shubolab/grok-codex-model-catalog) 的仓库组织方式，模型字段以[智谱官方 Codex 示例](https://docs.bigmodel.cn/cn/coding-plan/tool/codex)为基础。

## 模型列表

| 模型 | 上下文窗口（tokens） | 输入声明 | 推理档位 | 默认推理 | 排序 |
| --- | ---: | --- | --- | --- | ---: |
| `glm-5.3` | 1,048,576 | 文本 | `low` / `high` / `max` | `max` | 0 |
| `glm-5-turbo` | 204,800 | 文本 | 官方示例为空列表，原样保留 | `max` | 1 |
| `glm-5.3-flash` | 1,048,576 | 文本、图片 | `low` / `high` / `max` | `max` | 2 |

`glm-5.3` 和 `glm-5-turbo` 保留智谱官方示例中的字段和值，包括 Turbo 的空推理档位列表。

Flash 条目根据官方模型说明适配：官方标注 1M 上下文、原生图片输入、文本参数与 GLM-5.3 一致，并推荐 `reasoning_effort: max`。因此沿用 GLM-5.3 的上下文数值和三个推理档位，输入声明增加 `image`。其余工具字段沿用官方 Codex 模板；这部分属于目录适配，并非智谱已发布的 Flash 专属 Codex 示例。

## 文件

| 文件 | 用途 |
| --- | --- |
| `models.json` | 三个模型的完整 Codex 目录 |
| [`config.example.toml`](config.example.toml) | 智谱国内 Coding Plan 配置示例，默认使用 `glm-5.3-flash` |
| `README.md` | 安装方法、字段来源和验证边界 |
| `LICENSE` | MIT 许可证 |

## 使用方法

macOS / Linux 可以直接用 `curl` 下载两个文件：

```bash
mkdir -p "$HOME/.codex"

curl -fSL \
  https://raw.githubusercontent.com/shubolab/glm-codex-model-catalog/main/models.json \
  -o "$HOME/.codex/models.json"

curl -fSL \
  https://raw.githubusercontent.com/shubolab/glm-codex-model-catalog/main/config.example.toml \
  -o "$HOME/.codex/config.example.toml"
```

以上命令会写入或更新 `~/.codex/models.json` 和 `~/.codex/config.example.toml`，**不会写入或覆盖已有的 `~/.codex/config.toml`**。已有模型目录时，请先备份并检查是否需要保留其他模型。下载后，首次配置可以将示例复制为 `config.toml`；已有配置请按下方说明手动合并。

1. 下载本仓库的 [`models.json`](models.json)，保存为 `~/.codex/models.json`。已有模型目录时，先备份并检查是否需要保留其中其他模型。
2. 下载 [`config.example.toml`](config.example.toml)。首次配置时，将它复制为 `~/.codex/config.toml`；已有配置时，备份后合并对应字段。示例内容如下：

```toml
model_provider = "ZAI"
model = "glm-5.3-flash"
model_reasoning_effort = "max"
model_catalog_json = "~/.codex/models.json"

[model_providers.ZAI]
name = "ZAI"
base_url = "https://open.bigmodel.cn/api/v1"
wire_api = "responses"
experimental_bearer_token = "<替换为你的智谱 Coding Plan API Key>"
```

`model_provider`、`model`、`model_reasoning_effort` 和 `model_catalog_json` 是顶层字段，合并配置时放在 `[model_providers.ZAI]` 等表头之前。`model_provider = "ZAI"` 对应下方的 `[model_providers.ZAI]`。

如果模型目录保存在其他位置，请修改 `model_catalog_json` 为实际的绝对路径。`model` 可以切换为表中的任意一个 slug。

Windows 路径示例：

```toml
model_catalog_json = 'C:\Users\<you>\.codex\models.json'
```

3. 在自己的 `config.toml` 中，将 API Key 占位符替换为智谱国内 **Coding Plan** 的实际 Key。个人版在“个人编程套餐 → 套餐概览”创建，团队版在“团队编程套餐 → 我的套餐”创建；它们与模型标准 API 的 Key 分开管理。示例文件中只保留占位符。

本示例默认使用下表中的智谱中国站。若已有其他可用 provider，按需保留其配置；若使用 Z.ai 国际站，请按对应[官方说明](https://docs.z.ai/devpack/tool/codex)配置端点和鉴权。

| 服务 | 官方 Codex Base URL | `wire_api` |
| --- | --- | --- |
| 智谱中国站 | `https://open.bigmodel.cn/api/v1` | `responses` |
| Z.ai 国际站 | `https://api.z.ai/api/v1` | `responses` |

选择与自己账号和套餐对应的服务，具体鉴权以对应官方文档为准。

4. 重启 Codex CLI / Desktop，使启动时加载的模型目录生效。

## 本地校验

检查 JSON 格式：

```bash
python3 -m json.tool models.json > /dev/null
```

检查 Codex 能否解析模型目录：

```bash
codex debug models -c 'model_catalog_json="/absolute/path/to/models.json"'
```

输出应包含这三个模型的 slug。校验记录：2026-10-22，使用 `codex-cli 0.162.1` 成功加载本目录，输出包含全部三个模型。

这项检查只验证 Codex 目录解析。尚未实际请求这三个模型，Responses 端点的可用性、工具调用、推理档位和图片输入仍需使用自己的账号验证。`input_modalities`、工具字段等是客户端能力声明，实际支持取决于服务端。

## 配置说明

- `priority` 为 0、1、2，用于模型列表排序。
- `base_instructions` 保留官方示例中的空字符串。
- `effective_context_window_percent` 保留为 95。
- `truncation_policy.limit` 为工具输出截断参数，保留为 10000；它与上下文窗口、模型最大输出长度是不同字段。
- Flash 官方支持视频、文件等输入；本目录仅按 Codex 使用场景声明文本和图片。
- 本仓库提供模型目录和配置示例，API Key 请保存在自己的本机配置或环境变量中。

## 参考

- [智谱：Codex 接入说明及模型目录示例](https://docs.bigmodel.cn/cn/coding-plan/tool/codex)
- [智谱：GLM-5.3-Flash / FlashX 模型说明](https://docs.bigmodel.cn/cn/guide/models/vlm/glm-5.3-flash)
- [Z.ai：Codex 接入说明](https://docs.z.ai/devpack/tool/codex)
- [OpenAI：Codex 配置参考](https://developers.openai.com/codex/config-reference)
- [参考仓库：grok-codex-model-catalog](https://github.com/shubolab/grok-codex-model-catalog)

## License

MIT
