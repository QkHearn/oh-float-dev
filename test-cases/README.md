# 评测说明（hmos-arkui-kit-window-floatwindow）

## 目录

```text
test-cases/
├── README.md                 本说明
├── smoke.md                  分流冒烟（问答）
├── functional.md             接口/查错问答
├── smoke/                    冒烟 JSON
└── scene-functional/         功能 JSON
```

## 怎么跑

1. 把本 Skill 装到评测 Agent。
2. 按 `smoke.md` / `functional.md` 把「输入」喂给 Agent。
3. 对照「预期」核对**口头答案**是否命中能力与关键要点。

本套用例**聚焦问答**（分流 / 接口 / 查错说明）：

- **不要求**改工程、落盘 ets、追问工程路径
- **不要求** `CompileArkTS` / `hvigorw`
- **PASS**：答对能力或关键要点即可
- **FAIL**：分流错能力，或把范围外需求当成可写

## 指标

| 指标 | 口径 | 建议门槛 |
|------|------|----------|
| 分流正确率 | 命中预期能力 / 冒烟用例数 | ≥ 90% |
| 问答正确率 | 命中要点 / 功能问答用例数 | ≥ 90% |
