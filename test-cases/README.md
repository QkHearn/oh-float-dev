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
3. 对照「预期」核对流、落点、语法与编译。
4. 写程序类用例：Agent 改完应用工程后，必须在工程根自己跑：

```bash
cd <应用工程根> && hvigorw assembleHap -p product=default --no-daemon
```

日志里要有 `CompileArkTS` 成功。编不过不算该用例通过。

## 指标

| 指标 | 口径 | 建议门槛 |
|------|------|----------|
| 分流正确率 | 命中预期能力 / 冒烟用例数 | ≥ 90% |
| 语法自检通过率 | 交卷前完成 arkui-syntax 必检项 / 写程序用例数 | 100% |
| 编译成功率 | `CompileArkTS` 通过 / 产生 ets 的写程序用例数 | ≥ 70% |

纯分流、纯答疑、范围外停手类不计入编译分母。
