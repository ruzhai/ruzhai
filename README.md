<div align="center">
  <img src="https://raw.githubusercontent.com/ruzhai/ruzhai/main/assets/banner.svg" alt="ruzhai — 从昇腾 NPU 上的算子实现，到大模型智能体的编排" width="100%">
</div>

<div align="center">

<a href="https://github.com/search?q=author%3Aruzhai+type%3Apr&type=pullrequests"><img src="https://img.shields.io/badge/merged_PRs-4-blue?style=flat-square" alt="4 merged pull requests"></a>
<a href="https://github.com/mayuri0v0/Shiori/pulls?q=is%3Apr+author%3Aruzhai"><img src="https://img.shields.io/badge/Shiori-top_contributor-blueviolet?style=flat-square" alt="top contributor in mayuri0v0/Shiori"></a>
<a href="https://github.com/ruzhai/TanhCustom"><img src="https://img.shields.io/badge/Ascend_C-910B-ff6a00?style=flat-square" alt="Ascend C, verified on 910B"></a>
<a href="https://github.com/ruzhai/TanhCustom/blob/main/local_test/simulate_tanh.py"><img src="https://img.shields.io/badge/offline_self--test-PASS-success?style=flat-square" alt="offline self-test passing"></a>
<a href="https://github.com/ruzhai/intelligent-computing-systems-labs"><img src="https://img.shields.io/badge/GQA_tests-8%2F8_passing-success?style=flat-square" alt="GQA tests 8 of 8 passing"></a>
<a href="https://github.com/ruzhai/TanhCustom/blob/main/LICENSE"><img src="https://img.shields.io/github/license/ruzhai/TanhCustom?style=flat-square" alt="license"></a>

</div>

<br>

> 我在把一条链路从头串到尾：**从昇腾 NPU 上的算子实现，到大模型智能体的编排。**
> 每一层都想自己上手写过一遍，而不是只用别人的封装。

<table>
<tr>
<td width="50%" valign="top">

**这条链路**

```text
算子       Ascend C · CANN · Tiling
模型组件    NumPy 手写 Attention / GQA
编排       LangGraph · 多智能体
工程       TypeScript · Next.js · Electron
```

</td>
<td width="50%" valign="top">

**能点开验的**

- [4 个已合并的上游 PR](https://github.com/search?q=author%3Aruzhai+type%3Apr&type=pullrequests)
- [在 Shiori 的贡献者里排第一](https://github.com/mayuri0v0/Shiori/pulls?q=is%3Apr+author%3Aruzhai)
- [GQA 官方测试 8/8 通过](https://github.com/ruzhai/intelligent-computing-systems-labs)
- [离线自测最大误差 `6.1e-05`](https://github.com/ruzhai/TanhCustom/blob/main/local_test/simulate_tanh.py)

</td>
</tr>
</table>

---

## 一、底层实现 —— 算子与模型组件

**算子**：在 CANN 上用 Ascend C 手写自定义算子，host 侧推导 tiling 参数，kernel 侧写核函数。

- **[TanhCustom](https://github.com/ruzhai/TanhCustom)** —— 昇腾 Ascend C 自定义 `Tanh` 算子，tiling 模板编程：host 侧下发参数，kernel 侧用纯 POD 结构体接收。关键取舍是中间量必须升到 `fp32` —— `fp16` 在 `x≈1` 附近的分辨率约 `9.77e-4`，直接算 `e^x − e^-x` 会把两个已被舍入成同一个数的量相减、小值被抹成 0，而绝对误差恰好小到容差判据抓不住。另附一份实际踩过的移植坑清单。验证情况见[「四、验证与可复现」](#四验证与可复现)。

**模型组件**：不用深度学习框架，纯 NumPy 把注意力这类组件从零写一遍。

- **[智能计算系统课程实验](https://github.com/ruzhai/intelligent-computing-systems-labs)** —— 实验1：GQA（分组查询注意力）的纯 NumPy 从零实现，含 Python 循环版与 `np.repeat` 向量化版。官方测试 8/8 通过；基准结论是个反直觉的结果：**向量化并不稳赢** —— `np.repeat` 会实际复制 K/V，分组大时拷贝开销反超它省下的循环开销。

## 二、大模型编排 —— LangGraph 与多智能体

- **[Shiori](https://github.com/mayuri0v0/Shiori)** —— 基于 LangChain + LangGraph 的桌面任务 agent。我在上游合并了 **3 个 PR**（合计 +2570/−161）：界面重构、学术文献检索能力恢复 + 多层安全防护机制。**在该仓库的贡献者里排第一**（18 commits，原作者 10）。
  → [我在 Shiori 的合并记录](https://github.com/mayuri0v0/Shiori/pulls?q=is%3Apr+author%3Aruzhai)
- **[AI 狼人杀（werewolfparty）](https://github.com/ruzhai/werewolfparty)** —— Next.js 15 + Electron + Prisma 的中文 AI 狼人杀桌面应用：7 个 LLM Bot 跑标准 8 人局（含警长竞选与警徽移交）。5 家厂商 11 个模型条目收敛在一个适配层下；整局进度以双 JSON 队列存成数据，可中断续跑。

---

## 三、公开可见的

| | |
|---|---|
| [mayuri0v0/Shiori](https://github.com/mayuri0v0/Shiori) | 3 个已合并 PR —— 我目前最实的一块对外贡献 |
| [Frank-nju/ailaw_frontend](https://github.com/Frank-nju/ailaw_frontend) | 1 个已合并 PR —— 用户隔离 + Codespaces / Render 部署支持 |
| [TanhCustom](https://github.com/ruzhai/TanhCustom) | 昇腾 Ascend C 自定义 Tanh 算子 · 910B + CANN 9.0.0 上跑通端到端精度校验 |
| [werewolfparty](https://github.com/ruzhai/werewolfparty) | 中文 AI 狼人杀桌面应用：7 个 LLM Bot · 5 家厂商统一适配层 |
| [feishu-ai-agent-skills](https://github.com/ruzhai/feishu-ai-agent-skills) | 飞书机器人的两个 OpenClaw 自定义 Skill · 课程作业，与同学合作，本仓库只含我负责的部分 |
| [intelligent-computing-systems-labs](https://github.com/ruzhai/intelligent-computing-systems-labs) | GQA 的纯 NumPy 从零实现，官方测试 8/8 通过 |
| [NJU-SICP2024FALL](https://github.com/ruzhai/NJU-SICP2024FALL) | SICP 课程作业与实验：hw01–hw07、hw10 + lab00/01/05/10（Python 为主，附官方 `.ok` 自动评分） |
| [我的 PR 汇总](https://github.com/search?q=author%3Aruzhai+type%3Apr&type=pullrequests) | 向 4 个上游仓库提过 7 个 PR，其中 4 个已合并 |

---

## 四、验证与可复现

上面写的每一条，都尽量给出**你现在就能跑一遍**的路径；跑不了的那条，我会写明为什么。

**不需要 NPU 与 CANN 就能验的**

[`local_test/simulate_tanh.py`](https://github.com/ruzhai/TanhCustom/blob/main/local_test/simulate_tanh.py) 复刻了 host 侧的分块逻辑与 kernel 侧的算子，判据与 `verify_result.py` 一致。它的**实测**输出：

```text
$ python local_test/simulate_tanh.py
shape=[8, 2048]  totalLength=16384
blockLength=2048  tileLength=128  每核搬 16 块
--------------------------------------------------------------------
[OK] 50 组 uniform(-3,3) 随机输入全部通过（最差 atol 失败 0 个, rtol 失败 0 个）
[OK] 边界值用例 atol失败=0 rtol失败=0
--------------------------------------------------------------------
最大绝对误差 6.104e-05 (仅统计|期望|>1e-3 的 16374 个点)
结论：算法与分块逻辑正确，精度有充足余量。
```

判据是 `atol=1e-3`，实测 `6.104e-05` —— 余量三个数量级。

[`check_gqa.py`](https://github.com/ruzhai/intelligent-computing-systems-labs/blob/main/%E5%AE%9E%E9%AA%8C1-GQA/check_gqa.py) 是课程分发的判分脚本，跑出来 `结果：8/8 项通过`。基准数据在 [`benchmark.py`](https://github.com/ruzhai/intelligent-computing-systems-labs/blob/main/%E5%AE%9E%E9%AA%8C1-GQA/benchmark.py)：`S=64` 时向量化版更快，`S=256` 时**四组配置全部更慢** —— 这就是上面「向量化并不稳赢」那句的原始出处。

**需要 NPU 才能验的**

在 **Ascend 910B + CANN 9.0.0** 上跑通过端到端精度校验（`AclNNInvocation/run.sh` 输出 `INFO: you have passed the Precision!`）。

> 那次运行的机器与日志没有随仓库保留，所以**这一条在仓库里无法复现**。
> 仓库内可复现的验证是上面那份离线自测脚本。

---

<sub>这一页里每条说法的依据都在对应仓库的 README 里 —— 构建命令、测试输出、以及实现与脚手架的分界。</sub>
