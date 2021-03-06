# 论文笔记——Personalized News Recommendation with Knowledge-aware Interactive Matching

# Personalized News Recommendation with Knowledge-aware Interactive Matching

基于知识感知交互匹配的个性化新闻推荐

## 论文概况

[https://dl.acm.org/doi/abs/10.1145/3404835.3462861](https://dl.acm.org/doi/abs/10.1145/3404835.3462861)

SIGIR '21: Proceedings of the 44th International ACM SIGIR Conference on Research and Development in Information Retrieval 

July 2021 Pages 61–70

[https://doi.org/10.1145/3404835.3462861](https://doi.org/10.1145/3404835.3462861)

自翻：[https://github.com/leeshy-tech/PaperTranslate/blob/main/Personalized_news_recommendation.md](https://github.com/leeshy-tech/PaperTranslate/blob/main/Personalized_news_recommendation_with_%20Knowledge-aware_Interactive_Matching.md)

## 摘要

​	在本文中，我们提出了一种用于新闻推荐的知识感知交互式匹配方法。我们的方法以交互方式对候选新闻和用户兴趣进行建模，以促进它们的准确匹配。我们设计了一个知识感知新闻协同编码器，在知识图谱的帮助下捕获它们在语义和实体中的相关性，交互式地学习点击新闻和候选新闻的表示。我们还设计了一个用户新闻协同编码器来学习候选新闻感知用户兴趣表示和用户感知候选新闻表示，以实现更好的兴趣匹配。在两个真实世界数据集上的实验验证了，我们的方法可以有效地提高新闻推荐的性能。

## 1 引言

​	现有方法通常根据其文本信息对候选新闻进行建模，并以独立的方式从用户的点击历史中推断出用户兴趣[21, 37]。 候选新闻文章可能包含多个方面和实体 [18、33]，并且用户可能有多个兴趣 [32]。 因此，候选新闻和用户兴趣的独立建模可能不如兴趣匹配[31]。

​	在本文中，我们探索更好地模拟候选新闻和用户兴趣之间的相关性，以实现准确的兴趣匹配。 我们的论文受到以下观察的启发。

![image-20220523150639153](/image/Paper/image-20220523150639153.png)

1. 候选新闻可能涵盖不同的方面和实体，并且用户可能有多种兴趣。

   例如，图中 候选新闻2 与篮球明星和政治家有关，含有多个实体（库里和特朗普）。示例用户对车、音乐、体育等领域感兴趣，第二个候选新闻只能匹配特定的用户兴趣即“体育”，但是用户可能只对实体“库里”感兴趣。所以如果独立建模，用候选新闻匹配用户兴趣的效果是较差的。

2. 候选新闻和点击新闻的语义匹配有助于更准确地进行兴趣匹配。

   例如，第二条点击新闻也与第一条候选新闻有语义相关性，因为它们提到了相同的事件。基于这些语义相关性，我们可以推断用户可能对第一个候选新闻感兴趣。

3. 借助知识图谱，点击新闻和候选新闻中实体之间的知识匹配也有助于了解用户对候选新闻的兴趣。

   例如，第 4 条点击新闻中的实体“史蒂夫·科尔”与第 2 条候选新闻中的实体“斯蒂芬·库里”具有内在关联，因为前者和后者分别是“NBA”勇士队的球员和教练。根据知识匹配，我们可以推断出用户可能对第二个候选新闻感兴趣。因此，在语义和知识层面利用点击新闻和候选新闻之间的相关性有利于兴趣匹配。

​	在本文中，我们提出了一种用于个性化新闻推荐的知识感知交互式匹配框架（命名为KIM）。我们的方法可以交互地对候选新闻和用户兴趣进行建模，以学习候选新闻感知的用户兴趣表示和用户感知的候选新闻表示，从而更准确地匹配用户兴趣和候选新闻。在该框架中，我们提出了一种知识协同编码器，借助知识图谱从点击新闻中的实体与候选新闻中的实体之间的相关性来建模用户对候选新闻的兴趣。

​	更具体地说，我们

1. 首先提出了一个**图协同注意网络（ graph co-attention network）**：

   它通过选择和聚合对兴趣匹配有用的邻居，从知识图谱中学习实体的表示。

2. 进一步提出了使用**实体协同注意力网络（ entity co-attention network）**：

   它通过捕获实体之间的相关性来交互式地学习点击新闻和候选新闻的基于知识的表示。

3. 提出了一种**语义协同编码器（ semantic coencoder）**：

   通过对文本之间的语义相关性进行建模，交互式地学习用户点击新闻和候选新闻的基于语义的表示。新闻的统一表示被表述为基于知识和语义的表示的聚合。

4. 进一步提出了一种**用户新闻联合编码器（ user-news co-encoder）**：

   用于从点击新闻和候选新闻的表示中构建候选*新闻感知的用户兴趣表示*和*用户感知的候选新闻表示*，以更好地模拟用户对候选新闻的兴趣。最后，根据候选新闻的表示与用户兴趣之间的相关性对候选新闻进行排名。

我们对两个真实的数据集进行了广泛的实验，并表明我们的方法可以有效地提高新闻推荐的性能并优于其他基准方法。

## 2 相关的工作

​	现有方法通常通过内容对候选新闻进行建模，并根据点击新闻独立建模用户兴趣，然后根据候选新闻和用户兴趣的相关性进行匹配。 只有一部分候选新方面和用户兴趣对匹配用户兴趣和候选新闻有用。然而，这些方法独立地对候选新闻和用户兴趣进行建模，这对于进一步的兴趣匹配可能较差。与这些方法不同的是，在 KIM 方法中，我们提出了一个知识感知的交互式匹配框架，在考虑相关性的情况下对候选新闻和用户兴趣进行交互建模，可以更好地将用户兴趣与候选新闻进行匹配。

​	一些方法以候选感知的方式模拟用户兴趣。事实上，候选新闻可能包含多个方面和实体，其中只有一部分可能与用户兴趣相匹配。然而，这些方法在没有考虑目标用户的情况下对候选新闻进行建模，这对于进一步将用户兴趣与候选新闻进行匹配可能较差。与这些方法不同，我们的 KIM 方法在考虑目标用户的情况下对候选新闻进行建模。此外，这些方法在没有考虑相关性的情况下对点击新闻和候选新闻进行建模，这对于进一步衡量候选新闻和从点击新闻推断出的用户兴趣之间的相关性也可能不是最优的。与这些方法不同，KIM 可以交互地学习点击新闻和候选新闻的表示，以实现更好的兴趣匹配。

## 3 方法论

### 问题表述

​	给定一个用户𝑢和一个候选新闻$n^c$，我们需要计算相关性分数𝑧来衡量用户𝑢对候选新闻内容$n^c$的兴趣。 然后根据相关性得分对不同的候选新闻进行排名并推荐给用户𝑢。 用户𝑢与他/她点击的新闻集相关联。 每个新闻 𝑛 都与其文本 𝑇 和文本中的实体 𝐸 相关联。 此外，还有一个知识图 G 用于提供实体之间的相关性。 它包含实体和实体之间的关系。 G 中的每个实体 𝑒 与其嵌入相关联，e 基于知识图进行预训练。

![image-20220524165026495](/image/Paper/image-20220524165026495.png)

### KIM（ Knowledge-aware Interactive Matching）的框架

​	KIM 包含两个主要模块。：

1. 第一个是知识感知新闻协同编码器（knowledge-aware news co-encoder），它通过捕获语义和知识层面的相关性，交互式地学习用户**点击新闻**和**候选新闻**的**知识感知表示**（ knowledge-aware representations）。

2. 第二个是用户新闻协同编码器（user-news co-encoder），它从上个模块生成的用户**点击新闻的表征**和**候选新闻表征**中交互学习候选新闻感知的用户兴趣表征（ candidate news-aware user interest representation）u和用户感知的候选新闻表征（ user-aware candidate news representation）。

### 知识感知新闻协同编码器（knowledge-aware news co-encoder）

​	它从文本和文本中的实体中交互式地学习用户点击的新闻$n_u$和候选新闻$n_c$的表示，它包含三个子模块。

![image-20220524171536214](/image/Paper/image-20220524171536214.png)

1. 第一个是知识协同编码器（Knowledge co-encoder）

   对于点击新闻$n_u$和候选新闻$n_c$，它基于知识图谱从**实体**之间的相关性中交互式地学习基于知识的表示${k}^{u}$$和 $${k}^{c}$。

   它包含三个组件：

   - 图注意网络（ graph attention network）
   - 图协同注意网络（ graph co-attention network）
   - 实体协同注意网络（ entity co-attention network）

   ![image-20220524173941255](/image/Paper/image-20220524173941255.png)

2. 第二个是语义协同编码器（semantic co-encoder）

   对于点击新闻$n_u$和候选新闻$n_c$，它交互式地学习基于语义的表示${t}^{u}$和${t}^{c}$，以根据**文本**之间的语义相关性来模拟用户对候选新闻的兴趣。

   ![image-20220524175122044](/image/Paper/image-20220524175122044.png)

3. 最后，对于点击新闻$n_u$或候选新闻$n_c$，我们投影其基于**知识**和**语义**的表示来学习统一的新闻表示${n}^{u}$或${n}^{c}$。

### 用户-新闻 协同编码器

​	它从用户点击的新闻和候选新闻的表示中学习候选新闻感知的用户兴趣表示和用户感知的候选新闻表示。

​	通常，用户的兴趣是多样的，只有一部分可以与候选新闻匹配[20]。因此，学习候选**新闻感知的**用户兴趣表示可以更好地建模用户兴趣以匹配候选新闻。

​	类似地，候选新闻可能涵盖多个方面，用户可能只对其中的一部分感兴趣 [33, 34]。因此，学习**用户感知的**候选新闻表示也有利于兴趣匹配。

​	因此，我们应用新闻协同注意网络来学习候选新闻感知用户表示和用户感知候选新闻表示。

## 实验

### 数据集

- MIND2
- Feeds

此外，我们在实验中使用了 WikiData 作为知识图谱。

### 性能评估

​	我们将 KIM 与几种最先进的个性化新闻推荐方法进行比较，如下所示：

- EBNR [21]：通过 GRU 网络从用户的点击历史中表示用户兴趣。
- DKN [32]：将多通道 CNN 网络 [15] 应用于新闻标题中对齐的单词和实体的嵌入，以学习新闻表示。
- DAN [47]：通过 CNN 网络从新闻标题的单词和实体中学习新闻表示，并通过细心的 LSTM 网络 [8] 学习用户兴趣表示。
- NAML [33]：通过多个注意力集中的 CNN 网络从新闻标题、正文、类别和子类别中学习新闻表示。 
- NPA [34]：使用具有个性化注意查询的注意网络来学习新闻和用户表示。 
- LSTUR [1]：通过 GRU 网络从用户最近点击的新闻中建模短期用户兴趣，并通过用户 ID 嵌入建模长期用户兴趣。
- NRMS [37]：通过多头自注意力网络对新闻内容和用户点击行为进行建模。
- FIM [31]：通过CNN网络从用户点击新闻和候选新闻的文本中匹配用户和新闻。
- KRED [18]：通过图注意力网络从新闻中的实体及其在知识图中的邻居中学习新闻的表示。

![image-20220524202107049](/image/Paper/image-20220524202107049.png)

​	我们重复了五次不同的实验，并在表 2 中列出了不同方法的平均性能和相应的标准差。

- KIM 明显优于其他基准方法。

  LSTUR、NRMS和KRED这些方法独立地对候选新闻和用户兴趣进行建模，而不考虑它们的相关性。这是因为用户可能对多个领域感兴趣，并且候选新闻也可能包含多个方面和实体。因此，这些方法很难准确匹配用户兴趣和候选新闻，因为它们是在这些方法中独立建模的。

- KIM 还优于通过考虑候选新闻来建模用户兴趣的基线方法。

  例如 DKN、DAN。这是因为候选新闻可能涵盖多个方面，而用户可能只对其中的一部分感兴趣[33、34]。然而，这些方法在没有考虑目标用户的情况下对候选新闻进行建模，这对于进一步将候选新闻与用户兴趣匹配可能较差。

### 消融实验

​	进行了两项消融研究来评估 KIM 的有效性。首先评估不同信息（即文本和知识）对新闻内容建模的有效性。

![image-20220524202822431](/image/Paper/image-20220524202822431.png)

- 首先，删除语义信息（即新闻文本）会严重损害 KIM 的性能。这是因为文本通常包含有关新闻内容的丰富信息，对于新闻内容的理解至关重要 [45]。去除语义信息会使新闻表示失去很多重要信息，并且不能准确地对新闻内容进行建模。
- 其次，在新闻内容建模中去除知识图谱（即知识知识图谱中的实体及其邻居）也会使 KIM 的性能显著下降。这是因为文本信息通常不足以理解新闻内容 [18, 32]。幸运的是，知识图谱包含不同实体之间的丰富关联。此外，用户点击新闻中的实体与候选新闻之间的相关性可以提供语义信息之外的丰富信息，以了解用户对候选新闻的兴趣。因此，将实体信息纳入个性化新闻推荐有可能提高推荐的准确性。

​	接下来，我们通过分别用注意力网络替换它们来评估 KIM 中几个重要的注意力网络的有效性。

- 首先，在用户新闻协同编码器中去除**新闻关注网络**后，KIM 的性能变得更差。这是因为用户兴趣可能是多样的，并且只有一部分用户点击的新闻对于建模用户兴趣和候选新闻之间的相关性是有用的[32]。此外，候选新闻内容可能包含多个方面，用户可能只对其中的一部分感兴趣。因此，通过新闻共同关注网络学习候选新闻感知用户兴趣和用户感知候选新闻表示可以更好地捕捉用户对候选新闻的兴趣。
- 其次，去除**语义协同注意网络**也会损害 KIM 的性能。这是因为点击新闻和候选新闻之间的语义相关性可以帮助理解用户对候选新闻的兴趣。此外，一条候选新闻或一条点击新闻通常包含多个方面，其中只有一部分对兴趣匹配有用。如果它们的语义信息是独立建模的，则很难在语义级别有效地捕捉点击新闻和候选新闻的相关性。因此，通过语义共同注意网络交互式学习点击新闻和候选新闻的基于语义的表示可以更好地捕捉它们之间的相关性，以便将用户兴趣与候选新闻匹配。
- 同时去除**图协同注意力网络**和**实体协同注意力网络**会导致 KIM 的性能下降。这是因为实体级别的点击新闻和候选新闻之间的相关性对于兴趣匹配也非常有用。此外，如果该方法独立地表示来自其实体的点击新闻和候选新闻，则对于兴趣匹配也是次优的。在KIM方法中，图协同注意力网络和实体协同注意力网络都用于以交互方式捕捉点击新闻和候选新闻实体之间的相关性，可以将丰富的信息融入KIM模型进行兴趣匹配。

## 结论

​	在本文中，我们提出了一种用于个性化新闻推荐的知识感知交互式匹配框架（命名为 KIM）。该框架旨在对候选新闻和用户兴趣进行交互建模，以实现更准确的兴趣匹配。更具体地说，我们首先提出了一个图协同注意网络，通过选择和聚合它们的邻居的信息来基于知识图对实体进行建模，这些信息为兴趣匹配提供了丰富的信息。我们还提出使用实体协同注意网络从实体之间的相关性中交互式地对点击新闻和候选新闻进行建模。此外，我们建议使用语义共同注意网络从文本之间的语义相关性中交互式地对点击新闻和候选新闻进行建模。此外，我们提出了一种用户新闻协同编码器来学习候选新闻感知用户表示和用户感知候选新闻表示，以更好地捕捉用户兴趣和候选新闻之间的相关性。我们对两个真实世界的数据集进行了广泛的实验。实验结果表明，我们的 KIM 方法显著优于其他基准方法。

