## 3. 性能说明

ta模块中的函数与TA-Lib中对应函数相比，直接使用时的平均速度相似，但在分组计算时，ta模块中的函数性能远超TA-Lib中对应函数。本节的性能对比，我们以`wma`函数为例。

### 3.1 直接使用性能对比

在DolphinDB中：
```
use ta
close = 7.2 6.97 7.08 6.74 6.49 5.9 6.26 5.9 5.35 5.63
close = take(close, 1000000)
timer x = wma(close, 5);
```
对一个长度为1,000,000的向量直接使用ta模块中的`wma`函数，耗时为3毫秒。

与之对应的Python语句如下：
```python
close = np.array([7.2,6.97,7.08,6.74,6.49,5.9,6.26,5.9,5.35,5.63,5.01,5.01,4.5,4.47,4.33])
close = np.tile(close,100000)

import time
start_time = time.time()
x = talib.WMA(close, 5)
print("--- %s seconds ---" % (time.time() - start_time))
```
TA-Lib中`WMA`函数耗时为11毫秒，为DolphinDB ta module中`wma`函数的3.7倍。

### 3.2 分组使用性能对比

在DolphinDB中，构造一个包含1000只股票，总长度为1,000,000的数据表：
```
n=1000000
close = rand(1.0, n)
date = take(2017.01.01 + 1..1000, n)
symbol = take(1..1000, n).sort!()
t = table(symbol, date, close)
timer update t set wma = wma(close, 5) context by symbol;
```
使用ta模块中的`wma`函数对每只股票进行计算，耗时为17毫秒。

与之对应的Python语句如下：
```python
close = np.random.uniform(size=1000000)
symbol = np.sort(np.tile(np.arange(1,1001),1000))
date = np.tile(pd.date_range('2017-01-02', '2019-09-28'),1000)
df = pd.DataFrame(data={'symbol': symbol, 'date': date, 'close': close})

import time
start_time = time.time()
df["wma"] = df.groupby("symbol").apply(lambda df: talib.WMA(df.close, 5)).to_numpy()
print("--- %s seconds ---" % (time.time() - start_time))
```
使用TA-Lib中`WMA`函数对每只股票进行计算耗时为535毫秒，为ta模块中`wma`函数的31.5倍。


## 4. 向量化实现

ta模块中的所有函数与TA-Lib一样，都是向量函数：输入为向量，输出的结果也是等长的向量。TA-Lib底层是用C语言实现的，效率非常高。ta模块虽然是用DolphinDB的脚本语言实现，但充分的利用了内置的向量化函数和高阶函数, 避免了循环，极为高效。已经实现的57个函数中，28个函数比TA-Lib运行的更快，最快的函数是TA-Lib性能的3倍左右；29个函数比TA-Lib慢，最慢的性能不低于TA-Lib的1/3。

ta模块中的函数实现亦极为简洁。ta.dos总共765行，平均每个函数约14行。扣除注释、空行、函数定义的起始结束行，以及为去除输入参数开始的空值的流水线代码，每个函数的核心代码约4行。用户可以通过浏览ta模块的函数代码，学习如何使用DolphinDB脚本进行高效的向量化编程。

### 4.1. 空值的处理

若TA-Lib的输入向量开始包含空值，则从第一个非空位置开始计算。ta模块采用了相同的策略。

对一个滚动/累积窗口长度为k的函数，每组最初的(k-1)个位置的结果均为空。这一点TA-Lib与ta模块的结果一致。但若一组中第一个非空值之后再有空值，该组此空值位置以及所有以后位置在TA-Lib函数中的结果有可能均为空值。对ta模块函数，除非窗口中非空值数据的数量不足以计算指标（例如计算方差时只有一个非空值），否则在这些位置均会产生非空的结果。

DolphinDB代码与结果：
```
close = [99.9, NULL, 84.69, 31.38, 60.9, 83.3, 97.26, 98.67]
ta::var(close, 5, 1);

[,,,,670.417819,467.420569,539.753584,644.748976]
```
Python代码与结果：
```python
close = np.array([99.9, np.nan, 84.69, 31.38, 60.9, 83.3, 97.26, 98.67])
talib.VAR(close, 5, 1)

array([nan, nan, nan, nan, nan, nan, nan, nan])
```
上面的总体方差计算中，因为close的第二个值为空值，ta模块和TA-Lib的输出不同，TA-Lib输出全部为空值。如果替换空值为81.11，ta模块和TA-Lib得到相同的结果。在第一个元素99.9之前加一个空值，两者的结果仍然相同。简而言之，当输入参数中空值，若存在的话，只集中在开始的位置时，ta模块和TA-Lib的输出结果才会完全一致。

### 4.2 迭代处理
技术分析的很多指标计算会用到迭代，即当前值为前一个值和当前输入的线性函数： r[n] = coeff * r[n-1] + input[n]。对于这一类型的计算，DolphinDB引入了函数`iterate`进行向量化处理，以避免使用循环。
```
def ema(close, timePeriod) {
1 	n = close.size()
2	b = ifirstNot(close)
3	start = b + timePeriod
4	if(b < 0 || start > n) return array(DOUBLE, n, n, NULL)
5	init = close.subarray(:start).avg()
6	coeff = 1 - 2.0/(timePeriod+1)
7	ret = iterate(init, coeff, close.subarray(start:)*(1 - coeff))
8	return array(DOUBLE, start - 1, n, NULL).append!(init).append!(ret)
}
```
以`ema`函数实现为例，第5行代码计算第一个窗口的均值作为迭代序列的初始值。第6行代码定义了迭代参数。第7行代码使用`iterate`函数计算ema序列。内置函数`iterate`有非常高的运行效率，计算长度为1,000,000的向量的ema序列，窗口长度为10时，TA-Lib耗时7.4ms，ta模块仅耗时5.0ms，比TA-Lib更快。

### 4.3 滑动窗口函数的应用

大部分技术指标会指定一个滑动窗口，在每一个窗口中计算指标值。DolphinDB的内置函数中已经包括了一部分基本的滑动窗口指标的计算，包括`mcount`, `mavg`, `msum`, `mmax`, `mmin`, `mimax`, `mimin`, `mmed`, `mpercentile`, `mrank`, `mmad`, `mbeta`, `mcorr`, `mcovar`, `mstd`和`mvar`。这些滑动窗口函数经过了充分的优化，大部分函数的复杂度达到了O(n)，即耗时与窗口长度无关。

某些TA-Lib函数可以通过叠加或变换上述滑动窗口函数来实现。例如，TA-Lib中的VAR函数是总体方差，而DolphinDB内置的`mvar`函数是样本方差。ta模块中的`var`函数可由以下代码实现：
```
def var(close, timePeriod, nddev){
1	n = close.size()
2	b = close.ifirstNot()
3	if(b < 0 || b + timePeriod > n) return array(DOUBLE, n, n, NULL)
4	mobs =  mcount(close, timePeriod)
5	return (mvar(close, timePeriod) * (mobs - 1) \ mobs).fill!(timePeriod - 1 + 0:b, NULL)
}
```
下面的例子更为复杂，为ta模块中`linearreg_slope`函数的实现。`linearreg_slope`计算给定向量相对于序列 0..(timePeriod - 1)的beta。这个指标看上去无法实现向量化，必须取出每个窗口的数据，循环计算beta。但由于序列0..(timePeriod - 1)是一个固定的等差序列，所以仍然可以实现向量化计算。由于beta(A,B) = (sum(A\*B) - sum(A)\*sum(B)/n)/n/var(B)，而且var(B)和sum(B)是固定的（因为B是固定的向量），我们只需要优化滑动窗口时sum(A\*B)和sum(A)的计算。代码第10行通过向量化实现了sum(A\*B)在两个相邻窗口之间的变化。第12行计算第一个窗口的sum(A\*B)。第13行中的sumABDelta.cumsum()向量化计算所有窗口的sum(A\*B)值。
```
def linearreg_slope(close, timePeriod){
1	n = close.size()
2	b = close.ifirstNot()
3	start = b + timePeriod
4	if(b < 0 || start > n) return array(DOUBLE, n, n, NULL)
5	x = 0 .. (timePeriod - 1)
6	sumB = sum(x).double()
7	varB = sum2(x) - sumB*sumB/timePeriod
8	obs = mcount(close, timePeriod)
9	msumA = msum(close, timePeriod)
10	sumABDelta = (timePeriod - 1) * close + close.move(timePeriod) - msumA.prev() 
11	sumABDelta[timePeriod - 1 + 0:b] = NULL
12	sumABDelta[start - 1] =  wsum(close.subarray(b:start), x)
13	return (sumABDelta.cumsum() - msumA * sumB/obs)/varB
}
```
计算长度为1,000,000的向量的linearreg_slope序列，窗口长度为10时，TA-Lib耗时13ms，ta模块耗时14ms，两者几乎相等。这对于用脚本实现的ta来说，已属不易。当窗口增加到20时，TA-Lib的耗时增加到22ms，而ta的耗时仍为14ms。这说明TA-Lib的实现采用了循环，对每一个窗口分别计算，而ta则实现了向量化计算，与窗口长度无关。

### 4.4 减少数据复制的技巧

对向量进行slice，join，append等操作时，很有可能发生大量数据的复制。通常数据复制会比很多简单的计算更耗时。以下介绍如何减少数据复制的一些技巧。

#### 4.4.1 使用向量视图subarray减少数据复制

当使用某个向量的一部分元素进行计算时，例如若使用close[10:].avg()的语句，系统会从向量close中复制数据产生一个新的向量close[10:]再进行计算，不仅占用更多内存而且耗时。

函数`subarray`返回输入向量的一个subarray。它是原向量的一个视图，只记录了原向量的指针以及subarray的开始和结束位置。由于并没有分配大块内存来存储新向量，所以没有发生数据复制。所有向量的只读操作都可直接应用于subarray。`ema`和`linearreg_slope`的实现都大量使用了subarray。下面的例子中，我们对一个百万长度的向量进行100次slice操作，耗时62ms，每次操作耗时0.62ms。考虑到4.2中测试一个百万长度向量的ema操作耗时仅5ms，节约0.62ms是非常可观的。
```
close = rand(1.0, 1000000)
timer(100) close[10:]

Time elapsed: 62 ms
```

#### 4.4.2 为向量指定容量(capacity)避免扩容
在DolphinDB中，每一个向量都会被预分配一个内存容量。当我们向一个向量追加数据时，如果容量不够，那么系统会分配一个更大的内存空间，并把将旧的数据复制到新的内存空间，最后释放旧的内存空间。当向量比较大的时候，这个操作可能比较耗时。如果明确知道一个向量可能的最大的长度，那么事先指定这个长度为该向量的内存容量可以避免内存扩容的发生。向量的内存容量可通过DolphinDB内置函数`array`的capacity参数在创建向量时指定。譬如4.2小节中`ema`函数的第8行中先创建一个容量为n的向量，然后添加计算结果。