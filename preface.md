## 序：从一句请返回 JSON，到一份按构造成立的契约

一次提示约定给出一段看起来像 JSON 的文本，一次校验重试把错的挑出来再要一遍，约束解码则让不合法的续写从来没机会被采样。三者都能让下游拿到东西，但保证的强度完全不同：第一种是建议，第二种是兜底，第三种是构造上成立。把这三种混成一个词，是绝大多数结构化输出事故的源头。

这本专题精读写的是这条链路的十二道关：先分三层失败，再走提示约定、格式微调、工具调用、约束解码、自动机层级、文法引擎、Schema 遵循、token 边界、格式税、评测、校验重试，最后落到上线工程。每关回答同一个问题——多上一层保证，换来了什么确定性，又带来什么新代价。

**分工**：与两本 Agent 书不同，那两本讲单 Agent 的循环工具与多 Agent 的协作结构，工具调用只是其中一环；与《AI 代码工程精读》也不同，那本讲代码库里的上下文与验证，这本专讲一次回答的形状怎么被保证、值本身又怎么验真。与《深度研究智能体精读》分工：那本讲检索与引用链路，这本讲输出的最后一段字节怎么落成一个能解析、能校验、语义还对的对象。

## 抽取链路全景图

+----------------------------------------------------------------------+
| STRUCTURED OUTPUT: FROM A DRIFTING JSON TO A CONTRACT                 |
|                                                                      |
| [1] layers -> [2] prompt -> [3] finetune -> [4] tool call             |
| [5] constrained -> [6] automata -> [7] engines -> [8] schema          |
| [9] token -> [10] format tax -> [11] eval -> [12] serving             |
|                                                                      |
| a drifting JSON -> a valid shape -> a checked, meaningful value      |
+----------------------------------------------------------------------+

这张图不是调用顺序，而是「一次回答从会漂移变成能交付时，新出现的保证依次落在哪里」：先分清楚哪层坏，再决定要不要上更强的手段。越往下保证越强，代价与约束也越隐性。

## 十二关速查表

| 关 | 主题 | 一句话 |
|---|---|---|
| 1 | 三层失败 | 语法、schema、语义三套手段，别用一个指标衡量全部 |
| 2 | 提示约定 | 零依赖但有失败面，一到两个示例就够，别堆 few-shot |
| 3 | 格式微调 | 抬高水准，不替代保证，先约后微 |
| 4 | 工具调用 | 各家 schema 不兼容，跨厂商需要抽象层 |
| 5 | 约束解码 | 非法 token 从未可选，输出按构造成立 |
| 6 | 自动机层级 | `FSM` 是正则天花板，嵌套要 `PDA` 的栈 |
| 7 | 引擎 | `XGrammar`、`llguidance`、`Outlines` 各有适用面 |
| 8 | Schema | 接受不等于实现，分开测 declared 与 compliance |
| 9 | token 边界 | `token healing` 防住非典型分词把值质量拖垮 |
| 10 | 格式税 | 把推理字段放前面，别让约束拴住思考 |
| 11 | 评测 | 语法近乎完美、语义远非完美，分开测 |
| 12 | 上线 | 复用 schema、原始 schema 复验、看 p99 延迟 |

## 三层读法：机制、工程、判断

看到一个新方法，先问它属于哪一层。`finite state machine`、`context-free grammar`、`pushdown automaton`、`logit processor` 是机制层，论文讨论形状怎么被构造出来；`Outlines`、`XGrammar`、`llguidance`、`vLLM`、`SGLang`、`llama.cpp` 是工程层，给出可运行的自动机与接入方式；`compliance rate`、`over-constrained`、`under-constrained`、`format tax`、`tail latency` 是判断层，告诉你什么时候这些方法根本不该上、上了又该怎么验。

三层不能互相替代。机制漂亮不等于工程上跑得稳，工程能跑也不等于值就对。综合报告口径建议按「提示约定加校验、再约束解码、最后才微调」的顺序推进，每一步都用自己的 schema 与真实批量做对照，而不是照搬别人的基准分数。

## 一个可验收的出口

读完十二关，出口标准不是记住某个引擎名，而是能对着一个具体抽取任务回答四句话：三层失败里你先防哪一层；约束解码保证的是形状还是值；事后用不用原始 schema 再验一次；推理字段排在答案字段前面了没有。四句都答得出，说明这条链路已经能用于交付，而不只是读过一篇综述。

对做系统的人还要多一句：schema 复用 的命中率决定开销，固定 schema 集缓存全命中，每请求新 schema 则每次付编译成本。这一句答不出来，前面四句答得再好也算不对账。

## 语料边界与口径

一手论文逐字留存，用于机制与术语的精确写法：Let Me Speak Freely、Why is constrained neural language generation particularly challenging、Directed Beam Search、Accurate Online Posterior Alignments、Fast Structured Decoding for Sequence Models、SynCode、JSONSchemaBench、"We Need Structured Output"、JSON Schema Discovery、SGLang、Grammar-Constrained Decoding for Structured NLP、Speculative Decoding 与 StructGPT。官方文档给出接入形态：OpenAI Structured Outputs、Guidance、Pydantic、llama.cpp 的 GBNF 文法、vLLM 结构化输出、JSON Schema 规范、Anthropic 工具使用与 Gemini 结构化输出。

综合报告只补工程判断，比例、成本与趋势一律标注「综合报告口径」。十二关的顺序、场景与速查表属于「延伸，非语料原文」，不加行内代码标记；行内代码只用语料中确实出现的标识符。安全与合规内容只作工程提示，不构成法律意见。