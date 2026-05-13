# octoarms-x-collector

统一维护 Octoarms X/Twitter 监控 list 的查询与排障规范。

## 快速索引

- 主文档：`SKILL.md`
- 支持主题：`HBM` / `Tesla` / `SiC (Wolfspeed)`

## 主题配置速览

| Topic | list_id | source_name | members source | tweets source | base tag |
|---|---|---|---:|---:|---|
| HBM | `2053711728324874698` | `octoarms_hbm` | `42` | `43` | `hbm` |
| Tesla | `2054109618323030125` | `octoarms_tesla` | `44` | `45` | `tesla` |
| SiC | `2054481346467377636` | `octoarms_sic_wolfspeed` | `46` | `47` | `sic` |

## 常用入口

1. 标准查询模板（按账号、按时间窗、按 source_tags）
   - 见 `SKILL.md`：`Standard Query`、`Windowed Query Templates`、`Source Tag Filtering`
2. 空结果排障（尤其是 source_tags）
   - 见 `SKILL.md`：`Troubleshooting Empty Results`、`SiC-specific Troubleshooting`

## 使用前置

```bash
export OCTOARMS_BASE_URL="https://api.chainbot.io"
export OCTOARMS_API_KEY="<YOUR_API_KEY>"
```
