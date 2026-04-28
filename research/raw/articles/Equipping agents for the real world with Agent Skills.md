随着模型能力的提升，我们现在可以构建能够与功能齐全的计算环境交互的通用代理。例如， [Claude Code](https://claude.com/product/claude-code) 可以利用本地代码执行和文件系统完成跨领域的复杂任务。但随着这些代理变得越来越强大，我们需要更具可组合性、可扩展性和可移植性的方法，为它们配备特定领域的专业知识。

为智能体构建技能就像为新员工编写入职指南。如今，无需再为每个用例构建零散的、定制化的智能体，任何人都可以通过捕获和共享流程知识，为智能体添加可组合的功能。本文将解释什么是技能，展示其工作原理，并分享构建技能的最佳实践。

![To activate skills, all you need to do is write a SKILL.md file with custom guidance for your agent.](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2Fddd7e6e572ad0b6a943cacefe957248455f6d522-1650x929.jpg&w=3840&q=75)

技能是一个目录，其中包含一个 SKILL.md 文件，该文件包含组织有序的文件夹，其中包含指令、脚本和资源，这些指令、脚本和资源为代理提供额外的功能。

## 技能的剖析

为了更好地了解技能的实际应用，我们来看一个实际例子： [克劳德最近新增的文档编辑功能](https://www.anthropic.com/news/create-files) 就基于这项技能。克劳德已经对 PDF 文件有一定的了解，但直接操作 PDF 文件的能力有限（例如，填写表单）。这项 [PDF 技能](https://github.com/anthropics/skills/tree/main/document-skills/pdf) 让我们能够赋予克劳德这些新能力。

简单来说，技能就是一个包含 `SKILL.md file` 的目录。该文件必须以 YAML 前置元数据开头，其中包含一些必需的元数据： `name` 和 `description` 。启动时，代理会将每个已安装技能的 `name` 和 `description` 预加载到其系统提示符中。

![Anatomy of a SKILL.md file including the relevant metadata: name, description, and context related to the specific actions the skill should take.](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F6f22d8913dbc6228e7f11a41e0b3c124d817b6d2-1650x929.jpg&w=3840&q=75)

SKILL.md 文件必须以 YAML Frontmatter 开头，其中包含文件名和描述，该文件在启动时加载到系统提示符中。

随着技能复杂性的增加，它们可能包含过多的上下文信息，无法全部放入单个 `SKILL.md` 中，或者某些上下文信息仅在特定场景下才相关。在这种情况下，技能可以在技能目录中捆绑其他文件，并在 `SKILL.md` 文件中按名称引用这些文件。这些额外的链接文件构成了 **第三层** （及更高层）的详细信息，Claude 可以根据需要选择浏览和查找这些文件。

在下方所示的 PDF 技能示例中， `SKILL.md` 引用了两个附加文件（ `reference.md` 和 `forms.md` ），技能作者选择将这两个文件与核心的 `SKILL.md` 文件一起打包。通过将表单填写说明移至单独的文件（ `forms.md` ），技能作者可以保持技能核心内容的简洁，并相信 Claude 只会在填写表单时阅读 `forms.md` 文件。

![How to bundle additional content into a SKILL.md file.](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F191bf5dd4b6f8cfe6f1ebafe6243dd1641ed231c-1650x1069.jpg&w=3840&q=75)

你可以通过添加文件将更多上下文信息添加到你的技能中，然后克劳德可以根据系统提示触发该技能。

渐进式披露是使代理技能灵活且可扩展的核心设计原则。就像一本组织良好的手册，从目录开始，然后是具体章节，最后是详细的附录一样，技能允许 Claude 仅在需要时加载信息：

![This image depicts how progressive disclosure of context in Skills.](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2Fa3bca2763d7892982a59c28aa4df7993aaae55ae-2292x673.jpg&w=3840&q=75)

This image depicts how progressive disclosure of context in Skills.

拥有文件系统和代码执行工具的智能体在执行特定任务时，无需将技能的全部内容读取到上下文窗口中。这意味着技能中可以包含的上下文信息量实际上是无限的。

### 技能和上下文窗口

下图显示了当用户消息触发技能时，上下文窗口如何变化。

![This image depicts how skills are triggered in your context window.](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F441b9f6cc0d2337913c1f41b05357f16f51f702e-1650x929.jpg&w=3840&q=75)

技能会在上下文窗口中通过系统提示符触发。

所示操作顺序：

1. 首先，上下文窗口会显示核心系统提示符和每个已安装技能的元数据，以及用户的初始消息；
2. Claude 通过调用 Bash 工具读取 `pdf/SKILL.md` 的内容来触发 PDF 技能；
3. 克劳德选择阅读与该技能捆绑在一起 `forms.md` 文件；
4. 最后，Claude 从 PDF 技能中加载了相关说明后，便开始执行用户的任务。

### 技能和代码执行

技能还可以包括供 Claude 自行执行的工具代码。

大型语言模型在许多任务上表现出色，但某些操作更适合传统的代码执行。例如，通过生成标记来对列表进行排序远比直接运行排序算法成本高得多。除了效率问题之外，许多应用程序还需要只有代码才能提供的确定性可靠性。

在我们的示例中，PDF 技能包含一个预先编写的 Python 脚本，该脚本读取 PDF 文件并提取所有表单字段。Claude 无需将脚本或 PDF 文件加载到上下文中即可运行此脚本。由于代码是确定性的，因此该工作流程具有一致性和可重复性。

![This image depicts how code is executed via Skills.](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2Fc24b4a2ff77277c430f2c9ef1541101766ae5714-1650x929.jpg&w=3840&q=75)

Skills can also include code for Claude to execute as tools at its discretion based on the nature of the task.

## 技能发展与评估

以下是一些有助于入门写作和测试技能的实用指南：

- **首先进行评估：** 通过让智能体执行代表性任务，观察它们在哪些方面遇到困难或需要更多上下文信息，从而找出智能体能力的具体不足之处。然后逐步提升智能体的技能，以弥补这些缺陷。
- **为了便于扩展，结构设计应考虑以下几点：** 当 `SKILL.md` 文件变得过于庞大时，应将其内容拆分为多个单独的文件并分别引用。如果某些上下文互斥或很少同时使用，则保持路径分离可以减少令牌的使用。最后，代码既可以作为可执行工具，也可以作为文档。应该明确 Claude 是应该直接运行脚本，还是应该将其作为参考信息读取到上下文中。
- **站在克劳德的角度思考：** 观察克劳德在实际场景中如何使用你的技能，并根据观察结果进行迭代：注意是否存在意料之外的轨迹或对特定情境的过度依赖。特别注意技能的 `name` 和 `description` 。克劳德会根据这些信息来决定是否在当前任务中触发该技能。
- **与 Claude 迭代：** 在与 Claude 一起完成任务时，请 Claude 将成功的方法和常见错误记录下来，并将其转化为可复用的技能代码和上下文。如果它在使用技能完成任务时偏离了方向，请它进行自我反思，找出问题所在。这个过程将帮助你发现 Claude 实际需要的上下文，而不是试图预先预测它需要什么。

### 使用技能时的安全注意事项

技能通过指令和代码赋予克劳德新的能力。虽然这使其功能强大，但也意味着恶意技能可能会在其使用环境中引入漏洞，或者诱使克劳德窃取数据并执行非预期操作。

我们建议仅从可信来源安装技能。如果从不太可信的来源安装技能，请在使用前进行彻底审核。首先，阅读技能中包含的文件内容，了解其功能，尤其要注意代码依赖关系和捆绑的资源，例如图像或脚本。同样，也要注意技能中指示 Claude 连接到可能不受信任的外部网络资源的指令或代码。

## 技能的未来

[目前](https://www.anthropic.com/news/skills) ， [Claude.ai](http://claude.ai/redirect/website.v1.b664fe92-5c58-423a-ab9d-779e631ed3bc) 、Claude Code、Claude Agent SDK 和 Claude Developer Platform 都支持 Agent Skills。

在接下来的几周里，我们将继续添加功能，以支持技能创建、编辑、发现、共享和使用的完整生命周期。我们尤其兴奋的是，技能能够帮助组织和个人与 Claude 分享他们的上下文和工作流程。我们还将探索如何通过教授代理更复杂的工作流程（涉及外部工具和软件），使技能能够与 [模型上下文协议](https://modelcontextprotocol.io/) (MCP) 服务器互补。

展望未来，我们希望能够让代理自行创建、编辑和评估技能，让他们将自己的行为模式编码成可重用的能力。

技能的概念很简单，格式也同样简单。这种简洁性使得组织、开发人员和最终用户能够更轻松地构建定制化代理并赋予其新的功能。

我们非常期待看到大家用 Skills 创造出什么。立即查看我们的 Skills [文档](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) 和 [使用指南](https://github.com/anthropics/claude-cookbooks/tree/main/skills) ，开始体验吧！

## 致谢

本文由 Barry Zhang、Keith Lazuka 和 Mahesh Murag 撰写，他们都非常喜爱文件夹。特别感谢 Anthropic 的许多其他成员，感谢他们对 Skills 的支持、推广和建设。