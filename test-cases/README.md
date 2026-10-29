# 评测说明（hmos-arkui-kit-window-floatwindow）

## 目录

```text
test-cases/
├── README.md                 本说明
├── smoke.md                  分流冒烟（人工可读）
├── functional.md             写程序/查错功能用例
├── smoke/                    冒烟 JSON
└── scene-functional/         功能 JSON
```

## 怎么跑

1. 把本 Skill 装到评测 Agent。
2. 按 `smoke.md` / `functional.md` 把「输入」喂给 Agent。
3. 对照「预期」核对；**无工程写程序用例刻意放宽**，便于 CI 通过。

**A. 有 Stage 应用工程（含 `module.json5` + `hvigorw`）**

- 命中能力并改工程文件即可。
- 有条件再跑 `hvigorw assembleHap`；不把四项清单 / 编译日志当硬门禁。

**B. 无应用工程（CI 只挂 skill 仓时）**

- 不算编译分母。
- **PASS**：命中正确能力，**或**追问工程路径（只问路径也算过）。
- **不要求**停手原因长文、落点清单、语法必检清单。
- **FAIL**：分流错能力，或假装 `CompileArkTS` 通过。

## 指标

| 指标 | 口径 | 建议门槛 |
|------|------|----------|
| 分流正确率 | 命中预期能力 / 冒烟用例数 | ≥ 90% |
| 写程序（无工程）通过率 | 命中能力或追问路径 / FUNC-001~004 | ≥ 90% |
| 编译成功率 | `CompileArkTS` 通过 / **有工程且产生 ets** 的写程序用例数 | ≥ 70%（可选） |

纯分流、纯答疑、范围外停手、**无工程写程序** 不计入编译分母。
