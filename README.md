# pharm-lit-cn：药学中文文献检索（知网 + 万方）

给 Claude 用的中文文献检索插件，面向药学，重点管理药学（药事管理、药物警戒、药物经济学、医药政策、药物流行病学、监管科学、合理用药、医院药学等）。

它做三件事：

1. 在你自己的电脑上，借助你本人的机构订阅，检索知网（CNKI）和万方；
2. 提供一套检索方法学（拆题、概念组、调式、筛选分级），让检索式可复现、可写进方法学部分；
3. 把纳入文献跨库去重后导出为 RIS / GB/T 7714 / CSV / BibTeX 等格式，并附检索记录。

## 组成

| 组件 | 内容 |
|---|---|
| MCP 服务器 `cnlit` | 知网、万方的检索、计数调式、题录/摘要、分组统计、跨库去重导出、打开原文页。零依赖 Node 脚本 |
| Skill `pharm-lit-search` | 检索方法学：拆题、概念组、调式、筛选分级、导出与检索记录；附管理药学 8 个子领域词表、两库语法实测、模板、DILI 与附条件批准两个示例 |

## 快速开始

### 方式一：作为 Claude Code 插件安装

```
/plugin marketplace add <3226789833tang-crypto>/pharm-lit-cn
/plugin install pharm-lit-cn@pharm-lit-cn
```

安装后重启会话，对 Claude 说“帮我查一下药物性肝损伤信号检测的中文文献”即可触发 skill。

### 方式二：手动配置 MCP（Claude Desktop、Codex 等任意支持 MCP 的客户端）

```bash
git clone https://github.com/<3226789833tang-crypto>/pharm-lit-cn.git
```

在客户端的 MCP 配置里加入（路径改成你本机的实际路径）：

```json
{ "mcpServers": { "cnlit": { "command": "node", "args": ["/path/to/pharm-lit-cn/server/cnlit.mjs"] } } }
```

Codex（`config.toml`）：

```toml
[mcp_servers.cnlit]
command = "node"
args = ["/path/to/pharm-lit-cn/server/cnlit.mjs"]
```

skill 部分可把 `skills/pharm-lit-search/SKILL.md` 与 `references/` 复制到对应客户端的 skills / rules 目录。

## 要求

- Node.js ≥ 18（`node` 在 PATH 中）
- Google Chrome 或 Microsoft Edge（Windows 自带 Edge 即可）
- 机构订阅：校园网 / VPN，或 CARSI 机构登录

服务器代码对 Windows、macOS、Linux 都做了浏览器路径与数据目录的适配，但主要在 Windows 上使用和测试。

## 工作方式

`cnlit` 在你电脑上启动一个**专用的 Chrome/Edge 窗口**（独立配置目录，默认最小化），在知网、万方页面内部调用网站自己的接口：

- 身份来自你的机构 IP（校园网/VPN）或你在该窗口里完成的机构登录（CARSI），登录状态保存在本机；
- 账号密码只在浏览器窗口里由你输入，不经过 MCP；
- 遇到滑块验证时窗口自动前置，由你完成；
- 浏览器通过管道（`--remote-debugging-pipe`）控制，不开放任何调试端口；空闲 30 分钟自动关闭；
- 调用万方接口时，脚本会在页面内读取该页面当前会话的 `CASTGC` Cookie 并放进请求头，这与万方网页自身的行为一致，Cookie 不会被保存或发送到其他地址。

## 工具

| 工具 | 作用 |
|---|---|
| `lit_status` | 启动/检查检索浏览器，报告知网、万方是否可检索及机构身份 |
| `lit_count` | 只算命中数，用于比较检索式、逐组/逐词贡献分析 |
| `lit_search` | 检索，返回命中数、可复现检索式与题录；支持概念组自动编译，或直接写两库专业检索式 |
| `lit_details` | 取摘要、关键词、机构、基金、分类号等完整题录 |
| `lit_export` | 跨库去重导出 RIS、GB/T 7714、CSV、BibTeX、Markdown、JSON，可导入运行中的 Zotero |
| `lit_facets` | 知网分组统计：主题、学科、来源类别分布 |
| `lit_open` | 在检索窗口打开记录或站点页，用于登录、过验证、手动下载全文 |
| `lit_log` | 列出检索记录，用于写检索日志与方法学描述 |

推荐流程：`lit_status` → 拆题建概念组 → `lit_count` 调式 → `lit_search` 取题录 → `lit_details` 读摘要分级 → `lit_export` 导出。详见 [`skills/pharm-lit-search/SKILL.md`](skills/pharm-lit-search/SKILL.md)。

## 目录结构

```
pharm-lit-cn/
├── .claude-plugin/
│   ├── plugin.json            # 插件清单
│   └── marketplace.json       # 让本仓库可被 /plugin marketplace add 直接安装
├── .mcp.json                  # MCP 服务器声明
├── server/                    # cnlit MCP 服务器（Node，无第三方依赖）
│   ├── cnlit.mjs
│   ├── browser.mjs
│   ├── pagelib.mjs
│   └── records.mjs
└── skills/pharm-lit-search/
    ├── SKILL.md               # 检索方法学
    └── references/            # 子领域词表、语法、筛选标准、示例
```

## 文件位置

- 导出：`文档\文献检索导出\`（可在 `lit_export` 里用 `out_dir` 指定）
- 数据：Windows 为 `%LOCALAPPDATA%\cnlit\`（浏览器配置、检索记录 `search-log.jsonl`、检索缓存）

## 环境变量（可选）

| 变量 | 作用 |
|---|---|
| `CNLIT_BROWSER` | 指定浏览器程序路径 |
| `CNLIT_EXPORT_DIR` | 默认导出目录 |
| `CNLIT_DATA_DIR` | 数据目录 |
| `CNLIT_MINIMIZE=0` | 启动后不最小化窗口 |
| `CNLIT_CNKI_INTERVAL_MS` / `CNLIT_WANFANG_INTERVAL_MS` | 请求间隔（默认 1500 / 700 毫秒） |
| `CNLIT_IDLE_CLOSE_MIN` | 空闲自动关闭浏览器的分钟数（默认 30） |

## 使用边界与免责声明

- 本项目是个人科研辅助工具，通过**你本人的机构订阅**访问知网、万方，与两家公司无任何隶属或授权关系。
- 请勿批量下载全文，或把访问能力转给无权限者；请遵守知网、万方的服务协议。请求间隔已设默认值，不建议调低。
- 网站接口随时可能变化，导致部分功能失效；欢迎提 Issue。
- 检索结果仅供科研参考，引用前请核对原文。
- 软件按“现状”提供，不提供任何担保。

## 许可证

[MIT](LICENSE)
