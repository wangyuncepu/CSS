# 需求贡献分评分方案（Contribution Score）

> **AI 需求贡献分评分方案 v2.0** — 从 PRD/TRD 需求文档中客观计数，套公式计算贡献分，直接用于分成结算。
>
> 适用于：需求池认领、按贡献分比例分成的协作模式（PM 写需求，开发拿分成）

---

## 核心逻辑

**AI 从需求文档中客观计数 → 规则引擎套公式计算 → 人只做复核**

三步走，每一步都可被开发者手动验算。规则固定、输入客观、过程可复现、结果可审计。

## 安装

### Claude Code（推荐）

```bash
claude plugin marketplace add dontbesilent2025/contribution-score
claude plugin install 需求评分@contribution-score-skill
```

### npx（通用）

```bash
npx -y skills add dontbesilent2025/contribution-score -g --all
```

### Trae Solo

下载最新 Release 的 ZIP 包，解压后取得 `skills/需求评分/SKILL.md`，拖入 Trae Solo 上传窗口。

## 使用

触发方式：

| 说法 | 效果 |
|------|------|
| "给这份 PRD 评分" | 自动识别需求类型，按对应维度计数 |
| "评估这个需求" | 同上 |
| "算一下贡献分" | 同上 |
| `/需求评分` | 直接调用 skill |

## 需求类型

| 类型 | 输入 | 评分方式 |
|------|------|---------|
| **功能型** | PRD | 三维客观计数（功能广度×40% + 业务逻辑×35% + 数据复杂度×25%） |
| **研发型** | TRD | 三维计数 × 不确定性系数（几何平均） |
| **混合型** | PRD + TRD | 功能分 + 研发分×0.8 |
| **重构型** | 简述 | PM 拍固定档 S=2 / M=5 / L=8 |

## 贡献分映射

| 原始分 | 贡献分 | 等级 | 参考工作量 |
|--------|--------|------|-----------|
| 0–10 | 1 (XS) | XS | ≤0.5 天 |
| 11–22 | 2 (S) | S | 0.5–1.5 天 |
| 23–40 | 3 (S+) | S+ | 1.5–3 天 |
| 41–65 | 5 (M) | M | 3–6 天 |
| 66–95 | 8 (L) | L | 1.5–2.5 周 |
| 96–130 | 13 (XL) | XL | 2.5–4 周 |
| 130+ | 20 (XXL) | XXL | >1 月（强制拆分） |

## 仓库结构

```
contribution-score/
├── .claude-plugin/
│   └── marketplace.json      # 插件清单
├── skills/
│   └── 需求评分/
│       └── SKILL.md          # 技能文件
├── README.md
├── LICENSE
└── VERSION
```

## 许可

CC BY-NC 4.0 — 个人/学习/研究/非商业项目无需署名；公开衍生作品请注明来源；商业使用请联系作者。

## 作者

- GitHub: [@dontbesilent2025](https://github.com/dontbesilent2025)

---

> **原则**：规则固定、输入客观、过程可复现、结果可审计。每一步都可被开发者手动验算。
