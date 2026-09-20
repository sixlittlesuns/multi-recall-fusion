# multi-recall-fusion

RQ1：用户行为稀疏度变化时，不同召回方法的性能是否存在明显差异？

RQ2：多路召回能否提升不同用户群体的候选覆盖能力？

### 数据集

> 来源：[GitHub - chongminggao/KuaiRec: KwaiRec: A Fully-observed Dataset for Recommender Systems. · GitHub](https://github.com/chongminggao/KuaiRec)

1. `big_matrix.csv` and `small_matrix.csv`

| Column         | Explanation                 |
| -------------- | --------------------------- |
| user_id        | The ID of the user.         |
| video_id       | The ID of the viewed video. |
| play_duration  | 视频观看时间                      |
| video_duration | 视频时长                        |
| timestamp      | Unix timestamp              |
| watch_ratio    | 观看比例                        |

2. `kuairec_caption_category` 

| Column                     | Explanation                      |
| -------------------------- | -------------------------------- |
| video_id                   | The ID of the viewed video.      |
| manual_cover_text          | 封面文字                             |
| caption                    | 简介标题                             |
| topic_tag                  | Tags of the topics of this video |
| first_level_category_name  | 一级类别名称                           |
| second_level_category_name | 二级类别名称                           |
| third_level_category_name  | 三级类别名称                           |

### 划分数据集

- Leave-One-Out

- Global Temporal Split

> 参考：[Time to Split: Exploring Data Splitting Strategies for Offline Evaluation of Sequential Recommenders](https://dl.acm.org/doi/full/10.1145/3705328.3748164)

### 结果与分析

#### 实验一：

1. Popular Recall

| user group | recall@50 | recall@100 | recall@200 | coverage |
| ---------- | --------- | ---------- | ---------- | -------- |
| Cold       | 0.002126  | 0.002894   | 0.004632   | 0.050044 |
| Medium     | 0.005534  | 0.008984   | 0.015080   | 0.103505 |
| Warm       | 0.006121  | 0.010251   | 0.020655   | 0.187059 |

2. ItemCF

| user group | recall@50 | recall@100 | recall@200 | coverage |
| ---------- | --------- | ---------- | ---------- | -------- |
| Cold       | 0.015218  | 0.021069   | 0.031801   | 0.203207 |
| Medium     | 0.008371  | 0.021155   | 0.053136   | 0.124720 |
| Warm       | 0.019187  | 0.041055   | 0.083724   | 0.095544 |

3. ANN

| user group | recall@50 | recall@100 | recall@200 | coverage |
| ---------- | --------- | ---------- | ---------- | -------- |
| Cold       | 0.004987  | 0.009710   | 0.015892   | 0.474757 |
| Medium     | 0.005773  | 0.011578   | 0.020159   | 0.346561 |
| Warm       | 0.005690  | 0.010692   | 0.019542   | 0.251102 |

#### 实验二：多路召回

$$
Score(i) = \frac{w_P}{k + r_P(i)} + \frac{w_C}{k + r_C(i)} + \frac{w_A}{k + r_A(i)}
$$

- $w_P$,$w_C$,$w_A$: Popular 权重,ItemCF 权重,ANN 权重
- $r(i)$: 物品在对应召回列表中的 rank
- `rrf_k=5`
1. Equal-weight RRF ($w_P$=$w_C$=$w_A$)

| user group | recall@50 | recall@100 | recall@200 | coverage |
| ---------- | --------- | ---------- | ---------- | -------- |
| Cold       | 0.009605  | 0.016117   | 0.026131   | 0.437390 |
| Medium     | 0.007091  | 0.013295   | 0.028210   | 0.316358 |
| Warm       | 0.010412  | 0.020802   | 0.042761   | 0.231592 |

2. Weighted RRF

| Weight_ratio | user_group | num_users | Recall@50 | Recall@100 | Recall@200 | Coverage |
| ------------ | ---------- | --------- | --------- | ---------- | ---------- | -------- |
| 1:2:1        | Cold       | 2378      | 0.012104  | 0.019552   | 0.027994   | 0.418100 |
| 1:2:1        | Medium     | 2287      | 0.007445  | 0.015241   | 0.034991   | 0.279652 |
| 1:2:1        | Warm       | 2216      | 0.012955  | 0.026457   | 0.055777   | 0.226190 |
| 1:3:1        | Cold       | 2378      | 0.013428  | 0.020497   | 0.029241   | 0.405423 |
| 1:3:1        | Medium     | 2287      | 0.007572  | 0.016353   | 0.039733   | 0.264440 |
| 1:3:1        | Warm       | 2216      | 0.014528  | 0.029821   | 0.063038   | 0.207782 |
| 2:3:2        | Cold       | 2378      | 0.011011  | 0.018406   | 0.027131   | 0.426146 |
| 2:3:2        | Medium     | 2287      | 0.007237  | 0.014298   | 0.031800   | 0.288470 |
| 2:3:2        | Warm       | 2216      | 0.011818  | 0.024225   | 0.050381   | 0.235560 |

不同召回通道具有一定互补性。引入 ANN 后，融合方法能够显著扩大候选覆盖范围；进一步提高 ItemCF 权重后，Recall@K 得到恢复，但 Coverage 相应下降，表明多路召回融合存在明显的 Recall-Coverage 权衡。

### 结论

针对 RQ1，实验结果表明，用户行为稀疏度变化会导致不同召回方法表现出明显差异。ItemCF 在 Cold、Medium 和 Warm 三类用户上的 Recall 均高于 Popular 和 ANN，并且随着用户历史行为数量增加，Recall@200 从 Cold 用户的 0.0318 提升至 Warm 用户的 0.0837，说明基于用户历史行为的协同过滤方法能够从更加丰富的行为信号中获得更充分的个性化信息。相比之下，ANN 在不同用户群体上的 Recall 相对稳定，但 Coverage 分别达到 0.4748、0.3466 和 0.2511，明显高于 ItemCF 和 Popular，表明基于内容语义的向量召回能够覆盖更广泛的候选物品。因此，不同召回方法在行为相关性和候选覆盖方面具有明显的差异。

针对 RQ2，实验结果表明，多路召回融合能够在一定程度上兼顾不同召回通道的优势。Equal-weight RRF 的 Coverage 在 Cold、Medium 和 Warm 用户上分别达到 0.4374、0.3164 和 0.2316，明显高于单独 ItemCF 的 0.2032、0.1247 和 0.0955，说明将 ANN 等高覆盖召回通道引入融合可以有效扩大候选物品空间。但 Equal-weight RRF 的 Recall 低于 ItemCF，表明不同召回通道的贡献并不相同，简单等权融合可能引入较多低相关候选。

进一步采用 Weighted RRF 后，提高 ItemCF 权重能够逐步恢复 Recall。例如在 Warm 用户上，Recall@200 从 Equal-weight RRF 的 0.0428 提升至 1:2:1 的 0.0558，并进一步提升至 1:3:1 的 0.0630；Medium 用户也从 0.0282 提升至 0.0397。与此同时，Coverage 随 ItemCF 权重增加而下降，说明融合过程中存在明显的 Recall-Coverage trade-off。该结果表明，多路召回的价值并非简单地追求单一指标最大化，而是通过不同召回通道之间的互补性，在行为相关性和候选覆盖之间进行平衡。

综合来看，Popular、ItemCF 和 ANN 分别提供热门度、协同行为和内容语义三类互补信息。ItemCF 更适合提供高相关性的行为候选，ANN 则能够扩大候选空间，多路召回可以结合不同信息来源。在当前实验设置下，Weighted RRF 能够通过调整不同召回通道的贡献，在 Recall 和 Coverage 之间实现不同程度的权衡。


