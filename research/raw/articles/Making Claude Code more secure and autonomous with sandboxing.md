在 [Claude Code](https://www.claude.com/product/claude-code) 中，Claude 会与您一起编写、测试和调试代码，浏览您的代码库，编辑多个文件，并运行命令来验证其功能。赋予 Claude 如此高的权限访问您的代码库和文件可能会带来风险，尤其是在提示符注入的情况下。

为了解决这个问题，我们在 Claude Code 中引入了两项基于沙盒技术的新功能。这两项功能旨在为开发者提供更安全的工作环境，同时让 Claude 能够更自主地运行，并减少权限提示。在我们的内部使用中，我们发现沙盒技术能够安全地减少 84% 的权限提示。通过定义 Claude 可以自由工作的边界，这两项功能提高了安全性和自主性。

### 保障 Claude Code 用户安全

Claude Code 采用基于权限的模型：默认情况下，它是只读的，这意味着在进行任何修改或运行任何命令之前，它都会请求权限。但也有一些例外：我们会自动允许像 echo 或 cat 这样的安全命令，但大多数操作仍然需要显式批准。

不断点击“批准”会减慢开发周期，并可能导致“批准疲劳”，用户可能不会密切关注他们正在批准的内容，从而降低开发的安全性。

为了解决这个问题，我们为 Claude Code 推出了沙盒机制。

## 沙盒：一种更安全、更自主的方法

沙盒机制创建了预定义的边界，Claude 可以在其中更自由地工作，而无需为每个操作都请求权限。启用沙盒后，权限提示将大幅减少，安全性也会提高。

我们的沙箱方案建立在操作系统级功能之上，以实现两个边界：

1. **文件系统隔离** 确保 Claude 只能访问或修改特定目录。这对于防止通过提示符注入的 Claude 修改敏感系统文件尤为重要。
2. **网络隔离功能** 确保 Claude 只能连接到已批准的服务器。这可以防止通过提示符注入的 Claude 泄露敏感信息或下载恶意软件。

值得注意的是，有效的沙箱机制需要 *同时* 实现文件系统隔离和网络隔离。如果没有网络隔离，被入侵的代理程序可能会窃取敏感文件，例如 SSH 密钥；如果没有文件系统隔离，被入侵的代理程序则很容易逃逸出沙箱并获得网络访问权限。正是通过同时采用这两种技术，我们才能为 Claude Code 用户提供更安全、更快捷的代理体验。

### 《克劳德代码》新增两项沙盒功能

#### 沙盒式 Bash 工具：无需权限提示即可安全执行 Bash 脚本

我们推出了 [一款全新的沙箱运行时环境](https://docs.claude.com/en/docs/claude-code/sandboxing) ，目前以研究预览版的形式提供测试版。它允许您精确定义代理可以访问哪些目录和网络主机，而无需启动和管理容器，从而避免额外的开销。该环境可用于沙箱化任意进程、代理和 MCP 服务器。此外，它还以 [开源研究预览版的](https://github.com/anthropic-experimental/sandbox-runtime) 形式提供。

在 Claude Code 中，我们使用此运行时环境来隔离 bash 工具，从而允许 Claude 在您设定的限制范围内运行命令。在安全的沙箱内，Claude 可以更自主地运行，并安全地执行命令而无需权限提示。如果 Claude 尝试访问沙箱 *之外* 的内容，您将立即收到通知，并可以选择是否允许。

我们基于操作系统层面的基本机制（例如 [Linux 的 bubblewrap](https://github.com/containers/bubblewrap) 和 macOS 的 seatbelt）构建了此沙箱，以在操作系统层面强制执行这些限制。这些限制不仅涵盖 Claude Code 的直接交互，还包括该命令生成的任何脚本、程序或子进程。如上所述，此沙箱强制执行以下两项：

1. **文件系统隔离，** 允许对当前工作目录进行读写访问，但阻止修改该目录之外的任何文件。
2. **网络隔离** 通过仅允许通过连接到沙箱外部代理服务器的 Unix 域套接字访问互联网来实现。该代理服务器对进程可以连接的域进行限制，并处理新请求域的用户确认。如果您需要进一步提高安全性，我们也支持自定义此代理，以对出站流量强制执行任意规则。

这两个组件都是可配置的：您可以轻松地选择允许或禁止特定的文件路径或域。

![This image illustrations how sandboxing in Claude Code works.](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F0d1c612947c798aef48e6ab4beb7e8544da9d41a-4096x2305.png&w=3840&q=75)

Claude Code 的沙箱架构通过文件系统和网络控制隔离代码执行，自动允许安全操作，阻止恶意操作，并且仅在需要时才请求权限。

沙箱机制确保即使注入攻击成功，攻击者也能完全隔离攻击目标，不会影响用户的整体安全。这样，即使 Claude Code 被攻破，也无法窃取您的 SSH 密钥或连接到攻击者的服务器。

要开始使用此功能，请在 Claude Code 中运行 /sandbox 并查看有关我们安全模型的 [更多技术细节](https://docs.claude.com/en/docs/claude-code/sandboxing) 。

为了方便其他团队构建更安全的代理，我们已将此功能 [开源](https://github.com/anthropic-experimental/sandbox-runtime) 。我们认为，其他团队应该考虑在其代理中采用这项技术，以增强其代理的安全性。

#### 在网络上使用 Claude Code：在云端安全地运行 Claude Code

今天，我们还发布了 [Claude Code 的网页版，](https://docs.claude.com/en/docs/claude-code/claude-code-on-the-web) 使用户能够在云端的隔离沙箱中运行 Claude Code。网页版 Claude Code 会在每个 Claude Code 会话中运行于一个独立的沙箱内，并以安全可靠的方式完全访问其服务器。我们设计的这个沙箱确保敏感凭证（例如 Git 凭证或签名密钥）永远不会与 Claude Code 一起进入沙箱。这样，即使沙箱中运行的代码遭到入侵，用户也不会受到进一步的损害。

Claude Code 的 Web 版本使用自定义代理服务，透明地处理所有 Git 交互。在沙箱环境中，Git 客户端使用自定义的作用域凭据向该服务进行身份验证。代理会验证此凭据和 Git 交互的内容（例如，确保仅推送到已配置的分支），然后在向 GitHub 发送请求之前附加正确的身份验证令牌。

![This illustration depicts how Claude Code on the web uses a custom proxy to handle all git interactions.](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2Fe8f66bcf73d9d23cae67e67776b2d31373c13050-4096x2305.png&w=3840&q=75)

Claude Code's Git integration routes commands through a secure proxy that validates authentication tokens, branch names, and repository destinations—allowing safe version control workflows while preventing unauthorized pushes.

## 入门

我们全新的沙盒式 bash 工具和 Claude Code 在 Web 上为使用 Claude 进行工程工作的开发人员提供了安全性和生产力方面的显著改进。

要开始使用这些工具：

1. Run \`/sandbox\` in Claude and check out [our docs](https://docs.claude.com/en/docs/claude-code/sandboxing) on how to configure this sandbox.
2. 请访问 [claude.com/code](http://claude.ai/redirect/website.v1.b664fe92-5c58-423a-ab9d-779e631ed3bc/code) ，在网页上试用 Claude Code。

或者，如果您正在构建自己的代理，请查看我们的 [开源沙箱代码](https://github.com/anthropic-experimental/sandbox-runtime) ，并考虑将其集成到您的项目中。我们期待看到您的作品。

要了解更多关于 Claude Code 的信息，请查看我们的 [发布博客文章](https://www.anthropic.com/news/claude-code-on-the-web) 。

## 致谢

本文由大卫·德沃肯和奥利弗·韦勒-戴维斯撰写，梅根·崔、凯瑟琳·吴、莫莉·沃沃克、亚历克斯·伊斯肯、基尔·布拉德韦尔和凯文·加西亚亦有贡献。