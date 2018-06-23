“You are uniformly charming!” cried he, with a smile of associating and now and then I bowed and they perceived a chaise and four to wish for.       

上述是由简·奥斯丁（Jane Austen）的三元模型随机生成的句子  

能够预测未来并不总是一件好事。特洛伊的卡桑德拉（传说希腊、罗马神话中特洛伊的公主）有预测未来的天赋，但阿波罗诅咒她的预言永远不会被相信。所以她关于特洛伊城毁灭的警告被完全的忽略了。看来能够预测也不一定使未来的事情变好。  

在这一章中，我们将讨论一些不那么令人担忧的预测词汇的话题。例如，什么词很可能出现在下面这句话后面：  
**Please turn your homework ...**  

希望你们大多数人都能得出这样的结论，一个很可能的词出现了，或者可能已经结束了，但是不太可能是**refrigerator**或**the**。在下面的小节中，我们将通过引入模型来规范化这种直观感觉，这些模型为每个可能的下一个单词分配**概率**。同样的模型也可以为整个句子分配一个概率。例如，这种模型可以预测以下序列中，第一句话比第二句话出现在文本中的概率要高得多:  

**1. all of a sudden I notice three guys standing on the sidewalk**  
  
**2. on guys all I of notice sidewalk three a sudden standing the**  
  
你为什么要预测即将出现的单词，或者给句子分配概率?概率在任何任务中都是至关重要的，在这些任务中，我们必须在嘈杂、模糊的输入中识别单词，比如语音识别或手写识别。  
  
在影片《傻瓜入狱记》中，伍迪•艾伦试图用一张写得很潦草的便条抢劫一家银行（便条本意：I have a gun），出纳员错误地把它读作“I have a gub”。正如Russell和Norvig（2002）指出的那样，一个语言处理系统可以避免犯这样的错误，因为它使用了“I have a gun”的顺序，比“I have a gub”或者“I have a gull”这个词更有可能。  
  
在拼写纠正中，我们需要找到并改正拼写错误，就像**Their are two midterms in this class**，这句话把**There**拼写成了**Their**。 显然，以**There are**开头的句子比以**Their are**开头的句子更有可能出现，这样拼写检查器就可以检测并纠正这些错误。
  
  
在机器翻译中，给单词序列分配概率也很重要。假设我们正在翻译一个中文原文:  
  
**他 向 记者 介绍了 主要 内容**  
**He to reporters introduced main content**  
  
作为这个过程的一部分，我们可能已经建立了以下一组可能的粗略的英语翻译:  
he introduced reporters to the main contents of the statement  
he briefed to reporters the main contents of the statement  
**he briefed reporters on the main contents of the statement**  
  
基于词语序列的概率模型会认为**briefed reporters on**要比**briefed to reporter（此时to在briefed 后有些尴尬）**或者**introduced reporters to（单次使用不流畅）**出现的概率更高。所以模型会推荐使用第三种加粗的翻译结果。  
  
概率对于**增强通信（ augmentative communication）**系统也很重要(Newell等人，1998)。像物理学家斯蒂芬·霍金(Stephen Hawking)这样不能用肢体语言或手语的人，可以用简单的动作从选择系统中选择要讲的单词。  
  
为单词序列分配概率的模型称为语言模型（Language Models）或LMs。在这一章中，我们介绍了一个最简单的模型，它将概率LM分配给句子和单词序列N-gram。N-gram是由N个单词组成的序列：2-gram(或bigram)是由两个单词组成的序列，如" please turn "，“turn your”或“your homework”，3-gram(或trigram)是由三个单词组成的，如“please turn your”或“turn your homework”。  
  
我们将看到如何使用N-gram模型来估计在给定前一个词的情况下没计算后一个词的概率，并为整个序列分配概率。在一些术语歧义中，我们通常省略“模型”一词，因此N-gram一词被用来表示单词序列本身或给它分配概率的预测模型。  
  
无论是对下一个单词或整个序列的概率进行估计，N-gram模型是语音和语言处理中最重要的工具之一。  



# 4.1 N-Grams
让我们从计算概率$P(w|h)$开始吧，这个概率是指给定历史h的条件先单词w的概率，假设h表示“its water is so transparent that”，我们想着到它下一个单词为the的概率，那么就可以表示为：    
$$P(the\ |\ its\ water\ is\ so\ transparent\ that)  \tag{4.1}$$   

计算这个概率的一种方法是：从一个非常大的词库中计算“its water is so transparent that”的出现的次数，然后计算“its water is so transparent that”后面紧跟单词“the”的次数，这样就可以回答“当我们看到历史 h 的时候，w出现的次数有多少”，正如下面所表示的：  
$$P(the\ |\ its\ water\ is\ so\ transparent\ that)=\frac{C( its\ water\ is\ so\ transparent\ that\ the )}{C( its\ water\ is\ so\ transparent\ that)} \tag{4.2}$$
有一个足够大的词库，例如网页，我们可以计算这些序列出现的次数，并且通过公式4.2来估算概率，你也可以在这里暂停一下，去爬取网页数据，然后计算这个概率。

然而这个方法是直接从一些已知的实例中通过计数来估算概率，这就会导致在很多方面，为了获得更好的估算结果，web本身也显得不足够大。这是因为语言是创造性的，引得语句随时都在被创造出来，我们也不是总能统计出所有句子。甚至一条简单的语句在web上的出现次数都会为0（例如Walden Pond’s water is so transparent that the）。

类似的，如果我们想知道整个句子中每个单词的联合概率，如“its water is so transparent ”，我们可能会问自己“在这个5个单词出现的所有可能情况中，‘its water is so transparent’出现了多少次？”，我们将不得不将个整个序列拆分为5个独立的单词，然后统计5个单词出现的所有可能情况，这似乎是一个很大的估算任务。

由于这个原因，我们需要引入更聪明的方法来估计一个单词w在给定历史h下的概率，或者整个单词序列W的概率。让我们从符号的形式化开始。为了表示特定随机变量$X_i$取值" the "或$P(X_i = " the ")$的概率，我们将使用简化后的$P(the)$。我们将把N个单词的序列表示为$w_1···w_n$或$w_{1}^{n}$。对于序列中具有特定值的每个单词的联合概率$P(X = w_1,Y = w_2,Z = w3,···,W = w_n)$我们使用$P(w_1,w_2,···;w_n)$表示。

现在我们如何计算整个序列的概率，比如$P(w_1;w_2;···;w_n)$?，我们可以做的一件事是用概率链法则分解这个概率：
$$P(X_1 \dots Xn) = P(X_1)P(X_2|X_1)P(X_3|X_1^2)\dots P(X_n|X_1^{n−1})=\prod_{k=1}^n P(X_k|X_1^{k−1}) \tag{4.3}$$
将链式法则用到单词序列上将得到：
$$P(w_{1}^{n}) = P(w_1)P(w_2|w_1)P(w_3|w_1^2)\dots P(w_n|w_1^{n−1})=\prod_{k=1}^n P(w_k|w_1^{k−1}) \tag{4.3}$$
