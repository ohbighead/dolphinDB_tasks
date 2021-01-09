## 5. DolphinDB ta 指标列表

### Overlap Studies

**函数**|**语法**|**解释**
---|---|---
bBands|bBands(close, timePeriod, nbDevUp, nbDevDn, maType)|Bollinger Bands
dema|dema(close, timePeriod)|Double Exponential Moving Average
ema|ema(close, timePeriod)|Exponential Moving Average
kama|kama(close, timePeriod)|Kaufman Adaptive Moving Average
ma|ma(close, timePeriod, maType)|Moving average
mavp|mavp(inReal, periods, minPeriod, maxPeriod, maType)|Moving average with variable period
midPoint|midPoint(close, timePeriod)|MidPoint over period
midPrice|midPrice(low, high, timePeriod)|Midpoint Price over period
sma|sma(close, timePeriod)|Simple Moving Average
t3|t3(close, timePeriod, vfactor)|Triple Exponential Moving Average (T3)
tema|tema(close, timePeriod)|Triple Exponential Moving Average
trima|trima(close, timePeriod)|Triangular Moving Average
wma|wma(close, timePeriod)|Weighted Moving Average

### Momentum Indicators

**函数**|**语法**|**解释**
---|---|---
adx|adx(high, low, close, timePeriod)|Average Directional Movement Index
adxr|adxr(high, low, close, timePeriod)|Average Directional Movement Index Rating
apo|apo(close,fastPeriod,slowPeriod,maType)|Absolute Price Oscillator
aroon|aroon(high,low,timePeriod)|Aroon
aroonOsc|aroonOsc(high, low, timePeriod)|Aroon Oscillator
bop|bop(open, high, low, close)|Balance Of Power
cci|cci(high, low, close, timePeriod)|Commodity Channel Index
cmo|cmo(close, timePeriod)|Chande Momentum Oscillator
dx|dx(high, low, close, timePeriod)|Directional Movement Index
macd|macd(close, fastPeriod, slowPeriod, signalPeriod)|Moving Average Convergence/Divergence
macdExt|macdExt(close, fastPeriod, fastMaType, slowPeriod, slowMaType, signalPeriod, signalMaType)|MACD with controllable MA type
macdFix|macdFix(close, signalPeriod)|Moving Average Convergence/Divergence Fix 12/26
mfi|mfi(high, low, close, volume, timePeriod)|Money Flow Index
minus_di|minus_di(high, low, close, timePeriod)|Minus Directional Indicator
minus_dm|minus_dm(high, low, timePeriod)|Minus Directional Movement
mom|mom(close, timePeriod)|Momentum
plus_di|plus_di(high, low, close, timePeriod)|Plus Directional Indicator
plus_dm|plus_dm(high, low, timePeriod)|Plus Directional Movement
ppo|ppo(close, fastPeriod, slowPeriod, maType)|Percentage Price Oscillator
roc|roc(close, timePeriod)|Rate of change : ((price/prevPrice)-1)*100
rocp|rocp(close, timePeriod)|Rate of change Percentage: (price-prevPrice)/prevPrice
rocr|rocr(close, timePeriod)|Rate of change ratio: (price/prevPrice)
rocr100|rocr100(close, timeperiod)|Rate of change ratio 100 scale: (price/prevPrice)*100
rsi|rsi(close, timePeriod)|Relative Strength Index
stoch|stoch(high, low, close, fastkPeriod, slowkPeriod, slowkMatype, slowdPeriod, slowdMatype)|Stochastic
stochf|stochf(high, low, close, fastkPeriod, fastdPeriod, fastdMatype)|Stochastic Fast
stochRsi|stochRsi(real, timePeriod, fastkPeriod, fastdPeriod, fastdMatype)|Stochastic Relative Strength Index
trix|trix(close, timePeriod)|1-day Rate-Of-Change (ROC) of a Triple Smooth EMA
ultOsc|ultOsc(high, low, close, timePeriod1, timePeriod2, timePeriod3)|Ultimate Oscillator
willr|willr(high, low, close, timePeriod)|Williams' %R

### Volume Indicators

**函数**|**语法**|**解释**
---|---|---
ad|ad(high, low, close, volume)|Chaikin A/D Line
obv|obv(close, volume)|On Balance Volume

### Volatility Indicators

**函数**|**语法**|**解释**
---|---|---
atr|atr(high, low, close, timePeriod)|Average True Range
natr|natr(high, low, close, timePeriod)|Normalized Average True Range
trange|trange(high, low, close)|True Range

### Price Transform

**函数**|**语法**|**解释**
---|---|---
avgPrice|avgPrice(open, high, low, close)|Average Price
medPrice|medPrice(high, low)|Median Price
typPrice|typPrice(high, low, close)|Typical Price
wclPrice|wclPrice(high, low, close)|Weighted Close Price

### Statistic Functions

**函数**|**语法**|**解释**
---|---|---
beta|beta(high, low, timePeriod)|Beta
correl|correl(high, low, timePeriod)|Pearson's Correlation Coefficient (r)
linearreg|linearreg(close, timePeriod)|Linear Regression
linearreg_angle|linearreg_angle(close, timePeriod)|Linear Regression Angle
linearreg_intercept|linearreg_intercept(close, timePeriod)|Linear Regression Intercept
linearreg_slope|linearreg_slope(close, timePeriod)|Linear Regression Slope
stdDev|stdDev(close, timePeriod, nbdev)|Standard Deviation
tsf|tsf(close, timePeriod)|Time Series Forecast
var|var(close, timePeriod, nbdev)|Variance

### Other Functions
* 对Ta-Lib中的 Math Transform 与 Math Operators 类函数，可使用相应的DolphinDB内置函数代替。例如，Ta-Lib中的 SQRT, LN, SUM 函数，可分别使用DolphinDB中的 `sqrt`, `log`, `msum` 函数代替。
* 下列 Ta-Lib 函数尚未在ta模块中实现：所有 Pattern Recognition 与 Cycle Indicators 类函数，以及HT_TRENDLINE(Hilbert Transform - Instantaneous Trendline), ADOSC(Chaikin A/D Oscillator), MAMA(MESA Adaptive Moving Average), SAR(Parabolic SAR), SAREXT(Parabolic SAR - Extended)函数。

## 6. 路线图(Roadmap)

* 尚未实现的Ta-Lib函数将在未来版本中实现。
* 目前DolphinDB的自定义函数不支持默认参数，也不支持函数调用时基于键值来输入参数。这两点将在DolphinDB Server 1.20.0中实现，届时ta模块将实现跟TA-Lib一致的默认参数。
* 使用ta模块前必须使用 use ta 以加载，这在交互式查询中不尽方便。DolphinDB Server将在1.20版本中允许在系统初始化时预加载模块，ta模块函数与DolphinDB内置函数将拥有同等地位，以省去加载模块这个步骤。
