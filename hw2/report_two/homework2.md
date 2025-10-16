# Signal Detection Theory(SDT)
> 独宇涵 231880151

信号检测论试图解决当刺激信号与背景噪音有重叠时，如何区分观察者的敏感性（sensitivity）和反应偏差（bias）
## SDT简介
在SDT理论框架下，存在噪声信号$Noise$与刺激信号$Signal$，从代码中可看出，本次实验将二者都建模成了服从正太分布的形式来表示
$$\begin{align}
Noise &= N(0,1)\\
Signal &= N(d,1)
\end{align}$$

除此之外，还设置一个观察者的判断标准 Criterion
$$
Signal + Noise \geq Cirterion \to report\ 'signal'\ (response)
$$

进行上述实验我们有四种可能的结果，可以用如下所示的表格来表示
| 实际情况 | 观察者反应 | 结果类型                         |
| ---- | ----- | ---------------------------- |
| 有信号  | 报告有信号 | **命中 (Hit)**                 |
| 有信号  | 报告无信号 | **漏报 (Miss)**                |
| 无信号  | 报告有信号 | **虚警 (False Alarm)**         |
| 无信号  | 报告无信号 | **正确拒绝 (Correct Rejection)** |

通过统计上述结果，可以计算得到
$$
\begin{aligned}
命中率（Hit Rate） &= P(S + N \geq c | Signal Present) \\
虚警率（False Alarm Rate） &= P(S + N \geq c | Signal not Present)
\end{aligned}
$$

## Part one
### 代码补全
有了上述知识，很容易可以补全片段一的空白部分代码的内容
```julia
# 观察者是否做出反应 
responses = stimuli .>= criterion
# 四种结果的统计
hits = sum(responses .& (signalPresent .== 1))
misses = sum((.!responses) .& (signalPresent .== 1))
falseAlarms = sum(responses .& (signalPresent .== 0))
correctRejections = sum((.!responses) .& (signalPresent .== 0))
```

运行结果如下所示
![](SDT1.jpg)
```
d': 1.5
Criterion: 0.8
Hits: 349 (73.32%)
Misses: 127 (26.68%)
False Alarms: 99 (18.89%)
Correct Rejections: 425 (81.11%)
```

### Discussion
在阅读了一些参考文章后，发现信号检测模型中有两个重要指标
- 感受性（sensitivity）
- 反应偏向（response bias）

#### 感受性（sensitivity）
$$
d' = \phi^{-1}(Hit Rate) - \phi^{-1}(False Alarm Rate)
$$
在标准正态分布的前提下，上述计算实际上是两个正态分布的均值之差，在之前的结果图片中不难看出，当 d' 增大时，两个正态分布之间的距离增大，噪声和信号更容易区分，当 d' 减小的时候，正态分布之间的距离减小，噪声和信号区分困难，虚警率和命中率会下降，当 d' 减小为0时，噪声和信号完全重合，无法区分（此时相当于随机猜测）

用图像可以形象展示出不同 d' 对最终结果的影响

![alt text](Partonedis.png)

#### 反应偏向（response bias）
$$
c = -\frac{\phi^{-1}(H) + \phi^{-1}(F)}{2}
$$
在噪声和信号分布固定不变时，改变反应偏向的值的大小会对命中率和虚警率产生影响，当c接近 $\phi^{-1}(hit rate)$ 时，此时虚警率降低（更少的Noise会被误判为信号），但是命中率也会降低（更多的信号被误判为噪声）。 当c更接近$\phi^{-1}(False Alarm Rate)$时，此时虚警率升高，但是命中率也在升高

在d' = 1.5的前提下，不同反应偏向对实验结果的影响如下表所示
| | c = 0 | c = 0.5 | c = 1 | c = 1.5 |
| - | ----- | ------- | ----- | ------- |
|Hits| 473 (94.04%) |439 (86.59%) | 341 (70.02%) | 244 (48.51%) |
|Misses| 30 (5.96%) | 68 (13.41%) | 146 (29.98%) | 259 (51.49%) |
|False Alarms| 211 (42.45%) | 154 (31.24%) | 84 (16.37%) | 46 (9.26%) |
|Correct Rejections| 286 (57.55%) |339 (68.76%)| 429 (83.63%) | 451 (90.74%) |

## Part Two
ROC曲线横坐标是False Alarm Rate、纵坐标是Hit Rate，绘制时是在固定分布d' 下，改变反应偏向 c 得到一系列 （FAR，HR）并将其连接绘制成曲线

曲线越偏向左上角远离对角线则表示检测性能越好，可以使用AUC（Area Under Curve）来衡量，AUC = 0.5表示 d' = 0 (纯随机过程)，AUC越大表示信号检测的性能越高

在上一部分对反应偏向的讨论中可以看出，随着C的增大，命中率和虚警率都在减小，而随着c的减小，虚警率和命中率都在增大，而在不同的d'的设置下，FAR和HR关于 c 的增长曲线不同，使得在d'增大时，ROC曲线越来越接近右上角，AUC也随之增大 (这和Part one 中对感受性部分的讨论结论相同——感受性越大，信号和噪声的区分越容易)

片段二需要补全的部分和代码一实质相同，补全部分的代码如下所示
```julia
    for (i, criterion) in enumerate(criteria)
        responses = stimuli .>= criterion
        hits = sum(responses .& (signalPresent .== 1))
        falseAlarms = sum(responses .& (signalPresent .== 0))

        hitRates[i] = hits / sum(signalPresent)
        falseAlarmRates[i] = falseAlarms / sum(.!signalPresent)
```

最终结果

![](ROC.png)