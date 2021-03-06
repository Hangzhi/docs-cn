---
title: Key Visualizer 流量可视化
category: how-to
---

# Key Visualizer 流量可视化

Key Visualizer 是一款用于分析 TiDB 使用模式和排查流量热点的工具。它会将 TiDB 数据库集群一段时间的指标生成可视化报告，可用于快速直观地观察集群整体热点及流量分布情况。

## 访问 Key Visualizer

Key Visualizer 功能作为 TiDB Dashboard 组件的功能之一，直接集成在 PD 实例上，无需单独部署。可通过以下方式在浏览器中访问任一 PD 实例上的 TiDB Dashboard：

{{< copyable "" >}}

```
http://PDAddress:PDPort/dashboard
```

> **注意：**
>
> + 存在多个 PD 实例时，可通过任意一个 PD 的地址访问 TiDB Dashboard。
> + 默认情况下 `PDPort` 是 `2379`。若部署时修改过 PD 相应参数，则需要填写对应的端口。
> + 登录 TiDB Dashboard 需要使用 TiDB 的 `root` 账号。

## 界面示例

下图为一个 Key Visualizer 页面示例

![Key Visualizer 示例图](/media/dashboard/keyvisualizer/overview.png)

从以上 Key Visualizer 界面可以观察到以下信息：

* 一个大型热力图，显示整体访问流量随时间的变化情况。
* 热力图某个坐标的详细信息。
* 左侧为表、索引等信息。

## 概念介绍

本部分介绍与 Key Visualizer 相关的基本概念。

### Region

在 TiDB 集群中，数据以分布式的方式存储在所有的 TiKV 实例中。TiKV 在逻辑上是一个巨大且有序的 KV Map。整个 Key-Value 空间分成很多 Region，每一个 Region 是一系列连续的 Key。

> **注意：**
>
> 关于 Region 的详细介绍，请参考[三篇文章了解 TiDB 技术内幕 - 说存储](https://pingcap.com/blog-cn/tidb-internal-1/#region)

### 热点

在使用 TiDB 的过程中，热点是一个典型的现象，它表现为大量的流量都在读写一小块数据。由于连续的数据往往由同一个 TiKV 实例处理，因此热点对应的 TiKV 实例的性能就成为了整个业务的性能瓶颈。常见的热点场景有使用自增主键连续写入相邻数据导致的写入表数据热点、时间索引下写入相邻时间数据导致的写入表索引热点等。

> **注意：**
>
> 热点问题详情请参阅 [TiDB 热点问题详解](https://asktug.com/t/topic/358)。

### 热力图

热力图是 Key Visualizer 的核心，它显示了一个指标随时间的变化。热力图的横轴 X 是时间，纵轴 Y 则是按 Key 排序的连续 Region，横跨 TiDB 集群上所有数据库和数据表。颜色越暗 (cold) 表示该区域的 Region 在这个时间段上读写流量较低，颜色越亮 (hot) 表示读写流量越高，即越热。

### Region 压缩

一个 TiDB 集群中，Region 的数量可能多达数十万。在屏幕上是难以显示这么多 Region 信息的。因此，在一张热力图中，Region 会被压缩到约 1500 个连续范围，每个范围称为 Bucket。在一个热力图上，热的实例更需要关注，因此 Region 压缩总是倾向于将流量较小的大量 Region 压缩为一个 Bucket，而尽量让高流量的 Region 独立成 Bucket。

## 使用 Key Visualizer

本部分介绍如何使用 Key Visualizer。

![工具栏](/media/dashboard/keyvisualizer/toolbar.png)

### 观察某一段时间或者 Region 范围

打开 Key Visualizer 时，默认会显示最近六小时整个数据库内的热力图。其中，越靠近右侧（当前时间）时，每列 Bucket 对应的时间间隔会越小。如果你想观察某个特定时间段或者特定的 Region 范围，则可以通过放大来获得更多细节。具体操作描述如下：

* 在热力图中向上或向下滚动。
* 点击 **Select & Zoom** 按钮，然后点击并拖动以选择要放大的区域。
* 点击 **Reset** 按钮，将 Region 范围重置为整个数据库。
* 点击 **时间选择框**（以上界面的 `6 hours` 处），重新选择观察时间段。

> **注意：**
>
> 使用后三种方法，将引起热力图的重新绘制。你可能观察到热力图与放大前有较大差异。这是一个正常的现象，可能是由于在进行局部观察时，Region 压缩的粒度发生了变化，或者是局部范围内，“热”的基准发生了改变。

### 调整亮度

热力图使用颜色的明暗来表达一个 Bucket 的流量高低，颜色越暗 (cold) 表示该区域的 Region 在这个时间段上读写流量较低，颜色越亮 (hot) 表示读写流量越高，即越热。如果热力图中的颜色太亮或太暗，则可能很难观察到细节。此时，可以点击 **Brightness** 按钮，然后通过滑块来调节页面的亮度。

> **注意：**
>
> Key Visualizer 在显示一个区域内的热力图时，会根据区域内的流量情况来界定冷热。当整个区域流量较为平均时，即使整体流量在数值上很低，你依然有可能观察到较大的亮色区域。请注意一定要结合数值一起分析。

### 选择指标

你可以通过 **指标选择框**（以上界面中 `Write (bytes)` 处）来查看你关心的指标：

* `Read (bytes)`：读流量
* `Write (bytes)`：写流量
* `Read (keys)`：读取行数
* `Write (keys)`：写入行数
* `All`：读写流量的总和

### 刷新与自动刷新

可以通过点击 **Refresh** 按钮来重新获得基于当前时间的热力图。当需要实时观察数据库的流量分布情况时，可以点击 **Refresh** 右侧的向下箭头，选择一个固定的时间间隔让热力图按此间隔自动刷新。

> **注意：**
>
> 如果你进行了时间范围或者 Region 范围的调整，自动刷新会被关闭。

### 查看详情

可以将鼠标悬停在你所关注的 Bucket 上，来查看这个区域的详细信息。详细信息如下图所示：

<img src="../../media/dashboard/keyvisualizer/tooltip.png" width="50%" />

如果需要复制某个信息，可以进行点击 Bucket。此时相关详细信息的页面会被暂时钉住。点击你关注的信息，即可将其复制到剪切板。详细信息页面示例图如下：

<img src="../../media/dashboard/keyvisualizer/tooltip-copy.png" width="50%" />

## 常见热力图解读

本部分选取了 Key Visualizer 中常见的四种热力图进行解读。

### 均衡：期望结果

<img src="../../media/dashboard/keyvisualizer/well_dist.png" width="60%" />

如上图所示，热力图颜色均匀或者深色和亮色混合良好，说明读取或写入在时间和 Region 空间范围上都分布得比较均衡，访问压力均匀地分摊在所有的机器上。这种负载是最适合分布式数据库的。

### X 轴明暗交替：需要关注高峰期的资源情况

<img src="../../media/dashboard/keyvisualizer/period.png" width="60%" />

如上图所示，热力图在 X 轴（时间）上表现出明暗交替，但 Y 轴 (Region) 则比较均匀，说明读取或写入负载具有周期性的变化。这种情况可能出现在周期性的定时任务场景，如大数据平台每天定时从 TiDB 中抽取数据。一般来说可以关注一下使用高峰时期资源是否充裕。

## Y 轴明暗交替：需要关注产生的热点聚集程度

<img src="../../media/dashboard/keyvisualizer/continue.png" width="60%" />

如上图所示，热力图包含几个明亮的条纹，从 Y 轴来看条纹周围都是暗的，这表明明亮条纹区域的 Region 有很高的读写流量，可以从业务角度观察一下是否符合预期。例如，所有业务都关联用户表的情况下，用户表的整体流量就会很高，那么在热力图中表现为亮色区域就非常合理。

另外，明亮区域的高度（Y 轴方向的粗细）非常关键。由于 TiKV 自身拥有以 Region 为单位的热点平衡机制，因此涉及热点的 Region 越多其实越能有利于在所有 TiKV 实例上均衡流量。明亮条纹越粗、数量越多则意味着热点越分散、更多的 TiKV 能得到利用；明亮条纹越细、数量越少意味着热点越集中、热点 TiKV 越显著、越需要人工介入并关注。

### 明亮斜线：需要关注业务模式

<img src="../../media/dashboard/keyvisualizer/sequential.png" width="60%" />

如图上所示，热力图显示了明亮的斜线，表明读写的 Region 是连续的。这种场景常常出现在带索引的数据导入或者扫描阶段。例如，向自增 ID 的表进行连续写入等等。图中明亮部分对应的 Region 是读写流量的热点，往往会成为整个集群的性能问题所在。这种时候，可能需要业务重新调整主键，尽可能打散以将压力分散在多个 Region 上，或者选择将业务任务安排在低峰期。

> **注意：**
>
> 这里只是列出了几种常见的热力图模式。Key Visualizer 中实际展示的是整个集群上所有数据库、数据表的热力图，因此非常有可能在不同的区域观察到不同的热力图模式，也可能观察到多种热力图模式的混合结果。使用的时候应当视实际情况灵活判断。

## 解决热点问题

TiDB 内置了不少帮助缓解常见热点问题的功能，深入了解请参考 [TiDB 高并发写入场景最佳实践](/reference/best-practices/high-concurrency.md)。
