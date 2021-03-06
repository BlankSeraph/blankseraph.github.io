---
layout: post
title:  "量化金融-Hurst指数"
categories: 计量 data-science
tags:  data-science 
author: blankseraph
---

* content
{:toc}


## Hurst Exponent
Calculates the Hurst exponent of a time series based on Rescaled range (R/S) analysis.  
Reference: https://en.wikipedia.org/wiki/Hurst_exponent 
### Environment  
Python 3.6.2 AMD64  
numpy (1.13.3+mkl)  
pandas (0.20.3)  
### User Guide  
import Hurst  
ts = list(range(50))  
hurst = Hurst.hurst(ts)
### Tips
The input ts has to be object list(n_samples,) or np.array(n_samples,).





















### 赫斯特指数简介
赫斯特指数（英语：Hurst exponent）以英国水文学家哈罗德·赫斯特命名，起初被用来分析水库与河流之间的进出流量，后来被广泛用于各行各业的分形分析。利用Hurst参数可以表征网络流量的自相似性，Hurst参数越大，说明流量的自相似程度就越高，也就是说网络的业务流量在很长的时间内都具有长相关性，这主要是由于网络流量的突发性造成的。现有的文献给出的估计方法主要是两大类：时域法和频域法，其中时域法包括R/S分析法[1]、时间方差图法[2][3]、IDC法，频域法包括Whittle的最大似然估计[4]、小波法[5]等。常用的Hurst估值算法都有不同的适用条件，不能广泛的应用于各种情况，因为每一种算法在时域或者是频域的范围内应用了求和平均的方法，这样就会使得时间序列的高突发可变的细节信息丢失，从而导致出估算结果为负值，增大了估计误差。

### 简单应用
时域法是直接对时间序列进行处理，并用最小二乘法拟合估计出Hurst参数，频域法通过利用FFT对时间序列的谱密度进行估计。时域法及频域法都要求整个观察时间段内全部的时间序列，当时间范围较大时，就需要大量的序列样本和高采样率，同时很难观察到Hurst参数的时变性。同时，对有限长度的时间序列进行Hurst估算，结果虽然可以反映出网络流量局部的突发性，但是由于估值算法容易受到各种因素的干扰而产生误差，并且由于相邻的估算值之间没有数据关联，就不能够体现出突发的渐进性。因此如何估算出无限增长的流量的突发性，同时又能够体现出网络流量变化的全局渐进性，并且还能够体现出局部变化的时变性，这些都需要做进一步的研究。比如，在IDC基础上定义复数取值的赫斯特指数等等。

### 选取实例
本次分析选取了上交所2009年五支股票78日的日收益率数值，进行赫斯特指数的对比分析
![image.png](https://i.loli.net/2019/10/19/JImzgMXVEZYlvNK.png)
首先利用R语言进行描述性分析瞧一瞧这几只股票都长啥样
>install.packages('PerformanceAnalytics')
>install.packages('DAAG')
>install.packages('readr'); library('readr')
>a<-read_csv('hurst.csv')
>names(a)<-c('y','x2','x3','x4','x5')s
>summary(a)


### 运行后这样的
![image.png](https://i.loli.net/2019/10/19/MYZKuHEG5xy1WIR.png)
哇这几只股票长得好像啊有没有
尤其是前两支股票其收益数据几乎重叠（怨不得呢，回头看看原始数据本来就很像，连涨跌都好似是同步的）
因为1，2股票相近，3，4股票收益率也相近，那么我就选取1，3进行赫斯特指数对比分析了。。。

### 上代码!
>-*- coding: utf-8 -*-
 Reference: https://en.wikipedia.org/wiki/Hurst_exponent
 python 3.6.2 AMD64
 2018/4/19
 Calculate Hurst exponent based on Rescaled range (R/S) analysis
 
>import Hurst 
>import numpy as np
>import pandas as pd
>import  csv

>   ts = list(range(50))
>    hurst = Hurst.hurst(ts)
>     Tip: ts has to be object list(n_samples,) or np.array(n_samples,)
>  __Author__ = "Blank Seraph"

>      def hurst(ts):
>      N = len(ts)
>      print(N)
>       if N < 20:
>      raise ValueError("Time series is too short! input series ought to have at least 20 samples!")
>    max_k = int(np.floor(N/2))
>    R_S_dict = []
>     for k in range(10,max_k+1):
>        R,S = 0,0
>    # split ts into subsets
>    subset_list = [ts[i:i+k] for i in range(0,N,k)]

>     if np.mod(N,k)>0:
>        subset_list.pop()
>   #tail = subset_list.pop()
>  #subset_list[-1].extend(tail)
>        # calc mean of every subset
>     mean_list=[np.mean(x) for x in subset_list]

>      for i in range(len(subset_list)):
>         cumsum_list = pd.Series(subset_list[i]-mean_list[i]).cumsum()
>        R += max(cumsum_list)-min(cumsum_list)
>       S += np.std(subset_list[i])
>R_S_dict.append({"R":R/len(subset_list),"S":S/len(subset_list),"n":k})
> log_R_S = []
> log_n = []
> print(R_S_dict)

>     for i in range(len(R_S_dict)):
>        R_S = (R_S_dict[i]["R"]+np.spacing(1)) / (R_S_dict[i]["S"]+np.spacing(1))
>     log_R_S.append(np.log(R_S))

>     log_n.append(np.log(R_S_dict[i]["n"]))
>      Hurst_exponent = np.polyfit(log_n,log_R_S,1)[0]
>     print(Hurst_exponent)
>     return Hurst_exponent

>      if __name__ == '__main__':
>         ts = list()
> with open('C:/Users/13760/Desktop/hurst.csv', mode='r', encoding='utf-8') as infile:
> read = csv.reader(infile)
> for line in read:
> ts.append(line[1])
> print(ts)

>      N = len(ts)
>      ts = np.array(ts)
>      ts = ts.astype(np.float)
>           hurst(ts)

然后我们来看一下这几支股票的hurst指数究竟咋样？（分别修改ts.append(line[1]和line[2])运行
可得第一支股票的hurst指数为0.4386795017068711；第二支股票为0.5293240661196189
；第三支股票为0.6555969221053866；第四支为0.5492369921528519；第五支为0.37444714098902215。
hurst指数的取值范围在 0 和 1 之间（不包括 0 和 1）。当hurst指数=0.5时，该时间序列没有相关性。当hurst指数在0.5~1时，该时间序列有长记忆性；当hurst指数在0~0.5时，该时间序列表现出反持续性，因此它表现出比纯随机更强的波动，由此可以看出第一支和第五支股票的波动性较强，第二支第四支股票波动接近随机，而第三支股票则相对稳健呈现持续性周期性的波动，由于三四支股票收益率整体上相近，所以选取第三支股票比较稳妥。


## 沪深300期现与跨期套利分析

### 套利与跨期套利

套利交易一般分为跨期套利、跨品种套利和跨市场套利三种基本模式。在国内，较常见的是“跨期套利”和现货市场与期货市场之间的“期现套利”。跨期 套利占用资金不多，交易灵活，风险较小而收益可观，比较适合资金量不大的一 般中小投资者。而期现套利，因为需要交割，占用的资金量较大，而且还要有良 好的现货贸易渠道来收购或消化现货，一般只有投资机构才能操作；而且，随着 期货交易越来越成熟，可操作的期现套利机会越来越少，当然，期现套利机会一 旦出现，则风险极小，而收益稳定。



### 跨期套利的基本原理与交易方式

在期货交易中，由于同一商品的不同合约的价格是受到同样的因素影响，因 此一般都是同涨同跌的，而且最终合约间的价差的结果往往有一定的规律性，而 决定合约间价差关系的因素主要有持仓费用，季节性因素、现货供求状况变化和 人为因素等。但在合约存活的整个交易过程中，由于资金对各合约的交易的不均 衡，各合约的涨跌的幅度往往是不一致的。正是这种涨跌幅度的差异给了套利交 易者获利的机会。
跨期套利是利用不同到期月份期货合约的价差变化，买入一个期货合约的同 时，卖出另一个期货合约，等价差扩大或缩小到一定幅度将两个合约一起平仓来 获利的操作模式。跨期套利是现实条件下最为成熟和风险较小的套利交易模式， 理论上分为正向套利与反向套利两种模式：正向套利：买入近期合约、卖出远期 合约；反向套利：卖出近期合约、买入远期合约。由于反套不能转化为实盘套利， 当现货紧张时近月对远月升水可以无限增加，所以反套的风险是事先不可预知 的，在交易中必须要设好止损，执行严格的资金管理。
无论是正向套利还是反向套利，要点都是要知道什么情形下价差属于正常范 围、什么情形下价差属于不正常范围？只有价差处于不正常区域内，我们预期存 在价差回归正常的过程，才能存在跨期价差套利机会。 目前一般采取下面二种方式来确定价差的合理区域。
 

1、按照交割式套利（仓单回购）方式，计算各价差之间的套利成本，然后
对相应的价差进行追踪，如果价差超过此套利成本，则存在无风险（交割式）套 利机会。


2、根据历史数据统计结果导出一般情形下某对月份的合约间的价差变化规 律，然后对比本年度同月份的合约的价差变化情况，如果偏离过大，则可能存在 价差回归过程，从而产生价差套利机会。由于资金面的松紧程度、商品的紧缺或 过剩、国家的行业政策等因素很可能发生变化，使得目前的同种商品的价格环境 与历史环境产生差异，导致价差可能不再向历史规律回归，因此按这个方式确定 的套利机会还是存在一定的风险。


### 根据历史数据统计结果操作方法沪深 300 套利机会分析

沪深 300指数	10 月部分数据

![沪深 300指数	10 月部分数据](https://i.loli.net/2019/11/13/Gz8WK4wOdnNXptJ.png)
 
 
沪深 300 日价差

星期	收盘平均价	最高价差	最低价差	日价差


周一	3917.0405	-99.0670	-78.6951	-20.3719


周二	3916.0654	-55.7293	-57.6024	1.8731


周三	3896.8851	-62.6213	-56.8204	-5.8009


周四	3897.9447	-45.6168	-61.3345	15.7177


周五	3883.0850	-40.8186	-15.9315	-24.8871




 
从上述日期价差可以看出，虽然沪深 300 指数 10 月份日价差波动范围不是大，从 6
0 到 1190 点；但最后交易日价差范围却处于-24 到 15 之间，最终价差都是逐步走低到 -5 点左右。55％的绝对价差都是在 40-65 点之间。从此分析中还不能得出明显的套利行为。
 
 
### 跨期套利操作的风险及控制

跨期套利属于风险较低、收益稳定的投资。但是低风险不等于无风险。套利交易可能存在反向套利价差的不回归风险、正向套利的交割风险、流动性风险等。
1、反向套利价差不回归风险：这是最常见也最容易导致套利交易亏损的风险。在反向套利中，价差不回归风险尤其大，尤其是在商品价格熊市中，市场对商品价格预期可能越来越悲观，从而导致远月价格进一步低估，导致价差不能回归。


2、正向套利价差不回归——导致交割风险：在正向套利中，如果价差继续 扩大，投资者需要通过交割来完成套利操作时，就存在交割风险。虽然最初操作正向套利时，价差已经基本覆盖了交割操作所需的费用，但是，其中有一项事先基本上是不能完全确定的项目——增值税：在近月完成交割后，如果远月继续大幅上涨，则增值税的支出将持续增加，很快就会把本来并不太多的预期利润蚕食掉，甚至还可能会出现亏损。


3、做套利操作计划时还需要对所操作的合约的流动性进行分析，如果成交量太小，将会造成大的价格冲击，使得成交后的价差不是预先计划的数量，从而导致套利交易失败。
在这些风险中，交割风险和流动性风险较容易估计，而且一般造成的损失也有限。反向套利的风险则难以预先估计，而且一旦反向套利的价差不回归，往往可能会给投资者带来很大的损失，因此在反向套利交易中，投资者一定要注意价格冲击对于合约的影响。
