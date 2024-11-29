---
title: 公众号如何实现用户抽奖功能？
urlname: girytbel2gdup8q4
date: '2024-06-27 21:23:49'
updated: '2024-06-27 21:24:05'
cover: 'https://cdn.nlark.com/yuque/0/2024/png/22382235/1709885517843-80a6a2ca-81ee-40b7-acab-6a92559b9a79.png'
description: 实现背景朋友搞一个了公众号，找我帮忙开发一个功能。他最近为了提高公众号的用户粘性，需要增加一个功能，那就是用户可以点击公众【每日礼包】的按钮，实现抽奖，获得各种奖励等等。实现起来其实挺简单的，时序图如下：实现由于之前已经配置好了后台服务器的事件监听，其实这次主要做的工作就是实现抽奖功能。简单分...
---
# 实现背景
朋友搞一个了公众号，找我帮忙开发一个功能。
他最近为了提高公众号的用户粘性，需要增加一个功能，那就是用户可以点击公众【每日礼包】的按钮，实现抽奖，获得各种奖励等等。
实现起来其实挺简单的，时序图如下：
![image.png](https://oss1.aistar.cool/elog-offer-now/e6039d4b6e10c0aa877232d5666a3c2c.png)
![](https://oss1.aistar.cool/elog-offer-now/e3738c1d697277ae1954b57286a2c835.svg)# 实现
由于之前已经配置好了后台服务器的事件监听，其实这次主要做的工作就是实现抽奖功能。
简单分析一下，创建一个简单的抽奖程序需要考虑几个关键点：**奖品的配置、概率的计算以及抽奖算法**。
下面是一个简单的示例：
```go
package web_activity

import (
	"fmt"
	"math/rand"
)

type PrizeType int64

const (
	LotteryTypeDay    PrizeType = 1001 // 日卡
	LotteryTypeWeek   PrizeType = 1002 // 周卡
	LotteryTypeMonth  PrizeType = 1003 // 月卡
	LotteryTypeSeason PrizeType = 1004 // 季卡
	LotteryTypeYear   PrizeType = 1005 // 年卡

	LotteryTypeNum50  PrizeType = 50  // 增加50次聊天
	LotteryTypeNum20  PrizeType = 20  // 增加20次聊天
	LotteryTypeNum100 PrizeType = 100 // 增加100次聊天
	// ...
)

// Prize 定义了奖品及其概率
type Prize struct {
	PrizeType   PrizeType // 奖品类型
	Probability float64   // 中奖概率
}

// Lottery 抽奖程序
type Lottery struct {
	Prizes []Prize // 奖品列表
}

// NewLottery 创建一个新的Lottery实例
func NewLottery(prizes []Prize) *Lottery {
	return &Lottery{Prizes: prizes}
}

// Draw 执行抽奖过程
func (l *Lottery) Draw() *Prize {
	var sum float64
	for _, prize := range l.Prizes {
		sum += prize.Probability
	}

	r := rand.Float64()
	var s float64
	for _, prize := range l.Prizes {
		s += prize.Probability
		if r <= s {
			return &prize
		}
	}

	return nil
}

func main() {
	prizes := []Prize{
		{LotteryTypeNum20, 0.2},
		{LotteryTypeNum50, 0.1},
		{LotteryTypeNum100, 0.05},
		{LotteryTypeDay, 0.05},
		{LotteryTypeWeek, 0.001},
		{LotteryTypeMonth, 0.0005},
		{LotteryTypeSeason, 0.00001},
		{LotteryTypeYear, 0.000001},
	}

	lottery := NewLottery(prizes)

	for i := 0; i < 10; i++ {
		fmt.Println("抽奖结果：", lottery.Draw())
	}
}

```
在这个程序中，我定义了`Prize`结构体来表示奖品和它们对应的概率。`Lottery`结构体包含了一个奖品列表，并提供了一个`Draw`方法来执行抽奖过程。

在`Draw`方法中生成一个[0,1)范围内的随机数，根据奖品的概率区间来确定中奖的奖品。

这段代码其实实现的是一个权重随机选择器，其原理基于累积分布函数（Cumulative Distribution Function, CDF）：

1.  `r := l.random.Float64()`：生成了一个介于0到1之间的随机浮点数。这个数可以被看作是在概率分布线上的一个点。  
2.  `for _, prize := range l.Prizes`：这个循环遍历所有的奖品。 
3.  `s += prize.Probability`：在循环中，每个奖品的概率都被累加到`s`变量中。这个累加过程构建了一个概率的累积分布，其中`s`的值在每次迭代后都会增加，直到最后一个奖品的概率被加上。 
4.  `if r <= s`：这个条件检查随机数`r`是否小于或等于当前的累积概率`s`。由于概率是累积的，每个奖品都对应了概率线上的一个区段。如果`r`落在某个奖品对应的区段内，那么这个奖品就是被选中的奖品。 

这个方法确保了每个奖品被选中的概率与其设定的概率相匹配。举一个简单的例子：

假设有三个奖品，概率分别为0.2、0.3和0.5。概率线如下：
```
|-----0.2-----|-------0.3-------|------------0.5------------|
```

- 如果`r`介于0到0.2之间，第一个奖品被选中。
- 如果`r`介于0.2到0.5之间（0.2 + 0.3），第二个奖品被选中。
- 如果`r`介于0.5到1之间（0.2 + 0.3 + 0.5），第三个奖品被选中。

这种方法简单高效。
# 效果图
公众号菜单：
![image.png](https://oss1.aistar.cool/elog-offer-now/e5e71995d7391c6795ac73fb59009eb4.png)
测试抽奖100次，中奖次数大约占据40，综合中奖率足够了，再说了概率这东西，随时都可以调整。
![image.png](https://oss1.aistar.cool/elog-offer-now/1696ccd4726ae01ad13a267eb3bc52cf.png)
以上就是所有的过程了~希望各位看官赏脸点个赞哈😆
