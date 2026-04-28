在应用人工智能领域，提示工程一直是关注的焦点，而如今，一个新的术语—— **上下文工程——** 开始崭露头角。语言模型的构建不再仅仅是为提示找到合适的词语和短语，而是更多地关注于回答一个更广泛的问题：“什么样的上下文配置最有可能产生我们模型期望的行为？”

**上下文** 指的是从大型语言模型 (LLM) 中采样时所包含的词元集合。当前的 **工程** 问题在于，如何在 LLM 固有的约束条件下优化这些词元的效用，从而持续实现预期结果。有效管理 LLM 通常需要 *从上下文的角度思考* ——换句话说，需要考虑 LLM 在任何给定时间点的整体状态以及该状态可能产生的潜在行为。

在这篇文章中，我们将探讨新兴的上下文工程艺术，并提供一个改进的思维模型来构建可控、有效的代理。

## 情境工程与即时工程

在 Anthropic，我们认为上下文工程是提示工程的自然延伸。提示工程指的是编写和组织 LLM 指令以实现最佳结果的方法（有关概述和有用的提示工程策略，请参阅 [我们的文档](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview) ）。 **上下文工程** 指的是在 LLM 推理过程中，用于管理和维护最佳词元（信息）集的一系列策略，包括提示之外可能出现的所有其他信息。

在早期使用逻辑逻辑模型（LLM）进行工程设计时，提示是人工智能工程工作中最重要的组成部分，因为大多数日常聊天交互之外的应用场景都需要针对一次性分类或文本生成任务优化的提示。顾名思义，提示工程的主要重点在于如何编写有效的提示，尤其是系统提示。然而，随着我们朝着构建功能更强大的智能体方向发展，这些智能体需要在多轮推理和更长的时间跨度内运行，我们需要管理整个上下文状态（系统指令、工具、 [模型上下文协议](https://modelcontextprotocol.io/docs/getting-started/intro) （MCP）、外部数据、消息历史记录等）的策略。

一个循环运行的智能体会生成越来越多的数据，这些数据 *可能* 与下一轮推理相关，而这些信息必须不断进行提炼。上下文工程的 [艺术和科学](https://x.com/karpathy/status/1937902205765607626?lang=en) 在于，如何从不断演进的浩瀚信息海洋中，筛选出哪些信息能够进入有限的上下文窗口。

![Prompt engineering vs. context engineering](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2Ffaa261102e46c7f090a2402a49000ffae18c5dd6-2292x1290.png&w=3840&q=75)

与编写提示的离散任务不同，上下文工程是迭代的，每次我们决定将什么传递给模型时，都会进行内容整理阶段。

## 为什么上下文工程对构建功能强大的智能体至关重要？

尽管语言学习模型（LLM）速度很快，而且能够处理越来越大的数据量，但我们观察到，它们和人类一样，在某个阶段也会失去焦点或感到困惑。对“大海捞针”式基准测试的研究揭示了 [“上下文腐烂”](https://research.trychroma.com/context-rot) 的概念：随着上下文窗口中词元数量的增加，模型准确回忆上下文信息的能力会下降。

虽然有些模型的性能退化程度比其他模型更缓和，但所有模型都存在这一特性。因此，上下文必须被视为一种边际收益递减的有限资源。与 [工作记忆容量有限的](https://journals.sagepub.com/doi/abs/10.1177/0963721409359277) 人类类似，LLM（逻辑学习模型）也存在“注意力预算”，用于解析大量上下文信息。每引入一个新的词元，都会消耗一部分注意力预算，这就增加了 LLM 需要精心筛选可用词元的必要性。

这种注意力稀缺性源于 LLM 的架构限制。LLM 基于 [Transformer 架构](https://arxiv.org/abs/1706.03762) ，该架构允许每个词元 [关注整个上下文中的所有其他词元](https://huggingface.co/blog/Esmail-AGumaan/attention-is-all-you-need) 。这导致 n 个词元之间存在 n²个成对关系。

随着上下文长度的增加，模型捕捉这些成对关系的能力会逐渐捉襟见肘，从而在上下文大小和注意力集中度之间产生一种天然的矛盾。此外，模型的注意力模式是从训练数据分布中发展而来的，而训练数据分布中短序列通常比长序列更为常见。这意味着模型在处理上下文范围内的依赖关系方面经验较少，并且缺乏专门的参数。

[位置编码插值](https://arxiv.org/pdf/2306.15595) 等技术允许模型通过将其适应最初训练的较小上下文来处理更长的序列，尽管这会降低模型对词元位置的理解能力。这些因素造成了性能的梯度变化，而非断崖式下降：模型在较长的上下文中仍然保持着很高的能力，但与在较短上下文中的表现相比，其信息检索和长程推理的精度可能会降低。

这些现实意味着，周密的上下文工程对于构建有能力的智能体至关重要。

## 有效语境的剖析

鉴于低层逻辑模型（LLM）的注意力预算有限， *良好的* 上下文工程意味着找到 *尽可能* *小的* 高信号词元集合，以最大化预期结果的概率。实现这一目标说起来容易做起来难，但在接下来的章节中，我们将概述这一指导原则在上下文的不同组成部分中的实际意义。

**系统提示** 应极其清晰，使用简洁明了的语言，并以 *适合智能体理解的* 方式呈现概念。这种“合适的理解程度”介于两种常见故障模式之间，堪称“黄金地带”。一种极端情况是，工程师在提示中硬编码复杂且脆弱的逻辑，以精确地引导智能体的行为。这种方法会造成系统脆弱性，并随着时间的推移增加维护的复杂性。另一种极端情况是，工程师有时提供模糊不清、过于笼统的指导，无法为逻辑逻辑模型（LLM）提供期望输出的具体信号，或者错误地假设了共享上下文。最佳的理解程度应达到平衡：既要足够具体以有效引导行为，又要足够灵活以提供强大的启发式方法来指导模型的行为。

![Calibrating the system prompt in the process of context engineering.](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F0442fe138158e84ffce92bed1624dd09f37ac46f-2292x1288.png&w=3840&q=75)

一方面，我们看到脆弱的 if-else 硬编码提示；另一方面，我们看到过于笼统或错误地假设共享上下文的提示。

我们建议将提示信息组织成不同的部分（例如 `<background_information>` 、 `<instructions>` 、 `## Tool guidance` 、 `## Output description` 等），并使用 XML 标记或 Markdown 标题等技术来区分这些部分，尽管随着模型功能的增强，提示信息的确切格式可能变得不那么重要了。

无论您如何构建系统提示，都应该力求使用最少的信息完整地描述预期行为。（请注意，最少并不一定意味着简短；您仍然需要预先向代理提供足够的信息，以确保其遵循预期行为。）最好先使用最佳模型测试一个最少的提示，看看它在您的任务上的表现如何，然后根据初始测试中发现的故障模式，添加清晰的说明和示例来改进性能。

**工具** 使智能体能够与其环境互动，并在工作过程中获取新的、额外的上下文信息。由于工具定义了智能体与其信息/行动空间之间的契约，因此工具必须能够提高效率，这体现在两个方面：一是返回高效的信息，二是鼓励智能体采取高效的行为。

In [Writing tools for AI agents – with AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents), we discussed building tools that are well understood by LLMs and have minimal overlap in functionality. Similar to the functions of a well-designed codebase, tools should be self-contained, robust to error, and extremely clear with respect to their intended use. Input parameters should similarly be descriptive, unambiguous, and play to the inherent strengths of the model.

我们最常见的故障模式之一是工具集过于臃肿，涵盖的功能过多，或者导致在选择工具时出现模糊不清的决策点。如果人类工程师都无法明确判断在特定情况下应该使用哪个工具，就不能指望人工智能代理做得更好。正如我们稍后将讨论的，为代理精心挑选一套最小可行工具集，也有助于在长期交互过程中更可靠地维护和精简上下文信息。

Providing examples, otherwise known as few-shot prompting, is a well known best practice that we continue to strongly advise. However, teams will often stuff a laundry list of edge cases into a prompt in an attempt to articulate every possible rule the LLM should follow for a particular task. We do not recommend this. Instead, we recommend working to curate a set of diverse, canonical examples that effectively portray the expected behavior of the agent. For an LLM, examples are the “pictures” worth a thousand words.

Our overall guidance across the different components of context (system prompts**,** tools**,** examples**,** message history, etc) is to be thoughtful and keep your context informative, yet tight. Now let's dive into dynamically retrieving context at runtime.

## 上下文检索和自主搜索

在 [《构建高效的 AI 代理》](https://www.anthropic.com/research/building-effective-agents) 一文中，我们重点阐述了基于 LLM 的工作流与代理之间的区别。自那篇文章发表以来，我们逐渐倾向于采用一个 [简单的代理定义](https://simonwillison.net/2025/Sep/18/agents/) ：LLM 能够在一个循环中自主地使用各种工具。

Working alongside our customers, we’ve seen the field converging on this simple paradigm. As the underlying models become more capable, the level of autonomy of agents can scale: smarter models allow agents to independently navigate nuanced problem spaces and recover from errors.

We’re now seeing a shift in how engineers think about designing context for agents. Today, many AI-native applications employ some form of embedding-based pre-inference time retrieval to surface important context for the agent to reason over. As the field transitions to more agentic approaches, we increasingly see teams augmenting these retrieval systems with “just in time” context strategies.

与预先处理所有相关数据不同，采用“即时”方法构建的智能体维护轻量级标识符（文件路径、存储的查询、网页链接等），并利用这些引用在运行时通过工具动态地将数据加载到上下文中。Anthropic 的智能体编码解决方案 [Claude Code](https://www.anthropic.com/claude-code) 就采用了这种方法，对大型数据库进行复杂的数据分析。该模型可以编写目标查询、存储结果，并利用 Bash 命令（例如 head 和 tail）来分析海量数据，而无需将完整的数据对象加载到上下文中。这种方法类似于人类的认知：我们通常不会记忆整个信息库，而是会引入外部组织和索引系统（例如文件系统、收件箱和书签）来按需检索相关信息。

Beyond storage efficiency, the metadata of these references provides a mechanism to efficiently refine behavior, whether explicitly provided or intuitive. To an agent operating in a file system, the presence of a file named `test_utils.py` in a `tests` folder implies a different purpose than a file with the same name located in `src/core_logic/` Folder hierarchies, naming conventions, and timestamps all provide important signals that help both humans and agents understand how and when to utilize information.

让智能体自主地导航和检索数据，还能实现渐进式信息披露——换句话说，智能体可以通过探索逐步发现相关的上下文。每次交互都会产生上下文信息，为下一步决策提供依据：文件大小暗示复杂性；命名规则暗示用途；时间戳可以作为相关性的参考。智能体可以逐层构建理解，仅在工作内存中保留必要信息，并利用笔记策略来增强信息的持久性。这种自我管理的上下文窗口使智能体能够专注于相关的信息子集，而不是被冗长但可能无关的信息淹没。

Of course, there's a trade-off: runtime exploration is slower than retrieving pre-computed data. Not only that, but opinionated and thoughtful engineering is required to ensure that an LLM has the right tools and heuristics for effectively navigating its information landscape. Without proper guidance, an agent can waste context by misusing tools, chasing dead-ends, or failing to identify key information.

在某些情况下，最有效的智能体可能会采用混合策略：预先检索一些数据以提高速度，然后根据自身判断进行进一步的自主探索。自主程度的“合适”界限取决于具体任务。Claude Code 就是一个采用这种混合模型的智能体： [CLAUDE.md](http://claude.md/) 文件会被预先简单地放入上下文中，而像 glob 和 grep 这样的基本工具则允许它导航环境并即时检索文件，从而有效地绕过了过时的索引和复杂的语法树等问题。

混合策略可能更适合内容动态性较低的场景，例如法律或金融领域。随着模型能力的提升，智能体设计将朝着让智能模型自主行动的方向发展，并逐步减少人工干预。鉴于该领域发展日新月异，“做最简单有效的方法”可能仍将是我们对基于 Claude 构建智能体的团队的最佳建议。

### 针对长期任务的上下文工程

长周期任务要求智能体在一系列动作中保持连贯性、上下文关联性和目标导向行为，即使动作数量超过了 LLM 的上下文窗口。对于持续数十分钟到数小时的连续工作任务，例如大型代码库迁移或综合研究项目，智能体需要专门的技术来克服上下文窗口大小的限制。

等待更大的上下文窗口似乎是一种显而易见的策略。但可以预见的是，在可预见的未来，各种大小的上下文窗口都可能受到上下文污染和信息相关性问题的影响——至少在需要智能体发挥最佳性能的情况下是如此。为了使智能体能够在更长的时间范围内高效工作，我们开发了一些直接解决这些上下文污染限制的技术：压缩、结构化笔记和多智能体架构。

**压实**

压缩是指将接近上下文窗口上限的对话进行内容概括，然后用概括后的内容重新创建一个新的上下文窗口。压缩通常是上下文工程中提升长期连贯性的首要手段。其核心在于以高保真度的方式提炼上下文窗口的内容，从而使智能体能够在性能损失最小的情况下继续运行。

例如，在 Claude Code 中，我们通过将消息历史记录传递给模型来实现这一点，以汇总和压缩最关键的细节。该模型保留了架构决策、未解决的错误和实现细节，同时丢弃了冗余的工具输出或消息。然后，代理可以使用这个压缩后的上下文以及最近访问的五个文件继续执行任务。用户可以获得连续的操作体验，而无需担心上下文窗口的限制。

压缩的艺术在于选择保留哪些信息，舍弃哪些信息，因为过度压缩会导致丢失一些微妙但至关重要的上下文信息，而这些信息的重要性往往在后续处理中才会显现。对于实现压缩系统的工程师，我们建议在处理复杂的代理轨迹时，仔细调整提示信息。首先，最大化召回率，确保压缩提示信息能够捕捉到轨迹中的所有相关信息，然后通过迭代优化，去除冗余内容，从而提高精确度。

清除工具调用和结果就是一个很容易实现的冗余内容示例——一旦某个工具在消息历史记录中被调用过，客服人员为什么还需要再次查看原始结果呢？清除工具结果是最安全、最轻量级的压缩方式之一，最近已作为一项 [功能在 Claude 开发者平台上](https://www.anthropic.com/news/context-management) 推出。

**结构化笔记**

结构化笔记，或称智能体记忆，是一种让智能体定期记录笔记并将其存储在上下文窗口之外的记忆中的技术。这些笔记会在稍后被调回上下文窗口。

这种策略以最小的开销提供持久内存。就像 Claude Code 创建待办事项列表，或者您的自定义代理维护 NOTES.md 文件一样，这种简单的模式使代理能够跟踪复杂任务的进度，并保留关键的上下文和依赖关系，否则这些信息会在数十次工具调用中丢失。

[克劳德玩宝可梦的例子](https://www.twitch.tv/claudeplayspokemon) 展示了记忆如何在非编程领域提升智能体的能力。该智能体能够精确记录数千步游戏进程，追踪诸如“在过去的 1234 步里，我一直在 1 号道路训练我的宝可梦，皮卡丘已经升了 8 级，目标是升到 10 级”之类的目标。无需任何关于记忆结构的提示，它就能绘制已探索区域的地图，记住已解锁的关键成就，并保存战斗策略的笔记，从而学习哪些攻击对付不同的对手最为有效。

上下文重置后，智能体会读取自身的笔记，并继续进行数小时的训练序列或地牢探索。这种贯穿各个概要步骤的连贯性使得智能体能够执行长远策略，而如果仅将所有信息保存在 LLM 的上下文窗口中，则无法实现这一点。

作为 [Sonnet 4.5 发布](http://www.anthropic.com/effective-context-engineering-for-ai-agents) 的一部分，我们在 Claude 开发者平台上推出了 [一款内存工具](http://anthropic.com/news/context-management) 的公开测试版。该工具通过基于文件的系统，使用户能够更轻松地在上下文窗口之外存储和查阅信息。这使得智能体能够随着时间的推移构建知识库，跨会话维护项目状态，并在无需将所有内容都保留在上下文中的情况下引用之前的工作。

**Sub-agent architectures**

子代理架构提供了另一种绕过上下文限制的方法。与其让一个代理尝试维护整个项目的状态，不如让专门的子代理在清晰的上下文窗口中处理特定的任务。主代理负责协调高层计划，而子代理则执行深入的技术工作或使用工具查找相关信息。每个子代理可能进行广泛的探索，使用数万个或更多令牌，但最终只返回其工作的精简摘要（通常包含 1000 到 2000 个令牌）。

This approach achieves a clear separation of concerns—the detailed search context remains isolated within sub-agents, while the lead agent focuses on synthesizing and analyzing the results. This pattern, discussed in [How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system), showed a substantial improvement over single-agent systems on complex research tasks.

The choice between these approaches depends on task characteristics. For example:

- Compaction maintains conversational flow for tasks requiring extensive back-and-forth;
- Note-taking excels for iterative development with clear milestones;
- Multi-agent architectures handle complex research and analysis where parallel exploration pays dividends.

Even as models continue to improve, the challenge of maintaining coherence across extended interactions will remain central to building more effective agents.

## Conclusion

Context engineering represents a fundamental shift in how we build with LLMs. As models become more capable, the challenge isn't just crafting the perfect prompt—it's thoughtfully curating what information enters the model's limited attention budget at each step. Whether you're implementing compaction for long-horizon tasks, designing token-efficient tools, or enabling agents to explore their environment just-in-time, the guiding principle remains the same: find the smallest set of high-signal tokens that maximize the likelihood of your desired outcome.

The techniques we've outlined will continue evolving as models improve. We're already seeing that smarter models require less prescriptive engineering, allowing agents to operate with more autonomy. But even as capabilities scale, treating context as a precious, finite resource will remain central to building reliable, effective agents.

Get started with context engineering in the Claude Developer Platform today, and access helpful tips and best practices via our [memory and context management](https://platform.claude.com/cookbook/tool-use-memory-cookbook) cookbook.

## Acknowledgements

Written by Anthropic's Applied AI team: Prithvi Rajasekaran, Ethan Dixon, Carly Ryan, and Jeremy Hadfield, with contributions from team members Rafi Ayub, Hannah Moran, Cal Rueb, and Connor Jennings. Special thanks to Molly Vorwerck, Stuart Ritchie, and Maggie Vo for their support.