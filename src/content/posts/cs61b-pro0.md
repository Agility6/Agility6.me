---
title: cs61b-sp21-project0-2048
pubDate: 2023-10-10
categories: ['Tech']
description: ''
---

project0实现一个2048游戏，大部分逻辑其实已经写好了，只需要把目光放在`Model.java`中主要的任务实现以下几个函数

- `emptySpaceExists`
- `maxTileExists`
- `atLeastOneMoveExists`
- `tilt`

> 前三个较为简单这里就不多赘述，重点实现tile方法

## Tile method实现

在写tile函数之前务必阅读文档，了解一下函数的用法以及作用

- Border的tile
- move
- setViewingPerspective

完成以下检验是否真正的理解需求

- [Google Form quiz](https://forms.gle/pubhRx4fxYnPTGNX8)
- [Google Form quiz](https://forms.gle/AGrhEFbwfMJ7qwaB6)

### 实现

根据Tips的提示，我们可以先从只考虑向上移动来进行分析。Border.tile(c, r)的行为是一列一列进行遍历的。

```java
  for (int c = 0; c < border.size(); c++) {
    for (int r = 0; r < border.size(); r++) {
      // 具体实现逻辑，针对该列进行操作
    }
  }
```

判断是否合并其实就是寻找该列中是否满足某个性质，如果满足则移动到`X`行。需要维护每一列的数据。因为文档提供了`setViewingPerspective`，因此只需要专注实现向上移动的逻辑，如果我们的循环是从顶行开始，会方便许多，因为向上移动是最简单的情况，如果某一列上的顶行没有元素，则直接移动到顶行即可。

```java
  for (int c = board.size() - 1; c >= 0; c--) {

    int[] x = new int[4];

    for (int r = board.size() - 2; r >= 0; r--) {

      // 具体实现逻辑，针对该列进行操作
    }
  }
```

维护了一个数组`x`用于记录当前行中有什么元素，用于判断是否合并，所以大体的框架就可以写出来了

```java
  for (int c = board.size() - 1; c >= 0; c--) {

    int[] x = new int[4];

    // 将顶行元素添加到辅助数组中
    if (board.tile(c, 3) != null) x[3] = board.tile(c, 3).value();

    for (int r = board.size() - 2; r >= 0; r--) {

      if (board.tile(c, r) != null) {

        int currentValue = board.tile(c, r).value();

        Tile t = board.tile(c, r);

        x[r] = currentValue;

        // 获取更新到某行
        // int moveStep = TODO

        // TODO Score实现

        board.move(c, moveStep, t);

        // TODO 更新辅助数组值
        changed = true;
      }
    }
  }
```

现在只需要考虑如何获取该移动到多少行，以及怎么更新移动后到数组

- 合并

  **当前元素到顶行中，遇到的第一个非0的数，与其相等**

- 不合并

  **当前元素到顶行中，遇到的第一个非0的数，与其不相等**

#### 获取Step函数

```java
    private int getMoveStep(int[] x, int col, int row, int currentValue, int len) {

        // 记录移动到多少行
        int res = 0;

        for (res = row; res < x.length - 1 - len; res++) {
            // 当前位置上的元素不等于上面的元素且上一个元素不等于零则不动
            if (currentValue != x[res + 1] && x[res + 1] != 0) return res;
            // 当前元素等于上一个元素 当起行+1
            else if (currentValue == x[res + 1]) return res + 1;
        }
        return res;
    }

```

我们还需要维护一个`len`，寻找第一个非0的数，边界应该是顶行元素，注意这里的顶行元素不应该是被合并过的值（如下图

- 当我们遍历到蓝色的4，这时候我们就不能以最初的顶行为标准了

- 那len是从哪里来的，len的大小应该与合并的次数有关，只有当该列进行了合并，才会修改边界

![为什么需要len](./_assets/cs61b-pro0/为什么需要len.png)

#### update函数

在执行完move函数，循环遍历该列更新辅助数组

```java
  private void update(int[] x, int col) {

    for (int r = 0; r < board.size(); r++) {
      if (board.tile(col, r) != null) {
        x[r] = board.tile(col, r).value();
      } else {
        x[r] = 0;
      }
    }
  }

```

#### 计算Score函数

- 借助`moveStep`的值，只需要判断，移动的位置元素的值，是否与它本身元素值相等

- 并且返回的`moveStep`不能与它本身的位置相等。

```java
  if (board.tile(c, moveStep) != null && moveStep != r) {
    score += 2 * board.tile(c, moveStep).value();
    len++;
  }
```

### 完成

[可参考](https://github.com/Agility6/cs61b-sp21/blob/main/proj0/game2048/Model.java#L109)

![done](./_assets/cs61b-pro0/完成.png)
