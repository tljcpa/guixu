# 归墟 · 相邻领域开源/学术全景对比(2026-06)

> 来源:多 agent 深度调研(6 路搜索 → 25 源 → 119 条 claim → 对抗核验,23 confirmed / 2 killed)。
> 每条结论后附 confidence 与来源。带 caveat 的地方都标了,别把解读当原文。

---

## 0. 一句话结论

归墟的核心赌注——**原始完整过程 append-only 保留 + 运行时整段注入复用 + 反压缩/反 schema**——在被调研的项目里处于**一个相对空白、但并非无人触及**的位置:

- **"整段原始 trajectory 运行时注入新会话做经验迁移"——没有任何被核验的来源在做。这是真空白。**
- 但"原始留存"的哲学有人沾边(Generative Agents 的 memory stream、LLM 可观测性栈),"反压缩"的判断有顶会论文背书(ACE)。
- 主流(A-MEM / ExpeL / AWM / AutoManual / EvolveR)**几乎一致地走向你刻意反对的方向**:把经验蒸馏成结构化笔记/规则/workflow,再检索片段或注入规则复用。

**潜台词(我加的,不是调研结论):这个空位可能空得有原因——大家都压缩,正因为你已经撞到的那堵窗口墙。**

---

## 1. 全景对比表(A–F 六维度)

| 项目 | A 存什么 | B 复用机制 | C 规模/窗口解法 | D 结构化程度 | E 格式可移植 | F 定位 |
|---|---|---|---|---|---|---|
| **归墟** | 原始完整 trajectory,append-only | **整段注入(运行时)** | 撞墙,需"选哪段"(未解) | schemaless | json/md/txt 通用 | 经验复用 |
| A-MEM | 结构化笔记(描述/关键词/tag) | 检索片段 | 检索 top-k | **强结构 + Zettelkasten 图** | 私有库 | 长期记忆 |
| ExpeL | 蒸馏的 NL insights | Faiss top-k 检索相似 traj + 注入全部 insights | 检索 | 中(insights) | 私有库 | 自我改进 |
| AWM (Agent Workflow Memory) | 抽象出的 compact workflow(NL+参数化步骤) | 选择性注入 workflow | 抽象压缩 | 中结构 | 私有库 | 自我改进 |
| AutoManual | 六类结构化规则→人类可读手册 | 注入规则/手册 | 规则压缩 | **强结构** | 手册(可读) | 自我改进 |
| EvolveR | 抽象 strategic principles + 知识三元组 | top-3 检索 | **用量评分 + 周期剪枝** | 强结构 | 私有库 | 自我改进 |
| **Generative Agents**(memory stream) | **append-only 原始 NL 事件**(+上叠 reflections 抽象层) | **三因子打分 top-k 检索片段**(填满窗口为止) | 评分选择 | **schemaless(底层)** | NL 文本 | 长期记忆/模拟 |
| **MemGPT / Letta** | core 常驻 + 外部记忆分层 | LLM 自发 function call 按需 page-in | **OS 式分层虚拟上下文** | 半结构 | **agent-file(.af)可移植** | 长期记忆 |
| **LangSmith**(可观测) | **完整原始 trajectory**(每次 LLM/tool 调用) | 实际是可视化 debug + trace→dataset 评估 | 单 item ~1MB 截断 | trace 结构 | 接近 OTel | **debug 观测** |
| **OTel GenAI 语义约定** | **完整原始 prompt/completion**(JSON) | 不做复用(只是标准) | — | 正式 JSON schema | **跨厂商可移植 ✓** | 观测标准 |
| ACE | 带 ID + helpful/harmful 计数的结构化 bullet | delta 增量 + grow-and-refine | 去重剪枝 | 中结构 | — | 自我改进 |
| Claude Code memory / Agent Skills / AGENTS.md | 人写的规则/skill | 注入 | — | 半结构 | AGENTS.md / .af 跨工具 | 长期记忆/技能 |

(加粗 = 与归墟同向或最相关的格。)

---

## 2. 五个核验过的发现

**① 主流一致走向"蒸馏+结构化",没人做整段原始复用** — confidence: high
A-MEM(结构化笔记+图)、ExpeL(NL insights+检索)、AWM(抽象 workflow)、AutoManual(六类规则)、EvolveR(strategic principles)五个独立 primary 来源,**全部指向"压缩+结构化+检索/规则注入"范式**。归墟的反方向在这批里是孤例。
源:arxiv 2502.12110 / 2308.10144 / 2409.07429 / 2405.16247 / 2510.16079

**② 最接近归墟"原始留存"哲学的是 Generative Agents 的 memory stream** — high
append-only 的 memory object 列表,每条 = 自然语言描述 + 时间戳,基本单元是直接感知的 observation 事件——**schemaless、事件序列化、用 NL 而非预结构化**,与归墟高度对齐。
**但两点关键差异(caveat):** 它在 stream 之上叠了 reflections(合成的高层抽象),所以不是纯原始;且复用走 `score = recency + importance + relevance` 打分后 **top-k 检索片段**注入,**不是整段注入**——恰恰是归墟想避开的检索路线。(这条 raw-retention 的定性是 2-1 分票,即有争议。)
源:arxiv 2304.03442

**③ 离归墟最近的工程暗线:LLM 可观测性栈已在存完整原始 trajectory** — high
LangSmith trace 捕获完整执行(每次 LLM/tool 调用、决策序列、原始 inputs/outputs,单会话数 MB)。LangChain 自己写:"Tracing is **the primitive that makes the entire improvement loop possible**",明确不止 debug。
**caveat:这是 vendor 营销定位**,实际落地仍是可视化 debug + trace→dataset 评估,**并未真正"对新模型整段 replay 复用经验"**;且单 item ~1MB 截断。方向同源,落点不同。
源:langchain.com/blog/traces-start-agent-improvement-loop ; langchain.com/langsmith/observability

**④ ACE 论文从理论上背书了归墟的反压缩赌注** — high
ACE(2510.04618)明确把两个失败模式命名出来:**brevity bias**(简洁摘要丢领域洞见)与 **context collapse**(迭代改写随时间侵蚀细节)。这是"反 memory/summary 压缩"判断的学术背书。
**但 caveat:ACE 自己的解法是"结构化增量保留细节"的中间路线**(带 ID + helpful/harmful 计数的 bullet、delta 增量、grow-and-refine 剪枝),**既不是纯原始 append-only,也不是纯摘要**。它背书你"反压缩"的方向,不背书你"纯原始无 schema"的极端。
源:arxiv 2510.04618

**⑤ "选哪段喂/压缩"有三类可直接借鉴的成熟解法** — high(见第 4 节展开)
源:arxiv 2304.03442 / 2510.16079 / 2310.08560

**⑥ 已有可对接的跨厂商序列化标准:OpenTelemetry GenAI 语义约定** — high
定义了 `gen_ai.input.messages` / `gen_ai.output.messages` / `gen_ai.system_instructions`,**能捕获完整原始 prompt/completion**(含 roles/parts/tool calls),跨 OpenAI/Anthropic/Bedrock/Vertex 通用,遵循正式 JSON schema——正是归墟想要的"通用可移植序列化"。
**caveat:完整内容捕获默认关闭(opt-in)**,且规范仍处 Development(未 stable)。另有 Letta 的 **agent-file(.af)** 是已落地的可移植 agent 状态格式。
源:opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-spans ; github.com/letta-ai/agent-file

---

## 3. 定位:空位,还是重造轮子?

**归墟踩在一个真空位上,但这个空位被三面包围:**

```
        反压缩(理论)
        ACE 背书你方向 ──┐
                         │
原始留存(哲学)          ▼            完整 trajectory 存储(工程)
Generative Agents ──►【归墟:整段原始 + 运行时注入】◄── LangSmith / OTel
memory stream                │              已经在存,但只用于 debug/eval
(但它检索片段)              │
                            ▼
                    没有人真正做的那一步:
              "把整段原始 trajectory 在运行时注入新会话,
                  迁移人的判断/口吻"  ← 真空白
```

- **不是重造轮子**:存储(LangSmith/OTel)、哲学(memory stream)、理论(ACE)都有人占了,但**把它们接成"整段原始→运行时复用"这条闭环的,没有**。
- **是真空白**:你的核心动词(`use` 整段注入做经验迁移),被核验的来源里零实现。
- **但要清醒**:① 你自己的验证是 **n=1 单点**,外部无人复现或证伪;② 空位空得可能有原因——**所有人都压缩,正因为整段原始撞窗口墙**(你已亲身撞到)。所以这条路的赌注不在"想法新不新",而在"你能不能把'选哪段'这道工程做得比'干脆压缩掉'更值"。

---

## 4. 对你已撞到的"选哪段喂"问题:三类可借鉴解法

调研给出的解法高度一致,正好对应你需要的"聪明图书管理员":

1. **评分式选择性检索(Generative Agents)**
   `score = α·recency(0.995/小时指数衰减) + α·importance(LLM 打分) + α·relevance(query embedding 余弦)`,只注入 top-ranked 且塞得进窗口的部分。
   → **可直接搬**:给 raw/ 里的事件/片段打这三个分,激活时按 query 选 top-k,而不是尾部硬截断那个蠢办法。

2. **用量评分 + 周期剪枝(EvolveR)**
   `s = (成功次数+1)/(使用次数+2)`,低于阈值(0.3)丢弃,top-3 检索。
   → **可借鉴机制(不照搬其压缩)**:用"这段经验被复用后成没成"反过来给片段打分,长期自动浮出真正有用的段。

3. **OS 式分层虚拟上下文(MemGPT)**
   小 core 常驻窗口 + 外部记忆按需 page-in/evict,溢出递归 summarize。
   → **架构参照**:把"整段原始"留在冷层(外部),激活时像 OS 换页一样按需 page-in 相关段,而非一次性全塞。

**这三个正好是你绕不开的 selection / scoring-pruning / tiering 三种工程手段。** 你的"图书管理员"不用从零想,这里有现成范式。

---

## 5. 诚实的 caveats(调研自带,务必看)

1. **没有任何被验证来源真正实现归墟核心命题**(整段原始 runtime 注入做经验/口吻迁移)。空白是真的,但你的 n=1 也是真的孤证。
2. LangSmith"改进循环基础原语"出自 **vendor 营销博客**,别据此认为"已有人做复用 replay"。
3. 多处对比是**解读性投射**:append-only / schemaless / 反压缩是用你的框架去套人家,源材料只描述机制(尤其 Generative Agents 从没用过这些词,且含 reflections 抽象层——②的 2-1 分票就来自此)。
4. **时效**:ACE、EvolveR、OTel GenAI 约定都是 2025–2026 极新、快速演进的材料,半年内可能变。
5. **未覆盖对象**:Mem0、Letta(细节)、Zep、cognee、Memobase、Reflexion、Voyager skill library、Self-Refine、Cursor/CLAUDE.md 这些没进入被核验集合——下一轮要补。

## 6. 还没答的开放问题(留给你分析/下一轮)

- 是否真有项目系统做过"整段原始会话注入"并量化迁移了什么(任务能力 vs 判断/口吻)?这是归墟最该确认是否空白的点。
- 有没有人把可观测性 trace(LangSmith/OTel)直接 replay 进新模型做复用,而非 debug/eval?
- ACE 的"结构化增量保留细节"中间路线 vs 归墟"纯原始 append-only",长上下文下信息保真与成本谁更优?有没有直接对照实验?
- 未覆盖的 Mem0/Letta/Zep/cognee/Reflexion/Voyager 各自在 A–F 上偏哪边,有没有一个偏向原始留存?

---

## 附:全部来源

- A-MEM: arxiv.org/abs/2502.12110
- ExpeL: arxiv.org/pdf/2308.10144
- AWM: arxiv.org/abs/2409.07429
- EvolveR: arxiv.org/html/2510.16079v1
- AutoManual: arxiv.org/pdf/2405.16247
- ACE: arxiv.org/abs/2510.04618
- Generative Agents: arxiv.org/pdf/2304.03442
- MemGPT: arxiv.org/pdf/2310.08560
- LangSmith 改进循环: langchain.com/blog/traces-start-agent-improvement-loop
- LangSmith observability: langchain.com/langsmith/observability
- OTel GenAI 约定: opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-spans/
- Letta agent-file: github.com/letta-ai/agent-file
- Claude Code memory: code.claude.com/docs/en/memory
- Agent Skills 互操作: paperclipped.de/en/blog/agent-skills-open-standard-interoperability/
- Agent memory 厂商全景(blog): agentmarketcap.ai/blog/2026/04/10/agent-memory-vendor-landscape-2026-letta-zep-mem0-langmem
