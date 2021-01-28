# PromQL常用函数


# PromQL 常用函数

## abs()

`abs(v instant-vector)`返回输入向量，所有样本值都转换为其**绝对值**。

## absent()

<br>`absent(v instant-vector)`如果传递给它的向量具有任何元素，则返回空向量;如果传递给它的向量没有元素，则返回值为 1 的 1 元素向量。</br>
<br>这对于在给定度量标准名称和标签组合不存在时间序列时发出警报非常有用。</br>

```bash
absent(nonexistent{job="myjob"})
# => {job="myjob"}
absent(nonexistent{job="myjob", instance=~".*"})
# => {job="myjob"}
absent(sum(nonexistent{job="myjob"}))
# => {}
```

<br>第在二个例子中，absent()试图从输入向量中导出 1 元素输出向量的标签。</br>

## ceil()

<br>`ceil(v instant-vector)` 将 v 中所有元素的样本值舍入到最接近的整数。</br>

## changes()

<br>对于每个输入时间系列，`changes(v range-vector)` 将返回其值在所提供的时间范围内更改的次数作为即时向量。</br>

## clamp_max()

<br>`clamp_max(v instant-vector, max scalar)`钳制 v 中所有元素的样本值，使其上限为 max。</br>

## clamp_min()

<br>`clamp_min(v instant-vector, min scalar)`钳制 v 中所有元素的样本值，使其下限为 min。</br>

## day_of_month()

<br>`day_of_month(v=vector(time()) instant-vector)`返回 UTC 中每个给定时间的月中的某天。 返回值为 1 到 31。</br>

## day_of_week()

<br>`day_of_week(v=vector(time()) instant-vector)`返回 UTC 中每个给定时间的星期几。 返回值为 0 到 6，其中 0 表示星期日等。</br>

## days_in_month()

<br>`days_in_month(v=vector(time()) instant-vector)`返回 UTC 中每个给定时间的月中天数。 返回值为 28 到 31。</br>

## hour()

<br>`hour(v=vector(time()) instant-vector)`返回 UTC 中每个给定时间的一天中的小时。 返回值为 0 到 23。</br>

## minute()

<br>`minute(v=vector(time()) instant-vector)`以 UTC 为单位返回每个给定时间的分钟。 返回值为 0 到 59。</br>

## month()

<br>`month(v=vector(time()) instant-vector)`返回 UTC 中每个给定时间的一年中的月份。 返回值为 2 到 12，其中 1 表示 1 月等。</br>

## year()

<br>`year(v=vector(time()) instant-vector)`以 UTC 格式返回每个给定时间的年份。</br>

## time()

<br>`time()`返回自 1970 年 1 月 1 日 UTC 以来的秒数。 请注意，这实际上并不返回当前时间，而是返回计算表达式的时间。</br>

## timestamp()

<br>`timestamp(v instant-vector)`返回给定向量的每个样本的时间戳，作为自 1970 年 1 月 1 日 UTC 以来的秒数。</br>

## delta()

<br>`delta(v range-vector)`计算范围向量 v 中每个时间系列元素的第一个和最后一个值之间的差值，返回具有给定增量和等效标签的即时向量。</br>
<br>`delta` 被外推以覆盖范围向量选择器中指定的全时间范围，因此即使样本值都是整数，也可以获得非整数结果。delta 应仅用于仪表。</br>

```
以下示例表达式返回现在和2小时之前CPU温度的差异：
delta(cpu_temp_celsius{host="zeus"}[2h])
```

## idelta()

<br>`idelta(v range-vector)`计算范围向量 v 中最后两个样本之间的差异，返回具有给定增量和等效标签的即时向量。`idelta`只能用于仪表。</br>

## deriv()

<br>`deriv(v range-vector)`函数，计算一个范围向量 v 中各个时间序列二阶导数，使用简单线性回归,`deriv`应仅用于仪表。</br>

## floor()

<br>`floor(v instant-vector)`将 v 中所有元素的样本值舍入为最接近的整数。</br>

## histogram_quantile()

<br>`histogram_quatile(φ float, b instant-vector)` 计算 b 向量的 φ-直方图 (0 ≤ φ ≤ 1) 。（有关 φ-分位数的详细解释和直方图度量类型的使用，请参见直方图和摘要。）b 中的样本是每个桶中的观察计数。 每个样本必须具有标签 le，其中标签值表示桶的包含上限。 （没有这种标签的样本会被忽略。）直方图度量标准类型自动提供带有\_bucket 后缀和相应标签的时间序列。
使用 rate()函数指定分位数计算的时间窗口。</br>

```bash
示例：直方图度量标准称为 http_request_duration_seconds。 要计算过去 10m 内请求持续时间的第 90 个百分位数，请使用以下表达式：
histogram_quantile(0.9, rate(http_request_duration_seconds_bucket[10m]))
在 http_request_duration_seconds 中为每个标签组合计算分位数。 要聚合，请在 rate()函数周围使用 sum()聚合器。 由于 histogram_quantile()需要 le 标签，因此必须将其包含在 by 子句中。 以下表达式按作业聚合第 90 个百分点：
histogram_quantile(0.9, sum(rate(http_request_duration_seconds_bucket[10m])) by (job, le))
要聚合所有内容，请仅指定 le 标签：
histogram_quantile(0.9, sum(rate(http_request_duration_seconds_bucket[10m])) by (le))
histogram_quantile()函数通过假设桶内的线性分布来插值分位数值。 最高桶必须具有+Inf 的上限。 （否则，返回 NaN。）如果分位数位于最高桶中，则返回第二个最高桶的上限。 如果该桶的上限大于 0，则假设最低桶的下限为 0.在这种情况下，在该桶内应用通常的线性插值。 否则，对于位于最低桶中的分位数，返回最低桶的上限。
如果 b 包含少于两个桶，则返回 NaN。 对于 φ<0，返回-Inf。 对于 φ> 1，返回+Inf。
```

## holt_winters()

<br>`holt_winters(v range-vector, sf scalar, tf scalar)`根据 v 中的范围产生时间序列的平滑值。平滑因子 sf 越低，对旧数据的重要性越高。 趋势因子 tf 越高，则考虑的数据趋势越多。 sf 和 tf 都必须介于 0 和 1 之间。`holt_winters`只能用于仪表。</br>

## increase()

<br>`increase(v range-vector)`计算范围向量中时间序列的增加。 单调性中断（例如由于目标重启而导致的计数器重置）会自动调整。 增加外推以覆盖范围向量选择器中指定的全时间范围，因此即使计数器仅以整数增量增加，也可以获得非整数结果。</br>

```bash
以下示例表达式返回范围向量中每个时间系列在过去5分钟内测量的HTTP请求数：
increase(http_requests_total{job="api-server"}[5m])
increase只应与counters一起使用。 它是rate(v)的语法糖乘以指定时间范围窗口下的秒数，应该主要用于人类可读性。 在记录规则中使用rate，以便每秒一致地跟踪增量。
```

## exp()

<br>`exp(v instant-vector)`计算 v 中所有元素的指数函数。特殊情况是：</br>

```bash
xp(+inf) = +Inf
Exp(NaN) = NaN
```

## ln()

<br>`ln(v instance-vector)`计算 v 中所有元素的自然对数。特殊情况是：</br>

```bash
ln(+Inf) = +Inf
ln(0) = -Inf
ln(x<0) = NaN
ln(NaN) = NaN
```

## log2()

<br>`log2(v instant-vector)`计算 v 中所有元素的二进制对数。特殊情况等同于 ln 中的特殊情况。</br>

## log10()

<br>`log10(v instant-vector)`计算 v 中所有元素的 10 进制对数。特殊情况等同于 ln 中的特殊情况。</br>

## sqrt()

<br>`sqrt(v instant-vector)`计算 v 中所有元素的平方根。</br>

## predict_linear()

<br>`predict_linear(v range-vector, t scalar)`根据范围向量 v 使用线性回归预测从现在起 t 秒的时间序列值。`predict_linear`只应与仪表一起使用。</br>

## rate()

<br>`rate(v range-vector)`计算范围向量中时间序列的每秒平均增长率。 单调性中断（例如由于目标重启而导致的计数器重置）会自动调整。 此外，计算推断到时间范围的末端，允许错过刮擦或刮擦循环与范围的时间段的不完美对齐。</br>

```bash
以下示例表达式返回范围向量中每个时间系列在过去5分钟内测量的每秒HTTP请求率：
rate(http_requests_total{job="api-server"}[5m])
rate应仅用于计数器。 它最适用于警报和缓慢移动计数器的图形。
注意，当将rate()与聚合运算符（例如sum()）或随时间聚合的函数（任何以_over_time结尾的函数）组合时，始终首先采用rate()，然后聚合。 否则，当目标重新启动时，rate()无法检测计数器重置。
```

## irate()

<br>`irate(v range-vector)`计算范围向量中时间序列的每秒即时增长率。 这基于最后两个数据点。 单调性中断（例如由于目标重启而导致的计数器重置）会自动调整。</br>

```bash
以下示例表达式返回范围向量中每个时间序列的两个最新数据点的最多5分钟的HTTP请求的每秒速率：
irate(http_requests_total{job="api-server"}[5m])
只应在绘制易失性快速移动计数器时使用irate。 警报和缓慢移动计数器的使用率，因为速率的简短更改可以重置FOR子句，并且难以阅读完全由稀有峰值组成的图形。
注意，当将irate()与聚合运算符（例如sum()）或随时间聚合的函数（任何以_over_time结尾的函数）组合时，请始终首先采用irate()，然后进行聚合。 否则，当目标重新启动时，irate()无法检测计数器重置。
```

## resets()

<br>对于每个时序数据，`resets()`在所提供的时间范围内返回计数器重置次数作为即时向量。 两个连续样本之间的值的任何减少都被解释为计数器重置。 `resets()`只应与计数器一起使用。</br>

## round()

<br>`round(v instant-vector, to_nearest 1= scalar)`将 v 中所有元素的样本值舍入为最接近的整数。 通过四舍五入解决关系。 可选的 to_nearest 参数允许指定样本值应舍入的最近倍数。 这个倍数也可能是一个分数。</br>

## scalar()

<br>给定单元素输入向量，`scalar(v instant-vector)`将该单个元素的样本值作为标量返回。 如果输入向量不具有恰好一个元素，则 scalar 将返回 NaN</br>

## sort()

<br>`sort(v instant-vector)`返回按其样本值排序的向量元素，按升序排列。</br>

## sort_desc()

<br>`sort(v instant-vector)`和`sort()`相同，但按降序排序。</br>

## vector()

<br>`vector(s scalar)`将标量 s 作为没有标签的向量返回。</br>

## <aggregation>\_over_time()

```bash
以下函数允许聚合给定范围向量的每个系列随时间的变化并返回具有每系列聚合结果的即时向量：
avg_over_time(range-vector): 范围向量内每个度量指标的平均值。
min_over_time(range-vector): 范围向量内每个度量指标的最小值。
max_over_time(range-vector): 范围向量内每个度量指标的最大值。
sum_over_time(range-vector): 范围向量内每个度量指标的求和值。
count_over_time(range-vector): 范围向量内每个度量指标的样本数据个数。
quantile_over_time(scalar, range-vector): 范围向量内每个度量指标的样本数据值分位数，φ-quantile (0 ≤ φ ≤ 1)
stddev_over_time(range-vector): 范围向量内每个度量指标的总体标准偏差。
stdvar_over_time(range-vector): 范围向量内每个度量指标的总体标准方差。
请注意，即使值在整个时间间隔内的间隔不均匀，指定时间间隔内的所有值在聚合中都具有相同的权重。
```

