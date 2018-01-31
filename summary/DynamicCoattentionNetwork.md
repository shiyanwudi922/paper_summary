一、摘要：本文提出的DCN包涵两个主要部分

1、对问题和文档的依赖表示（co-dependent representation）进行融合，从而更加关注相关的部分

2、使用动态指向解码器（dynamic pointing decoder）在潜在的答案范围上进行迭代，从而得到正确的答案范围。<u>**该迭代过程使得模型可以从错误答案对应的局部极值上逐步进行修正。**</u>

二、DCN：dynamic coattention network

