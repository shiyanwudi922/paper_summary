一、摘要

1、神经机器翻译需要大规模的平行语料库进行训练，但是，由于人工标注的代价比较高，所以获取这样的语料库比较苦难。对偶学习用于解决这种瓶颈。

2、对偶学习用于机器翻译的原因：每一个机器翻译任务都有一个对偶任务，例如，英语翻译到法语（主任务）的对偶任务是法语翻译到英语。主任务和对偶任务形成了一个闭环，从而在没有人工标注数据的条件下，产生包含对应信息的反馈信号来训练模型。

3、对偶学习的训练方式：使用一个agent来表示主任务的模型，另一个agent表示对偶任务的模型，然后通过强化学习的过程来让这两个agent互相学习。

4、使用单语数据来提升翻译系统性能的方法：

（1）使用目标语言的单语数据来训练语言模型，然后集成到翻译模型中。但是该方法没有从根本上解决平行语料少的问题。

（2）先使用平行语料训练翻译模型，然后使用单语数据通过该翻译模型生成伪平行语料，从而扩充训练数据，用于之后的训练。该方法虽然扩充了平行语料，但是对于伪平行语料的质量没有保证

5、对偶学习的优点

（1）通过强化学习，使用无标注数据训练翻译模型

（2）验证了强化学习应用于复杂现实问题的能力，而不仅仅是游戏



二、对偶学习

![image](https://github.com/shiyanwudi922/paper_summary/blob/master/picture/DuleLearning/algorithm1.png)

1、在训练中，主任务的翻译模型在生成翻译结果时使用beam search；

2、使用强化学习，把对偶任务中的语言模型打分和翻译模型打分分别作为reward，从而通过最大化reward的期望来进行训练；

3、这里有一点暂时没弄明白：reward期望对主任务中的翻译模型参数求导时，有一个单独的r，这个暂时没有推导出来



三、结论

1、对偶学习可以应用的更广泛，只要两个任务具有对偶形式，就可以使用对偶学习通过无标注数据和强化学习同时训练这两个任务的模型；

2、对偶学习可以扩展到更多的任务上，核心的想法是多个任务可以构成一个闭环，从而可以通过对比原始输入数据和最终输出数据来提取反馈信号，从而训练模型。（所以对偶学习更应该被叫做闭环学习）。