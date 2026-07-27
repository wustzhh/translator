# translator — Warhammer 3 恐虐模组翻译工具链

`git@github.com:wustzhh/translator.git` 的本地克隆。用于《全面战争：战锤3》恐虐（Khorne）模组的英译中翻译管理。

## 项目

- **语言/栈**：Python 3（翻译流水线脚本），Markdown（翻译数据）
- **数据目录**：`warhammer3-khorne/`
  - `english_fields.md` — 原始英文字段（1761 条）
  - `translations.md` — 已有 600 条参考翻译
  - `translations_full.md` — 最终完整翻译输出（1761 条，已全部完成）
  - `glossary.md` — 术语表（翻译规范）
  - `progress.md` — 进度追踪（全部 89 切片已完成）
- **Python 脚本**：预计在 `tools/` 目录（`.gitignore` 忽略），包括 `split.py`, `prefill.py`, `merge.py`, `check_coverage.py`, `export_todo.py`

## 命令

```bash
python tools/split.py              # 切分 english_fields.md → slices/
python tools/prefill.py            # 预填已有翻译 → translated/
python tools/merge.py               # 合并 translated/ → translations_full.md
python tools/check_coverage.py      # 查看翻译覆盖率
python tools/export_todo.py         # 导出待翻译清单
```

## 架构

| 模块/文件 | 职责 |
|-----------|------|
| `english_fields.md` | 全量原文输入（1761 条英文文本） |
| `translations.md` | 已有的 600 条参考翻译（翻译记忆库） |
| `glossary.md` | 术语规范（核心术语、人名音译、格式规则） |
| `split.py` | 将原文按 N 条/文件切分为 `slices/slice_NNN.md` |
| `prefill.py` | 从 `translations.md` 预填已有翻译到 `translated/trans_NNN.md` |
| `merge.py` | 将所有 `translated/` 文件合并为 `translations_full.md` |
| `check_coverage.py` | 统计翻译覆盖率 |
| `export_todo.py` | 导出未翻译条目清单 |
| `progress.md` | 追踪每个切片状态（⬜ 🔄 ✅ ❌），支持中断续传 |

## 约定

- **术语规范**：所有翻译必须遵守 `glossary.md` 中的术语对照表（如 Khorne→恐虐, Bloodthirster→嗜血狂魔）
- **格式保留**：`%+n` 数值占位符、`{{tr:...}}` 模板标记、`[[img:...]]` 图片标记均保持原样不译
- **进度标记**：切片状态使用规定的 4 种符号（⬜ pending / 🔄 translating / ✅ done / ❌ blocked）
- **切片粒度**：每个切片 20 条，序号从 1 开始
- **中断续传**：查看 `progress.md` → 运行 `prefill.py` 恢复 → 继续未完成切片 → 运行 `merge.py` 合并

## 备注

（待补充）
