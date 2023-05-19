# prometheus-study

## PromQL
### 数据类型
#### 介绍
PromQL表达式计算出来的值有一下几种类型：
+ 瞬时向量（Instant vector）：一组时序，每个时许只有一个采样值
+ 区间向量（Range vector）：一组时序，每个时序包含一段时间内的多个采样值
+ 标量数据（Scalar）：一个浮点数
+ 字符串（String）：一个字符串
  
#### 瞬时向量
普通非[]查询都是瞬时向量，即某个点的值
```
node_cpu_seconds_total{ip="127.0.0.1"}
```

#### 区间向量
以通过在瞬时向量选择器后面添加包含在[]里的时长来得到区间向量选择器
```
node_cpu_seconds_total{ip="127.0.0.1"}[5m]
```

### 匹配
通过PromQL查询时，经常需要根据label进行匹配，比如常用的tagKey=tagVal. PromQL目前主要支持完全匹配和正则匹配。

#### 完全匹配
PromQL支持使用=和!=两种完全匹配：

#### 正则匹配
PromQL支持使用正则表达式作为匹配条件，并且可以使用多个匹配条件。
正则匹配分为正向匹配和反向匹配：
+ 正向匹配：使用=~符号进行正向匹配，例如label=~regx，表示选择那些标签符合正则表达式定义的时间序列。
+ 反向匹配：使用!~符号进行反向匹配，例如label!~regx,表示排除那些标签符合正则表达式定义的时间序列。

#### 案例
##### 案例一：反向正则匹配
查询指标http_server_requets_seconds_count中，状态码为非2xx的序列：
```
http_server_request_seconds_count{status!~"2.."}
```
##### 案例二：正向正则匹配
查询指标prometheus_http_requests_total中，所有标签以/app/v1开头的记录，那么表达式为：
```
prometheus_http_request_total{handler=~"/api/v1/.*"}
```

### 内置函数
#### irate和rate
irate和rate都用于计算某个指标再一定时间间隔内的变化速率，但是计算方法不同：
+ irate：在指定范围内取最近两个数据点来算速率
  1. 适合快速变换的计数器
+ rate：取指定范围内的所有数据点，算出一组速率，然后取平均值作为结果
  1. 适合缓慢变化的计数器

#### 聚合函数sum, avg, max, min, count
+ sum: 参数是range-vector,
+ avg: 平均值
+ max：最大值
+ min：最小值
+ count：样本数量计数

#### absent
参数为瞬时向量instan-vector，当该向量非空时，返回空向量，否则返回返回不带名称值为1的指标，用来监控空数据的情况，即nodata监控

#### delta与idelta
+ delta：参数是一个区间向量，返回一个瞬时向量。计算一个区间向量v的第一个元素和最后一个元素之间的差值。
+ idelta：参数是一个区间向量，返回一个瞬时向量。它计算最新的2个样本值之间的差值。当区间仅有一个向量时无返回值。

#### bottomk与topk
+ bottomk：样本值最小的k个元素
+ topk：样本值最大的k个元素

#### 取整ceil和floor
+ ceil：参数为instance-vector，四舍五入取整
+ floor：参数为instance-vector，舍弃小数部分取整

### by的用法
与mysql的group by类似，即按照某个label进行聚合操作。
比如：
+ sum by(label)：按照label分类，然后每个分类进行sum聚合
+ avg by(lable)：按照label分类，然后每个分类进行avg聚合
+ max by(label)：按照label分类，然后每个分类进行max聚合
+ min by(label)：按照label分类，然后每个分类进行min聚合
+ count by(label)：按照label分类，然后每个分类进行count聚合