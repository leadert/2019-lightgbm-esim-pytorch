# 2019-lightgbm-esim-pytorch
2019中国高校计算机大赛--大数据挑战赛Top 12（福建大三本）-Solutions

比赛链接：[2019中国高校计算机大赛--大数据挑战赛](https://www.kesci.com/home/competition/5cc51043f71088002c5b8840  "悬停显示文字")
## 正式赛题--文本点击率预估（5月26日开赛）
搜索中一个重要的任务是根据query和title预测query下doc点击率，本次大赛参赛队伍需要根据脱敏后的数据预测指定doc的点击率，结果按照指定的评价指标使用在线评测数据进行评测和排名，得分最优者获胜。

### 比赛数据
#### sample 样本格式
| 列名      | 类型     | 示例     |
| ---------- | :-----------:  | :-----------: |
| query_id     | int，一个query的唯一标识     | 1     |
| query     | 字符string，term空格分割     | "字节跳动"     |
| title     | 字符string，term空格分割     | "字节跳动 - 百科"     |
| label     | int，取值{0, 1}，有点击为1，无点击为0   | 1     |

#### training 样本
脱敏后的query和网页文本数据，并已经分词为term并脱敏，term之间空格分割。

| 列名      | 类型     | 示例     |
| ---------- | :-----------:  | :-----------: |
| query_id     | int     | 3     |
| query     | hash string，term空格分割    | 1 9 117     |
| query_title_id    | title在query下的唯一标识    | 2    |
| title     | hash string，term空格分割    | 3 9 120     |
| label     | int，取值{0, 1}   | 0     |

### 评估标准
qAUC, qAUC为不同query下的AUC的平均值，计算如下：

<a href="https://www.codecogs.com/eqnedit.php?latex=qAUC&space;=&space;\frac{\sum&space;AUC_{i}}{query\_num}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?qAUC&space;=&space;\frac{\sum&space;AUC_{i}}{query\_num}" title="qAUC = \frac{\sum AUC_{i}}{query\_num}" /></a>

其中AUCi为同一个query_id下的AUC（Area Under Curve）。

### 最终成绩
初赛Rank 34(lgb)，复赛A榜Rank 17，B榜Rank 12，score=0.63834。

（第一次参加NLP比赛，能拿到这个名次已经很开心了。希望以后能再接再厉，更进一步）



## 项目说明
训练lgb所采用的数据为10亿训练集中的后5kw，训练ESIM所采用的数据为10亿训练集中随机抽样的8kw。

#### lgb_word2vec.ipynb
训练用于lgb的word2vec词向量，5kw训练集与1亿最终测试集作为数据集，min_count=1, size=150，未对句子进行去重。

#### nn_word2vec.ipynb
训练用于ESIM的word2vec词向量，8kw训练集与1亿最终测试集作为数据集，min_count=4, size=150，未对句子进行去重。

#### nn_fasttext.ipynb
训练用于ESIM的fasttext词向量，8kw训练集与1亿最终测试集作为数据集，min_count=4, size=150，未对句子进行去重。

#### lgb_model.ipynb
训练lgb，A榜0.595266采用了40维向量，线下loss为0.435457左右（后两位不记得了）。包括统计特征，频率特征，nlp特征，jaccard、levenshtein、fuzzywuzzy距离度量特征等，特征都比较普通。最终成绩中采用了约50维特征，线下的loss为0.435312。lightgbm中num_leaves=127。

#### esim_pytorch.ipynb
训练ESIM，纯nn模型，8kw训练集在A榜上能够取得0.588336的成绩，在最终测试集的效果应该更好，因为A榜训练集并没有作为我们训练词向量的数据集来源。Embedding层使用word2vec+fasttext拼接而成的词向量，即每个词对应300维，在训练过程中进行微调。

#### 模型融合
简单地对lgb和esim的结果加权融合，0.595266 lgb与0.586439 esim平均融合能取得0.612357的成绩，0.55*lgb + 0.45*esim 能取得0.612852的成绩。0.595266 lgb与0.587132 esim 平均融合能取得0.613485的成绩，0.55*lgb + 0.45*esim 能取得0.613587的成绩。
