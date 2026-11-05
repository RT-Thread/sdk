# Env 插件索引

Env 的在线插件市场默认读取本目录；插件包可发布在 GitHub Release，无需部署市场服务器。显式设置 `ENV_PLUGIN_MARKET_URL` 或 `rt-env plugin market set <url>` 时仍优先使用 HTTP 市场。

## 目录结构

```text
env-plugins/
  index.json
  org.example.demo/
    index.json
```

根 `index.json` 参考 RT-Thread Studio sdk-index 的目录索引格式：

```json
{
    "name": "Env Plugin Index",
    "version": 1,
    "index": ["org.example.demo"]
}
```

`index` 的每个字符串是本目录下的直接子目录名称，每个子目录必须包含 `index.json`。不支持绝对路径、`..`、路径分隔符和指向目录外的符号链接。

首个插件是 `org.rt-thread.kconfig`（RT-Kconfig 配置器）0.1.34；索引中的下载记录对应本仓库 GitHub Release 的真实 `.epack` 资产，不包含占位下载记录。

## 插件记录

下面仅说明格式，发布前必须用真实的插件 ID、版本、Release 地址和完整 SHA-256 替换示例值：

```json
{
    "id": "org.example.demo",
    "name": "Demo",
    "description": "插件功能简介",
    "capabilities": ["commands", "webui"],
    "updated_at": "2026-11-05T00:00:00Z",
    "status": "published",
    "versions": [
        {
            "version": "1.0.0",
            "status": "published",
            "artifacts": [
                {
                    "filename": "org.example.demo-1.0.0-py3-none-any.epack",
                    "url": "https://github.com/OWNER/REPO/releases/download/v1.0.0/org.example.demo-1.0.0-py3-none-any.epack",
                    "sha256": "REPLACE_WITH_THE_ACTUAL_64_CHARACTER_LOWERCASE_SHA256",
                    "compatibility": {
                        "env": ">=2.0.2,<3.0.0",
                        "python": ">=3.7.0,<4.0.0",
                        "implementations": ["cpython"],
                        "abis": ["py3"],
                        "platforms": ["any"],
                        "architectures": ["any"]
                    }
                }
            ]
        }
    ]
}
```

- `id`、`name`、非空 `versions` 必填。同一根索引的插件 ID 及同一插件内的版本不可重复。
- 每个版本必须有非空 `artifacts`。同一版本可列出不同平台、架构或 Python ABI 的制品。
- `filename`、HTTPS `url`、小写十六进制 `sha256`、`compatibility` 必填；可选 `size_bytes` 是包的实际正整数字节数，最大 256 MiB。
- `compatibility` 的六个字段必填，应与包内 manifest 一致；平台使用 `linux`、`windows`、`darwin` 或 `any`，架构使用 `x86_64`、`aarch64` 等规范名称或 `any`。
- `status` 省略时为 `published`；将插件或版本标记为 `yanked` 可撤回，撤回记录不会被安装。
- Env 按语义版本选择最高兼容且未撤回的版本；无需依靠 JSON 中的顺序。
- `updated_at` 可选，建议统一用 UTC ISO 8601 格式。去服务器化来源不统计下载次数，市场显示为 0。

## 发布和更新

1. 用 `epack build` 构建 `.epack`，用 `epack inspect` 检查包。
2. 将包上传至 GitHub Release；Release 地址必须可公开下载，不要在索引中放置令牌或凭证。
3. 在 Linux 上运行 `sha256sum <package.epack>`，或在 PowerShell 中运行 `Get-FileHash <package.epack> -Algorithm SHA256`；PowerShell 的结果需转为小写后填入索引。
4. 新建或更新插件目录的 `index.json`，新插件还需加入根 `index` 列表，提交 SDK 仓库。
5. 用户通过已有 SDK 仓库更新流程获取索引，或在 `${ENV_ROOT}/packages/sdk` 手动运行 `git pull`；随后刷新 WebUI 的在线目录。WebUI 刷新只重读本地文件，不隐式运行 Git 或获取远端索引。

`epack push` 针对 HTTP 市场，不会修改本 Git 索引；去服务器化发布应按上述步骤维护。

## 安全和使用

- 下载限制为 HTTPS；同源重定向允许，GitHub Release 仅可跨源跳转到明确允许的 GitHub 资产域名。HTTP 市场原来的同源限制不变。
- 下载必须匹配索引 SHA-256 和可选大小，之后继续执行包完整性、插件 ID/版本、兼容性和未签名包确认检查。摘要不是发布者签名，应审核索引变更并保护 SDK 仓库。
- 沿用 Env 网络设置中的代理和超时。索引可离线浏览，安装仍需要下载网络。
- `rt-env plugin market` 可查看当前索引路径；`rt-env plugin market clear` 清除文件中的 HTTP 市场配置后恢复 SDK 来源（环境变量仍具有最高优先级）。
- 市场来源在 WebUI 启动时选定：清除市场地址或首次添加根索引后需要重启 WebUI；已启用 SDK 来源后，索引内容的更新只需刷新目录。
