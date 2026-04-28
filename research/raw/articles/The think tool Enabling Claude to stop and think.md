- 扩展思维更新
	2025年12月15日
	自最初发布以来，扩展思维功能已得到改进，因此我们建议在大多数情况下使用该功能，而不是专门的思考工具。扩展思维功能提供类似的优势——让 Claude 有空间来推理复杂问题——并且集成度更高，性能更佳。有关实现细节，请参阅我们的扩展思维功能文档。

为了不断提升克劳德解决复杂问题的能力，我们发现了一种特别有效的方法：一种“思考”工具，可以在执行复杂任务时为结构化思考创造专门的空间。

这项简单而强大的技术——正如我们将在下文解释的，它与克劳德新获得的“ [扩展思维](https://www.anthropic.com/research/visible-extended-thinking) ”能力有所不同（ [扩展思维的实现细节](https://platform.claude.com/docs/en/build-with-claude/extended-thinking) 请参见此处）——显著提升了克劳德的智能体工具使用能力。这包括遵循策略、做出一致的决策以及处理多步骤问题，所有这些都只需极少的实现开销。

在这篇文章中，我们将探讨如何在不同的应用程序中实现“思考”工具，并根据经过验证的基准测试结果，为开发人员提供实用的指导。

### 什么是“思考”工具？

借助“思考”工具，我们让 Claude 能够在得出最终答案的过程中，增加一个额外的思考步骤——并为其分配一个专门的空间。

虽然听起来与扩展思考相似，但两者是不同的概念。扩展思考是指 Claude 在开始生成响应之前所做的操作。通过扩展思考，Claude 会在采取行动之前深入思考并反复迭代其计划。“思考”工具的作用是，在 Claude 开始生成响应之后，增加一个步骤，使其停下来思考是否已掌握继续执行所需的所有信息。这在执行大量工具调用或与用户进行冗长的多步骤对话时尤为有用。

这使得“思考”工具更适用于克劳德无法仅凭用户查询获得所有必要信息来形成响应，并且需要处理外部信息（例如工具调用结果中的信息）的情况。克劳德使用“思考”工具进行的推理不如通过扩展思考所能获得的推理全面，而是更侧重于 模型发现的 *新信息。*

我们建议在较为简单的工具使用场景（例如非顺序工具调用或简单的指令执行）中使用扩展思维。扩展思维也适用于编码、数学和物理等无需 Claude 调用工具的用例。“思考”工具更适合以下情况：Claude 需要调用复杂工具、仔细分析长链工具调用中的工具输出、在包含详细指南的策略密集型环境中导航，或者做出每一步都建立在前一步之上且错误代价高昂的顺序决策。

以下是使用 [τ-Bench](https://arxiv.org/abs/2406.12045) 标准工具规范格式的示例实现：

```
{
  "name": "think",
  "description": "Use the tool to think about something. It will not obtain new information or change the database, but just append the thought to the log. Use it when complex reasoning or some cache memory is needed.",
  "input_schema": {
    "type": "object",
    "properties": {
      "thought": {
        "type": "string",
        "description": "A thought to think about."
      }
    },
    "required": ["thought"]
  }
}
```

### τ-Bench 性能测试

我们使用 τ-bench（tau-bench）对“think”工具进行了评估。τ-bench 是一个综合基准测试，旨在测试模型在真实的客户服务场景中使用工具的能力，“think”工具是评估标准环境的一部分。

τ-bench 评估 Claude 的以下能力：

- 与模拟用户进行真实的对话
- 始终遵循复杂的客户服务代理政策准则
- 使用各种工具访问和操作环境数据库

τ-bench 中使用的主要评估指标是 pass^ *k* ，它衡量的是给定任务的所有 *k 次* 独立试验均成功的概率，并取所有任务的平均值。与其他 LLM 评估中常用的 pass@ *k指标（衡量* *k* 次试验中至少有一次 成功）不同，pass^ *k* 评估的是一致性和可靠性——这对于客户服务应用至关重要，因为在这些应用中，始终如一地遵守策略至关重要。

#### 绩效分析

我们的评估对比了几种不同的配置：

1. 基线（无“思考”工具，无扩展思考模式）
2. 单独的扩展思维模式
3. 单独使用“思考”工具
4. 带有优化提示的“思考”工具（适用于航空领域）

结果显示，当 Claude 3.7 在基准测试的“航空”和“零售”客户服务领域有效使用“思考”工具时，客户服务水平都得到了显著提高：

- **航空公司领域** ：经过优化的提示的“思考”工具在 pass^1 指标上达到了 0.570，而基准指标仅为 0.370——相对提高了 54%；
- **零售领域** ：仅“思考”工具就达到了 0.812，而基准值为 0.783。

![一张折线图显示了 Claude 3.7 Sonnet 在 Tau-Bench 评估的“航空公司”域上的性能。](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2Fff91e5c84be59ae71306bcc60adba9affed86484-2200x1300.jpg&w=3840&q=75)

Claude 3.7 Sonnet 在 Tau-Bench 评估的“航空公司”域中四种不同配置下的性能。

Claude 3.7 Sonnet 在 Tau-Bench 评估的“Airline”域上的表现

| 配置 | *k* \=1 | *k* \= 2 | *k* \= 3 | *k* \= 4 | *k* \= 5 |
| --- | --- | --- | --- | --- | --- |
| “思考” + 提示 | 0.584 | 0.444 | 0.384 | 0.356 | 0.340 |
| “思考” | 0.404 | 0.254 | 0.186 | 0.140 | 0.100 |
| 拓展思维 | 0.412 | 0.290 | 0.232 | 0.192 | 0.160 |
| 基线 | 0.332 | 0.206 | 0.148 | 0.116 | 0.100 |

四种不同配置下的评估结果。分数以比例形式呈现。

在航空领域，最佳性能的实现方式是将“思考”工具与优化后的提示相结合，该提示提供了分析客户请求时可使用的推理方法示例。以下是优化后的提示示例：

```
## Using the think tool

Before taking any action or responding to the user after receiving tool results, use the think tool as a scratchpad to:
- List the specific rules that apply to the current request
- Check if all required information is collected
- Verify that the planned action complies with all policies
- Iterate over tool results for correctness 

Here are some examples of what to iterate over inside the think tool:
<think_tool_example_1>
User wants to cancel flight ABC123
- Need to verify: user ID, reservation ID, reason
- Check cancellation rules:
  * Is it within 24h of booking?
  * If not, check ticket class and insurance
- Verify no segments flown or are in the past
- Plan: collect missing info, verify rules, get confirmation
</think_tool_example_1>

<think_tool_example_2>
User wants to book 3 tickets to NYC with 2 checked bags each
- Need user ID to check:
  * Membership tier for baggage allowance
  * Which payments methods exist in profile
- Baggage calculation:
  * Economy class × 3 passengers
  * If regular member: 1 free bag each → 3 extra bags = $150
  * If silver member: 2 free bags each → 0 extra bags = $0
  * If gold member: 3 free bags each → 0 extra bags = $0
- Payment rules to verify:
  * Max 1 travel certificate, 1 credit card, 3 gift cards
  * All payment methods must be in profile
  * Travel certificate remainder goes to waste
- Plan:
1. Get user ID
2. Verify membership level for bag fees
3. Check which payment methods in profile and if their combination is allowed
4. Calculate total: ticket price + any bag fees
5. Get explicit confirmation for booking
</think_tool_example_2>
```

尤其有趣的是不同方法之间的比较。使用带有优化提示的“思考”工具，其结果明显优于扩展思考模式（后者与未提示的“思考”工具表现相近）。单独使用“思考”工具（不带提示）虽然比基线水平有所提高，但仍不及优化后的方法。

将“思考”工具与优化提示相结合，取得了最强的性能，优势非常明显，这可能是因为基准测试中 [航空公司政策](https://github.com/sierra-research/tau-bench/blob/main/tau_bench/envs/airline/wiki.md) 部分的高度复杂性，而该模型从“思考”的示例中受益最大。

在零售领域，我们也测试了各种配置，以了解每种方法的具体影响。

![折线图显示了 Claude 3.7 Sonnet 在 Tau-Bench 评估的“零售”领域中的表现](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F5819616b4cc109d30f1a7d47ec8a32a6b839637b-7638x4513.jpg&w=3840&q=75)

Claude 3.7 Sonnet 在 Tau-Bench 评估的“零售”域中三种不同配置下的性能。

Claude 3.7 Sonnet 在 Tau-Bench 评估的“零售”领域中的表现

| 配置 | *k* \=1 | *k* \= 2 | *k* \= 3 | *k* \= 4 | *k* \= 5 |
| --- | --- | --- | --- | --- | --- |
| “思考” + 无提示 | 0.812 | 0.735 | 0.685 | 0.650 | 0.626 |
| 拓展思维 | 0.770 | 0.681 | 0.623 | 0.581 | 0.548 |
| 基线 | 0.783 | 0.695 | 0.643 | 0.607 | 0.583 |

三种不同配置下的评估结果。分数以比例形式呈现。

即使没有额外的提示，“思考”工具也取得了最高的 pass^1 分数 0.812。与航空领域相比， [零售政策](https://github.com/sierra-research/tau-bench/blob/main/tau_bench/envs/retail/wiki.md) 明显更容易理解，克劳德仅仅通过一个思考的空间就取得了进步，无需进一步指导。

#### τ-Bench 分析的关键见解

我们的详细分析揭示了几种可以帮助您有效运用“思考”工具的模式：

1. **提示在难题领域至关重要** 。仅仅提供“思考”工具或许能略微提升表现，但将其与优化的提示相结合，在难题领域能取得显著更佳效果。然而，对于较简单的领域，仅仅提供“思考”工具可能就足够了。
2. **各次试验的一致性有所提高** 。使用“think”带来的改进在 pass^k 直至 k=5 时均得以保持，这表明该工具帮助 Claude 更有效地处理了极端情况和异常场景。

### SWE-Bench 性能测试

在评估 Claude 3.7 Sonnet 时，我们在 SWE-bench 设置中添加了一个类似的“思考”工具，这使得我们获得了 0.623 的最先进分数。以下是经过调整的“思考”工具的定义：

```
{
  "name": "think",
  "description": "Use the tool to think about something. It will not obtain new information or make any changes to the repository, but just log the thought. Use it when complex reasoning or brainstorming is needed. For example, if you explore the repo and discover the source of a bug, call this tool to brainstorm several unique ways of fixing the bug, and assess which change(s) are likely to be simplest and most effective. Alternatively, if you receive some test results, call this tool to brainstorm ways to fix the failing tests.",
  "input_schema": {
    "type": "object",
    "properties": {
      "thought": {
        "type": "string",
        "description": "Your thoughts."
      }
    },
    "required": ["thought"]
  }
}
```

我们的实验（ *n* \= 30 个使用“思考”工具的样本， *n* \= 144 个不使用“思考”工具的样本）表明，单独使用该工具平均可使性能提高 1.6%（Welch *t* 检验： *t* (38.89) = 6.71， *p* <.001， *d* \= 1.47）。

### 何时使用“思考”工具

根据这些评估结果，我们确定了克劳德最能从“思考”工具中受益的具体场景：

1. **工具输出分析。** 当 Claude 需要仔细处理先前工具调用的输出结果后再采取行动，并且可能需要回溯其方法时；
2. **策略繁多的环境** 。当克劳德需要遵循详细的指导方针并验证合规性时；
3. **顺序决策** 。当每个行动都建立在先前行动的基础上，并且错误代价高昂时（常见于多步骤领域）。

## 实施最佳实践

为了充分利用 Claude 的“think”工具，我们根据 τ-bench 实验推荐以下实施实践。

#### 1\. 利用特定领域示例进行策略性提示

最有效的方法是提供关于何时以及如何使用“思考”工具的清晰说明，例如用于 τ-bench 航空公司领域的工具。提供针对您具体用例的示例可以显著提高模型使用“思考”工具的效率：

- 推理过程中所需的细节程度；
- 如何将复杂的指令分解成可执行的步骤；
- 用于处理常见场景的决策树；以及
- 如何检查是否已收集所有必要信息。

#### 2\. 在系统提示中添加复杂指导

我们发现，当指令较长或较为复杂时，在系统提示中包含关于“思考”工具的说明比将其直接放在工具描述中更为有效。这种方法提供了更广泛的上下文，并有助于模型更好地将思考过程融入其整体行为中。

### 何时不应使用“思考”工具

虽然“思考”工具可以带来显著的改进，但它并不适用于所有工具使用场景，而且会增加提示符的长度和输出标记的数量。具体来说，我们发现“思考”工具在以下使用场景中没有任何改进：

1. **非顺序工具调用** 。如果 Claude 只需要进行一次工具调用或多次并行调用即可完成任务，那么添加“思考”功能不太可能带来任何改进。
2. **简单的指令如下** 。当 Claude 需要遵守的限制不多，而且其默认行为已经足够好时，额外的“思考”不太可能带来任何好处。

### 入门

“思考”工具是 Claude 实现的一个简单易用的补充，只需几个步骤即可带来显著的改进：

1. **使用智能体工具的使用场景进行测试。** 首先从具有挑战性的使用案例入手——例如 Claude 目前在策略遵守或处理冗长工具调用链中的复杂推理方面遇到的困难。
2. **添加工具定义** 。实现一个针对您领域定制的“思考”工具。它只需要极少的代码，就能实现更结构化的推理。此外，请考虑在系统提示中包含关于何时以及如何使用该工具的说明，并提供与您领域相关的示例。
3. **观察并改进** 。观察克劳德在实践中如何使用该工具，并调整你的提示，以鼓励更有效的思维模式。

最棒的是，添加此工具对性能的影响微乎其微。除非 Claude 决定使用它，否则它不会改变外部行为，也不会干扰您现有的工具或工作流程。

### 结论

我们的研究表明，“思考”工具能够显著提升 Claude 3.7 Sonnet 在复杂任务上的性能 <sup><font dir="auto"><font dir="auto">¹，</font></font></sup> 尤其是在需要遵循策略和进行推理的长链工具调用任务中。“思考”并非万能解决方案，但它在合适的用例中能够带来显著的优势，且实现复杂度极低。

我们期待看到您将如何使用“思考”工具与 Claude 一起构建更强大、更可靠、更透明的 AI 系统。

1\. 虽然我们的 τ-Bench 结果主要集中在使用“think”工具改进 Claude 3.7 Sonnet 上，但我们的实验表明，Claude 3.5 Sonnet（新版）在与 3.7 Sonnet 相同的配置下也能获得性能提升，这表明这种改进也适用于其他 Claude 模型。