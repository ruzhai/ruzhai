# ruzhai

`Ascend C / CANN` · `LangGraph / 多智能体` · `Python · TypeScript`

> 我在把一条链路从头串到尾：**从昇腾 NPU 上的算子实现，到大模型智能体的编排。**
> 每一层都想自己上手写过一遍，而不是只用别人的封装。

---

## 一、底层实现 —— 算子与模型组件

**算子**：在 CANN 上用 Ascend C 手写自定义算子，host 侧推导 tiling 参数，kernel 侧写核函数。

- **TanhCustom** —— 昇腾 Ascend C 自定义 `Tanh` 算子，tiling 模板编程：host 侧下发参数，kernel 侧用纯 POD 结构体接收。已在 **910B + CANN 9.0.0** 上端到端验证，精度判据 `1e-3` 通过。关键取舍是中间量必须升到 `fp32` —— `fp16` 在 `x≈1` 附近的分辨率约 `9.77e-4`，直接算 `e^x − e^-x` 会把两个已被舍入成同一个数的量相减、小值被抹成 0，而绝对误差恰好小到容差判据抓不住。另附一份实际踩过的移植坑清单。*（代码目前沉淀在 GitCode，GitHub 侧整理中）*

**模型组件**：不用深度学习框架，纯 NumPy 把注意力这类组件从零写一遍。

- **[智能计算系统课程实验](https://github.com/ruzhai/intelligent-computing-systems-labs)** —— 实验1：GQA（分组查询注意力）的纯 NumPy 从零实现，含 Python 循环版与 `np.repeat` 向量化版。官方测试 8/8 通过；基准结论是个反直觉的结果：**向量化并不稳赢** —— `np.repeat` 会实际复制 K/V，分组大时拷贝开销反超它省下的循环开销。

## 二、大模型编排 —— LangGraph 与多智能体

- **[Shiori](https://github.com/mayuri0v0/Shiori)** —— 基于 LangChain + LangGraph 的桌面任务 agent。我在上游合并了 **3 个 PR**（合计 +2570/−161）：界面重构、学术文献检索能力恢复 + 多层安全防护机制。**在该仓库的贡献者里排第一**（18 commits，原作者 10）。
  → [我在 Shiori 的合并记录](https://github.com/mayuri0v0/Shiori/pulls?q=is%3Apr+author%3Aruzhai)
- **[AI 狼人杀（werewolfparty）](https://github.com/ruzhai/werewolfparty)** —— Next.js 15 + Electron + Prisma 的中文 AI 狼人杀桌面应用：7 个 LLM Bot 跑标准 8 人局（含警长竞选与警徽移交）。5 家厂商 11 个模型条目收敛在一个适配层下；整局进度以双 JSON 队列存成数据，可中断续跑。

---

## 公开可见的

| | |
|---|---|
| [mayuri0v0/Shiori](https://github.com/mayuri0v0/Shiori) | 3 个已合并 PR —— 我目前最实的一块对外贡献 |
| [werewolfparty](https://github.com/ruzhai/werewolfparty) | 中文 AI 狼人杀桌面应用：7 个 LLM Bot · 5 家厂商统一适配层 |
| [intelligent-computing-systems-labs](https://github.com/ruzhai/intelligent-computing-systems-labs) | GQA 的纯 NumPy 从零实现，官方测试 8/8 通过 |
| [NJU-SICP2024FALL](https://github.com/ruzhai/NJU-SICP2024FALL) | SICP 课程作业与实验：hw01–hw10 + lab00–lab10（Python / Scheme / SQL，官方 `.ok` 自动评分） |
| [我的 PR 汇总](https://github.com/search?q=author%3Aruzhai+type%3Apr&type=pullrequests) | 向 4 个上游仓库提过 7 个 PR，其中 4 个已合并 |

---

## 眼下在做的

- 把算子线整理上 GitHub —— 它目前主要沉淀在 GitCode 上，这里还看不到
