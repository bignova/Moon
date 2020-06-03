---
layout: post
title: "【PR】Generating Sentences from a Continuous Space"
date: 2020-05-28
excerpt: "Paper Reading"
tags: [Paper, Generative Model]
comments: true
---

**Generating Sentences from a Continuous Space**, SR Bowman et al. 

[`Paper`](https://arxiv.org/abs/1511.06349)

相比于传统的RNNLM，VAE学出来的latent space可以保证一定的regularity，并且是“global latent representation of sentence content"。从这点motivation出发，作者采用了一个VAE+LSTM的结构，loss的组成还是vanilla vae的loss。

![](http://rsarxiv.github.io/2017/03/02/PaperWeekly%E7%AC%AC%E4%BA%8C%E5%8D%81%E4%B8%83%E6%9C%9F/media/14884595350366.jpg)

在实际训练的过程中出现了一个很严重的问题：KL-vanishing。LSTM作为decoder本身具有强大的reconstruct能力，并且对于输入的hidden state的变化非常敏感。因此在训练阶段模型会倾向于将encode出的posterior $q(z|x)$完全等于prior $p(z)$，让KL项迅速降为0。同时decoder在忽略hidden state $z$情况下，单独依靠decoder的输入token优化reconstruct error项。模型退化成为一个non-generative RNNLM。

针对这个问题，文章提出了两种办法：

- KL cost annealing - 为loss中的KL项增加一个weight，weight的变化接近sigmoid。以此鼓励模型在初始阶段优化reconstruction的同时，努力在latent space中encode更多的信息，且不受过多的惩罚。在之后开始逐渐增加KL loss项的weight，优化其分布。weight变化的速率设为一个超参。
- Word dropout - 采用弱化decoder能力的思路，通过将decoder lstm input的单词替换为UNK token，迫使decoder依赖$z$的信息。input保留的概率同样设为超参，keep rate。把keep rate=0的情况叫做inputless decoder。

结果方面，inputless的情况下，vae要远优于rnnlm。

在填充句尾空缺词（imputing missing words）的任务中，受到GAN的启发，作者提出了adversarial evaluation，即使用50/50真假的句子训练出一个句子分类器。结果同样是vae更好（分类器在vae生成的句子上accuracy更接近50%，难以分辨）。

最后为了证明vae的确学习出了更为平滑的representation，作者在latent space中做了线性插值的实验，从space中选任意两点$z_1$、$z_2$，并从两点的连线中均匀的采样多个$z$。通过解码发现，多个latent vector构成的句子均符合语法规则，而且句子间语意基本保持平滑过渡。

---

[PaperWeekly第二十七期](http://rsarxiv.github.io/2017/03/02/PaperWeekly第二十七期/) - Paper Review

[Understanding Variational Autoencoders (VAEs)](https://towardsdatascience.com/understanding-variational-autoencoders-vaes-f70510919f73) - vae latent space特征的可视化解释

[Improve Diverse Text Generation by Self Labeling Conditional Variational Auto Encoder](https://arxiv.org/pdf/1903.10842.pdf) - KL-Vanishing Problem
