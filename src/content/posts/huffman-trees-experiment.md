---
title: Huffman Trees Experiment
pubDate: 2023-05-14
categories: ['Tech']
description: ''
---

> 实现一个基于哈夫曼编码由字符转换成由二进制的字符串
> [项目地址](https://github.com/AnnularLabs/huffman-coding/tree/main)

### 前言

- 在远程通讯中，要将待传字符转换成由二进制的字符串

设要传送的字符以及对应的比编码如下

```text
A —— 00

B —— 01

C —— 10

D —— 11
```

此时如果需要表示`ABACCDA`则对应转化为`00010010101100`

在设计编码时，应该遵循**出现的次数大的字符则采用尽可能短的编码**，以保证整体的二进制字符串长度短

在得出二进制字符串称为**编码**，可以再次通过编码表转化为字符串称为**解码**

### 重码

- 当我们在设计编码时，应该确保任一字符的编码都不是另一个字符的编码的**前缀**否则将会出现**重码**

设要传送的字符以及对应的比编码如下设

```text
A —— 0

B —— 00

C —— 1

D —— 01
```

如果使用上述编码表，将会得出`0000`二进制字符串，将会产生歧义，`0000`通过**解码**可得出以下情况

- AAAA

- ABA

- BB

**因此会造成我们解码的不确定性**

> 关键：在设计长度不等的编码，必须是任一字符的编码都不是另一个字符的编码的前缀

### 哈夫曼编码

- 保证是前缀码

- 保证字符编码总是最短

#### Example

`const word = 'abcd'`

- a：weight = 1
- b：weight = 2
- c：weight = 3
- d：weight = 4

![Diagram](./_assets/huffman-coding/Tweelet.png)

### 实现

总体实现步骤分为`Coding`和`Decoding`

#### Coding

1. 设计每个结点的数据类型(典型的树结构)，

   - str：记录存放的字符
   - weight：该字符的**权重**
   - parent：父节点
   - leftChild：左孩子
   - rightChild：右孩子

   ```js
   class HNode {
     constructor(str = null, weight = 999, parent = -1, leftChild = -1, rightChild = -1) {
       this.str = str
       this.weight = weight,
       this.parent = parent,
       this.leftChild = leftChild,
       this.rightChild = rightChild
     }
   }
   ```

2. 初始化，因为要保证字符编码总是最短，所以在字母出现的频率上就有要求，这里参考了[blog](https://www3.nd.edu/~busiforc/handouts/cryptography/letterfrequencies.html)。有了字母出现的频率就可以进行哈夫曼树的生成。简单复习Huffman的特点

   - 初始时有n颗二叉树，要进过`n-1`次合并最终形成哈夫曼树
   - 经过n-1 次合并产生n-1个新结点，且n-1个新结点都是具有两个孩子的分支结点
   - 哈夫曼树共有`2n-1`个结点，且分支结点度不均不为1

   ```js
   // 初始化
   function initHuffmanTree(letter) {
     // 去除下标0
     const HuffmanTree = [0]

     // 原字母
     letter.forEach((item, index) => {
       HuffmanTree.push(new HNode(item, index + 1).toJSON())
     })

     // 初始化哈夫曼树生成结点
     for (let i = 1; i <= letter.length - 1; i++) {
       HuffmanTree.push(new HNode().toJSON())
     }
   }

   // 生成哈夫曼树
   function createHuffmanTree(HuffmanTree, length) {
     for (let i = length + 1; i < 2 * length; i++) {
       // 挑选权重最小两个
       const [minFirst, minSecond] = SelectMin(HuffmanTree, i - 1)
       /**
        * 1. minFirst的权重 + minSecond的权重
        * 2. 改变minFirst和miSecond的parent值
        * 3. 新HNode设置左孩子和右孩子（遵循左小右大）
        */
       HuffmanTree[i].weight = HuffmanTree[minFirst].weight + HuffmanTree[minSecond].weight
       HuffmanTree[minFirst].parent = i
       HuffmanTree[minSecond].parent = i
       HuffmanTree[i].leftChild = minFirst
       HuffmanTree[i].rightChild = minSecond
     }
   }
   ```

   - 使用`toJSON`的原因让它序列化，这样在后续加入到redux中不会有问题
   - createHuffmanTree的关键是在每次循环中找到两个最小的值。[实现连接](https://github.com/AnnularLabs/huffman-coding/blob/main/src/utils/huffman-coding/selectMin.js)

3. 将获取的word转化为`01`(这里的哈夫曼树遵循左边为0、右边为1)，例如输入的是`hello`，

   - 在letterLETTER_FREQUENCIES中找到`h`的位置，对应的就是在哈夫曼树的下标记为`Child`
   - 获取`Child`的`parent`
   - 只需要看parent的leftChild和rightChild哪边等于Child
   - 循环即可

   ```js
   function createWordCode(HuffmanTree, Letter, words) {
     const resultCodeArray = []
     for (let i = 0; i < words.length; i++) {
       let child = LETTER_FREQUENCIES.indexOf(words[i]) + 1
       let parent = HuffmanTree[child].parent

       const HNodeCode = []
       while (parent !== -1) {
         HuffmanTree[parent].leftChild === child ? HNodeCode.push(0) : HNodeCode.push(1)
         child = parent
         parent = HuffmanTree[child].parent
       }

       // print
       let resultCode = ''
       while (HNodeCode.length > 0) {
         resultCode += HNodeCode.pop()
       }
       resultCodeArray.push(resultCode)
     }

     return resultCodeArray
   }
   ```

### Decoding

1. 获取`code`

2. 从上到下对HuffmanTree进行遍历，0走`leftChild`1走`rightChild`

3. 如果当前为叶子结点则直接取出`HuffmanTree[].str`
