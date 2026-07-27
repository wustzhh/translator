---
name: khorne-translator
description: Warhammer Khorne模组翻译流水线 — 切分、翻译、合并、进度追踪，支持中断续传
---

# Warhammer Khorne 模组翻译流水线

用于 `D:\pyProj\translator` 项目的英译中翻译管理。将大型英文原文按切片拆分、翻译、合并，全程记录进度以支持中断续传。

## 目录结构

```
translator/
├── english_fields.md      # 原始英文字段（全量）
├── translations.md         # 原有600条翻译参考
├── glossary.md             # 术语表（翻译规范）
├── progress.md             # 进度追踪文件
├── slices/                 # 切片目录（每20条一个文件）
│   ├── slice_001.md
│   └── ...
├── translated/             # 翻译结果（与切片一一对应）
│   ├── trans_001.md
│   └── ...
├── translations_full.md    # 最终合并输出
├── split.py                # 切分脚本
├── prefill.py              # 预填已知翻译脚本
├── merge.py                # 合并脚本
├── check_coverage.py       # 覆盖率检查
└── export_todo.py          # 导出待翻译清单
```

## 完整工作流

### 1. 检查状态
```bash
python check_coverage.py   # 查看翻译覆盖率
```
读取 `progress.md` 了解哪些切片已完成/待处理。

### 2. 切分原文（首次或原文更新后）
```bash
python split.py   # 按20条/文件切分 english_fields.md → slices/
```
自动创建 `slices/` 目录，生成 `slice_001.md` ~ `slice_NNN.md`。

### 3. 预填已知翻译
```bash
python prefill.py  # 从 translations.md 加载已知翻译 → translated/
```
自动创建 `translated/` 目录，将已有翻译填入对应位置，无翻译的标记为 `【待翻译】`。

### 4. 批量翻译
使用 fleet 并行翻译多个切片。每个子agent负责10个切片，约200条：
- 每个子agent先读取 `glossary.md` 获取术语规范
- 依次处理分配的 `trans_XXX.md` 文件
- 用 `edit_file` 将 `【待翻译】` 替换为中文翻译
- 翻译规范：
  - Khorne→恐虐, Blood God→血神, Bloodthirster→嗜血狂魔, Bloodletter→放血鬼
  - Bloodflame→血焰, Hell-Forged→地狱锻造, Hellblade→地狱之刃
  - "X of Khorne" → "恐虐X"
  - 技能名保持四字/五字格简洁风格
  - 描述文本保持暴力美学和史诗感
  - 人名音译参考 glossary.md

### 5. 更新进度
每完成一批切片，更新 `progress.md` 中对应切片状态为 `✅ done`。

### 6. 合并输出
```bash
python merge.py   # 合并 translated/ → translations_full.md
```

## 中断续传

1. 查看 `progress.md` 确定哪些切片未完成
2. 运行 `python prefill.py` 确保 translated/ 文件状态最新
3. 运行 `python export_todo.py` 导出剩余待翻译清单
4. 对未完成的切片继续翻译
5. 完成后 `python merge.py` 重新合并

## 翻译规范速查

### 核心术语
| English | 中文 |
|---------|------|
| Khorne | 恐虐 |
| Blood God | 血神 |
| Bloodthirster | 嗜血狂魔 |
| Bloodletter | 放血鬼 |
| Bloodflame | 血焰 |
| Hell-Forged | 地狱锻造 |
| Hellblade | 地狱之刃 |
| Blood Queen | 血之女王 |
| Skull Throne | 颅骨王座 |
| Brass Citadel | 黄铜要塞 |
| Gore | 血/血淋淋 |
| Slaughter/Carnage | 屠杀 |
| Ruin | 毁灭 |
| Rage/Fury/Wrath | 狂怒/愤怒/怒火 |

### 人名音译
- Khorgoth → 科尔戈斯
- Varrok → 瓦洛克
- Azrakaar → 阿兹拉卡尔
- Khazdrakk → 卡兹德拉卡
- Skorrog → 斯科罗格
- Hespia → 赫斯帕
- Gormak → 戈马克
- Ka'vuul → 卡武尔
- Asmodan → 阿斯莫丹
- Calarax → 卡拉拉克斯

### 格式保留
- `%+n` / `%+n%` 数值占位符保持不变
- `{{tr:rank7}}` 等模板标记保持不变
- `[[img:...]]` 等图片标记保持不变
- 损坏的技术标识符（如 `C.unit_description_short_texts_...`）保留不译
