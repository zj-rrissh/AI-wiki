---
title: "Contextual Retrieval in AI Systems"
source: "https://www.anthropic.com/engineering/contextual-retrieval"
author:
published:
created: 2026-04-26
description: "Explore how Anthropic enhances AI systems through advanced contextual retrieval methods. Learn about our approach to improving information access and relevance in large language models."
tags:
  - "clippings"
---
人工智能模型要想在特定场景下发挥作用，通常需要获取背景知识。例如，客服聊天机器人需要了解其所服务的特定业务，而法律分析机器人则需要了解大量的过往案例。

开发者通常使用检索增强生成（RAG）来增强人工智能模型的知识。RAG 是一种从知识库中检索相关信息并将其附加到用户提示中的方法，从而显著提升模型的响应能力。然而，传统的 RAG 解决方案在信息编码过程中会丢失上下文信息，这往往会导致系统无法从知识库中检索到相关信息。

本文介绍了一种能够显著提升 RAG 检索步骤的方法。该方法名为“上下文检索”，它采用了两种子技术：上下文嵌入和上下文 BM25。该方法可以将检索失败率降低 49%，与重排序结合使用时，可降低 67%。这些显著的提升意味着检索准确率的大幅提高，进而直接转化为下游任务性能的提升。

您可以借助 [我们的指南，](https://platform.claude.com/cookbook/capabilities-contextual-embeddings-guide) 轻松地使用 Claude 部署您自己的上下文检索解决方案。

### 关于使用更长提示的说明

有时候最简单的解决方案就是最好的。如果你的知识库小于 20 万个词元（大约 500 页材料），你可以直接将整个知识库包含在你给模型的提示中，而无需使用 RAG 或类似方法。

几周前，我们发布了Claude 的 [提示缓存功能](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) ，这使得这种方法速度更快、成本更低。现在，开发人员可以在 API 调用之间缓存常用提示，从而将延迟降低 2 倍以上，成本降低高达 90%（您可以阅读我们的 [提示缓存指南](https://platform.claude.com/cookbook/misc-prompt-caching) 了解其工作原理）。

然而，随着知识库的增长，您将需要更具可扩展性的解决方案。这时，上下文检索就派上了用场。

## RAG入门指南：如何扩展到更大的知识库

对于超出上下文窗口范围的大型知识库，RAG 是一种典型的解决方案。RAG 的工作原理是通过以下步骤对知识库进行预处理：

1. 将知识库（文档“语料库”）分解成更小的文本块，通常不超过几百个词元；
2. 使用嵌入模型将这些块转换为编码含义的向量嵌入；
3. 将这些嵌入存储在向量数据库中，以便按语义相似性进行搜索。

在运行时，当用户向模型输入查询时，向量数据库会根据与查询的语义相似度查找最相关的词块。然后，这些最相关的词块会被添加到发送给生成模型的提示信息中。

虽然词嵌入模型在捕捉语义关系方面表现出色，但它们可能会遗漏关键的精确匹配。幸运的是，有一种较早的技术可以帮助解决这些问题。BM25（Best Matching 25）是一种排名函数，它使用词汇匹配来查找精确的单词或短语匹配项。对于包含唯一标识符或技术术语的查询，它尤其有效。

BM25 的工作原理基于 TF-IDF（词频-逆文档频率）概念。TF-IDF 衡量一个词在文档集中的重要性。BM25 通过考虑文档长度并对词频应用饱和函数来改进 TF-IDF，从而有助于防止常用词主导结果。

以下是 BM25 如何弥补语义嵌入的不足之处：假设用户在技术支持数据库中查询“错误代码 TS-999”。嵌入模型或许能找到关于错误代码的一般性内容，但却可能遗漏“TS-999”这个精确匹配项。而 BM25 则会查找这个特定的文本字符串来识别相关的文档。

RAG 解决方案通过结合嵌入和 BM25 技术，可以更准确地检索最适用的数据块，具体步骤如下：

1. 将知识库（文档“语料库”）分解成更小的文本块，通常不超过几百个词元；
2. 为这些数据块创建 TF-IDF 编码和语义嵌入；
3. 使用 BM25 根据精确匹配查找最佳块；
4. 利用词嵌入技术，根据语义相似性找到最佳词块；
5. 使用排序融合技术合并和去重（3）和（4）的结果；
6. 将前 K 个数据块添加到提示中以生成响应。

通过结合 BM25 和嵌入模型，传统的 RAG 系统可以提供更全面、更准确的结果，在精确的术语匹配和更广泛的语义理解之间取得平衡。

![](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F45603646e979c62349ce27744a940abf30200d57-3840x2160.png&w=3840&q=75)

一种标准检索增强生成（RAG）系统，它同时使用词嵌入和最佳匹配25（BM25）算法来检索信息。TF-IDF（词频-逆文档频率）用于衡量词语的重要性，是BM25算法的基础。

这种方法可以经济高效地扩展到庞大的知识库，远远超出单个提示所能容纳的内容。但这些传统的红黄绿系统有一个重大局限性：它们常常会破坏上下文信息。

### 传统 RAG 中的语境难题

在传统的 RAG 系统中，文档通常会被分割成较小的块以便高效检索。虽然这种方法对许多应用场景都有效，但当单个块缺乏足够的上下文信息时，就会导致问题。

例如，假设你的知识库中嵌入了一系列财务信息（例如，美国证券交易委员会的文件），你收到了以下问题： *“ACME 公司 2023 年第二季度的收入增长是多少？”*

相关的文本片段可能包含以下内容： *“该公司营收较上一季度增长了3%。”* 然而，仅凭这一片段并未具体说明指的是哪家公司或相关的时间段，因此难以检索到正确的信息或有效地利用这些信息。

## 引入上下文检索

上下文检索通过在嵌入之前为每个块添加特定于块的解释性上下文（“上下文嵌入”）并创建 BM25 索引（“上下文 BM25”）来解决此问题。

让我们回到之前提到的美国证券交易委员会（SEC）文件收集示例。以下是一个数据块转换示例：

```
original_chunk = "The company's revenue grew by 3% over the previous quarter."

contextualized_chunk = "This chunk is from an SEC filing on ACME corp's performance in Q2 2023; the previous quarter's revenue was $314 million. The company's revenue grew by 3% over the previous quarter."
```

值得注意的是，过去也曾有人提出过其他利用上下文信息提升检索效果的方法。这些方法包括： [在文档块中添加通用文档摘要](https://aclanthology.org/W02-0405.pdf) （我们进行了实验，发现效果非常有限）、 [假设文档嵌入](https://arxiv.org/abs/2212.10496) 以及 [基于摘要的索引](https://www.llamaindex.ai/blog/a-new-document-summary-index-for-llm-powered-qa-systems-9a32ece2f9ec) （我们进行了评估，发现性能不佳）。这些方法与本文提出的方法有所不同。

### 实现上下文检索

当然，手动标注知识库中成千上万甚至数百万个文本块的工作量实在太大了。为了实现上下文检索，我们采用了 Claude 模型。我们编写了一个提示，指示模型提供简洁的、特定于文本块的上下文，并结合整个文档的上下文来解释该文本块。我们使用以下 Claude 3 Haiku 提示来为每个文本块生成上下文：

```
<document> 
{{WHOLE_DOCUMENT}} 
</document> 
Here is the chunk we want to situate within the whole document 
<chunk> 
{{CHUNK_CONTENT}} 
</chunk> 
Please give a short succinct context to situate this chunk within the overall document for the purposes of improving search retrieval of the chunk. Answer only with the succinct context and nothing else.
```

生成的上下文文本（通常为 50-100 个词元）会在嵌入数据块之前以及创建 BM25 索引之前添加到数据块的前面。

以下是实际的预处理流程：

![](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F2496e7c6fedd7ffaa043895c23a4089638b0c21b-3840x2160.png&w=3840&q=75)

上下文检索是一种预处理技术，可以提高检索准确率。

[如果您对使用上下文检索感兴趣，可以从我们的操作指南](https://platform.claude.com/cookbook/capabilities-contextual-embeddings-guide) 开始 。

### 利用提示缓存降低上下文检索的成本

得益于我们前面提到的特殊提示缓存功能，Claude 能够以极低的成本实现上下文检索。有了提示缓存，您无需为每个数据块都传入参考文档。您只需将文档加载到缓存中一次，然后引用之前缓存的内容即可。假设有 800 个 token 数据块、8000 个 token 文档、50 条 token 上下文指令，并且每个数据块包含 100 个 token 的上下文信息，那么 **生成上下文数据块的一次性成本为每百万个文档 token 1.02 美元** 。

#### 方法论

[我们针对不同的知识领域（代码库、小说、arXiv 论文、科学论文）、嵌入模型、检索策略和评估指标进行了实验。附录二](https://assets.anthropic.com/m/1632cded0a125333/original/Contextual-Retrieval-Appendix-2.pdf) 列出了我们针对每个领域使用的一些问答示例 。

下图展示了在所有知识领域中使用性能最佳的嵌入配置（Gemini Text 004）并检索前 20 个数据块的平均性能。我们使用 1 减去 recall@20 作为评估指标，该指标衡量的是未能在前 20 个数据块中检索到的相关文档的百分比。您可以在附录中查看完整结果——上下文关联在我们评估的每种嵌入源组合中都提高了性能。

#### 性能提升

我们的实验表明：

- **上下文嵌入将前 20 个数据块的检索失败率降低了 35%** （5.7% → 3.7%）。
- **结合上下文嵌入和上下文 BM25，前 20 个数据块检索失败率降低了 49%** （5.7% → 2.9%）。

![](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F7f8d739e491fe6b3ba0e6a9c74e4083d760b88c9-3840x2160.png&w=3840&q=75)

结合上下文嵌入和上下文 BM25，可将前 20 个数据块检索失败率降低 49%。

#### 实施注意事项

在实现上下文检索时，需要注意以下几点：

1. **数据块边界：** 考虑如何将文档分割成数据块。数据块大小、数据块边界和数据块重叠的选择会影响检索性能 <sup><font dir="auto"><font dir="auto">¹</font></font></sup> 。
2. **嵌入模型：** 虽然上下文检索提升了我们测试的所有嵌入模型的性能，但某些模型可能受益更多。我们发现 [Gemini](https://ai.google.dev/gemini-api/docs/embeddings) 和 [Voyage](https://www.voyageai.com/) 嵌入模型尤其有效。
3. **自定义上下文提示：** 虽然我们提供的通用提示效果不错，但如果您使用针对特定领域或用例量身定制的提示（例如，包含可能仅在知识库的其他文档中定义的关键术语词汇表），则可能会获得更好的结果。
4. **信息块数量：** 在上下文窗口中添加更多信息块可以提高包含相关信息的概率。但是，过多的信息可能会分散模型的注意力，因此信息块的数量应该有所限制。我们尝试了 5 个、10 个和 20 个信息块，发现 20 个信息块的性能最佳（详见附录中的对比），但建议您根据实际使用情况进行实验。

**始终运行 evals：** 通过向其传递上下文块并区分什么是上下文什么是块，可以改进响应生成。

## 通过重新排名进一步提升性能

In a final step, we can combine Contextual Retrieval with another technique to give even more performance improvements. In traditional RAG, the AI system searches its knowledge base to find the potentially relevant information chunks. With large knowledge bases, this initial retrieval often returns a lot of chunks—sometimes hundreds—of varying relevance and importance.

Reranking is a commonly used filtering technique to ensure that only the most relevant chunks are passed to the model. Reranking provides better responses and reduces cost and latency because the model is processing less information. The key steps are:

1. Perform initial retrieval to get the top potentially relevant chunks (we used the top 150);
2. Pass the top-N chunks, along with the user's query, through the reranking model;
3. Using a reranking model, give each chunk a score based on its relevance and importance to the prompt, then select the top-K chunks (we used the top 20);
4. Pass the top-K chunks into the model as context to generate the final result.

![](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F8f82c6175a64442ceff4334b54fac2ab3436a1d1-3840x2160.png&w=3840&q=75)

Combine Contextual Retrieva and Reranking to maximize retrieval accuracy.

### Performance improvements

There are several reranking models on the market. We ran our tests with the [Cohere reranker](https://cohere.com/rerank). Voyage [also offers a reranker](https://docs.voyageai.com/docs/reranker), though we did not have time to test it. Our experiments showed that, across various domains, adding a reranking step further optimizes retrieval.

Specifically, we found that Reranked Contextual Embedding and Contextual BM25 reduced the top-20-chunk retrieval failure rate by 67% (5.7% → 1.9%).

![](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F93a70cfbb7cca35bb8d86ea0a23bdeeb699e8e58-3840x2160.png&w=3840&q=75)

Reranked Contextual Embedding and Contextual BM25 reduces the top-20-chunk retrieval failure rate by 67%.

#### Cost and latency considerations

One important consideration with reranking is the impact on latency and cost, especially when reranking a large number of chunks. Because reranking adds an extra step at runtime, it inevitably adds a small amount of latency, even though the reranker scores all the chunks in parallel. There is an inherent trade-off between reranking more chunks for better performance vs. reranking fewer for lower latency and cost. We recommend experimenting with different settings on your specific use case to find the right balance.

## Conclusion

We ran a large number of tests, comparing different combinations of all the techniques described above (embedding model, use of BM25, use of contextual retrieval, use of a reranker, and total # of top-K results retrieved), all across a variety of different dataset types. Here’s a summary of what we found:

1. Embeddings+BM25 is better than embeddings on their own;
2. Voyage and Gemini have the best embeddings of the ones we tested;
3. Passing the top-20 chunks to the model is more effective than just the top-10 or top-5;
4. Adding context to chunks improves retrieval accuracy a lot;
5. Reranking is better than no reranking;
6. **所有这些好处叠加起来** ：为了最大限度地提高性能，我们可以将上下文嵌入（来自 Voyage 或 Gemini）与上下文 BM25 结合，再加上重新排序步骤，并将 20 个块添加到提示中。

我们鼓励所有使用知识库的开发人员使用 [我们的操作指南](https://platform.claude.com/cookbook/capabilities-contextual-embeddings-guide) 来尝试这些方法，以释放新的性能水平。

## 附录一

以下是按数据集、嵌入提供商、除嵌入外还使用 BM25、使用上下文检索以及对检索 @ 20 进行重排序的结果细分。

有关第 10 次和第 5 次检索的详细分类，以及每个数据集的示例问题和答案， 请参阅 [附录 II 。](https://assets.anthropic.com/m/1632cded0a125333/original/Contextual-Retrieval-Appendix-2.pdf)

![](https://www.anthropic.com/_next/image?url=https%3A%2F%2Fwww-cdn.anthropic.com%2Fimages%2F4zrzovbb%2Fwebsite%2F646a894ec4e6120cade9951a362f685cd2ec89b2-2458x2983.png&w=3840&q=75)

1 减去跨数据集和嵌入提供商的 20 个结果的召回率。

## 致谢

本文由丹尼尔·福特撰写并研究。感谢奥罗瓦·西克德、高塔姆·米塔尔和肯尼斯·利恩提供的宝贵意见，感谢塞缪尔·弗拉米尼负责食谱的实施，感谢劳伦·波兰斯基负责项目协调，以及感谢亚历克斯·阿尔伯特、苏珊·佩恩、斯图尔特·里奇和布拉德·艾布拉姆斯对本文的润色。