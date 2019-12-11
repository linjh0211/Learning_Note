{具体内容参考https://zhuanlan.zhihu.com/p/34879333}

这里先列一些概念：

- ICS 
- 白化 ： 主要有两种PCA 和 ZCA ， 注意它们的区别和缺点，BN是对白化的简化和改进
- 梯度饱和区





补充一下背景知识：

- ### 神经网络常用激活函数

  https://blog.csdn.net/edogawachia/article/details/80043673

- ### 非饱和性激活函数

  非饱和性激活函数的意思是说：当 (|limz→−∞f(z)|=+∞)∨|limz→+∞f(z)|=+∞)

  https://www.zhihu.com/question/58648102





总结一下：

BN就两步，

- 第一步是规范化和学习r 和 b![image-20191129113332104](/home/linjinhao/.config/Typora/typora-user-images/image-20191129113332104.png)

- 第二步是 将小个的batch 合并出整体样本空间的特征![image-20191129113540024](/home/linjinhao/.config/Typora/typora-user-images/image-20191129113540024.png)

  