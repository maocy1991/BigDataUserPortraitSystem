# BigDataUserPortraitSystem

# 综述
随着互联网的快速发展，数据的不断增大，导致从海量数据中提取有用数据变的越来越困难，本系统旨在从复杂海量的数据中通过数据挖掘与可视化的方法大大降低对有用数据提取的难度。本系统主要运用技术包括hbase、flask、html。最终形成一个具有强交互性的画像系统，该系统可助使用者清晰直观地观察地域信息与用户信息。该系统完整且泛化性好，并且其数据储存与处理的工具为hbase，扩展能力强，因此该系统具有较高的参考价值与实用性。

# 数据收集
## 数据来源
本次实验使用了天池大数据比赛中所公布数据与国家数据中的2019年中国各省人均收入数据。并且在其基础上，进行数据增强。在进行数据增强的同时，将所得数据直接放入HBASE中。

## 数据增强标准
此处的数据增强运用的方法是，在一定规律的基础上，进行数据伪造。这些规律包括：各个地区的用户人均收入要符合2019年中国各省人均收入数据；学历高的人普遍比学历低的人收入要高；姓名是可以重复的，但手机号不可以重复。

## 源数据展示
经过数据增强与简单处理后，最终形成了四张表。
用户自然属性表，共20万条数据，行键为手机号，列族包括姓名、性别、年龄、身高、体重、居住地，部分数据展示如下表1。
![表1](https://s1.ax1x.com/2020/07/14/Ut3loF.png)

用户社会属性表，共20万条数据，行键为手机号，列族包括行业、职业、学历、收入，部分数据展示如下表2。
![表2](https://s1.ax1x.com/2020/07/14/Ut3wdO.png)

用户消费表，该表数据是20万用户自2019年8月1日之2019年10月8日，共十周70天的消费记录，该表一共有5498454条数据，列族包括物品名、价格、物品类型、日期，部分数据展示如下表3。
![表3](https://s1.ax1x.com/2020/07/14/Ut3fw8.png)

用户互联网行为表，该表数据是20万用户自2019年8月1日之2019年10月8日，共十周70天的互联网行为记录，该表一共有1400万条数据，列族包括新闻资讯、通信交流、娱乐休闲、生活服务、商务应用、工具使用、日期，部分数据展示如下表4。
![表4](https://s1.ax1x.com/2020/07/14/Ut3Hln.png)

# 数据分析
## 用户自然属性表
一个人的自然属性，几乎都可以直接用作标签，例如姓名、性别、年龄、居住地。在自然属性表中，以身高、体重数据为原数据，使用身体质量指数BMI公式（1）求出每个人的BMI指数，并根据图1中的中国标准，为每个人添加一条BMI标签：
BMI=weight/〖height〗^2

## 用户社会属性表
对每个人的社会属性增加了indLevel、eduLevel、incLevel三条数据。indLevel，以行业分类，求出各个行业的最高与最低收入，以该数据等距分成3份，这3份分别对应“新手”、“老手”与“精英”；incLevel，以地区分类，求出每个地区的最高与最低收入，等距分5份；eduLevel，将学历分成5份，“初中”->1，“高中”->2……“博士”->5。

## 用户消费表
将一个人的消费金额进行总合，为总消费额CSMoney；对每个人的消费记录进行计数，即为消费次数CSTimes；求出所有人中的最大最小总消费额与消费次数，并等距分成5份，对每个人的消费金额、消费次数打分CMLevel、CTLevel；对每个人的消费记录以物品类型分类，求出消费最多的物品类型与金额、消费第二多的物品类型与金额FCType、FCMoney、SCType、SCMoney；最后以消费数据对用户进行消费特征分析character。

## 用户互联网行为表
从数据中可知，互联网行为包括news、communications、domersticServices、entertainment、busApp、toolUse六种。对个人，将其六种互联网行为用时总合，即为个人互联网行为时间intTime；求出所有人中的最大最小互联网行为总用时，并等距分成5份，对每个人的互联网行为总用时打分intLevel；对一个用户，以不同互联网行为分类，求出每种互联网行为的总用时，求出用时最多与第二多的互联网行为与时间FIType、FITime、SIType、SITime。

# 地区特征数据分析
## 整体思路
首先，对每个用户以“place”标签分类，即将一个省的用户数据放一起。
对每个省，分别求出以下数据：每个省的男性数量与女性数量；每个省的年龄分布情况，其中年龄段为15～20、21～25，26～30……；每个省的各个行业中的人数；每个省的平均收入、最高收入、最低收入；每个省的每种学历的人数；每个省的最高消费金额、最低消费金额；每个省的每种互联网行为的平均时长。

## 亮点设计
在20万条用户数据中，查找特定某个省的全部数据，是一件困难且时间复杂度较高的一件事。并且，在后续的使用中，会多次以省为单位进行数据查找与统计。本人选择空间换时间的方法，进行优化，即将每个省的所有用户的行键保存本地。此后每次进行以省为单位的数据查找与统计时，就可以通过保存在本地的每个省的所有用户的行键信息进行处理，这样大大加快了处理速度。
其中，精准获取一个地区的所有人的数据，运用了HBASE的高级扫描方法
table.scan(filter="ValueFilter(=, 'substring:%s')" % place[i])，其中place是各个省的名字。

# 前端展示页面
总结小组讨论结果，将大数据画像系统网站大致分为三个页面：
1、导航页面：除了基本的网站修饰以外，核心部分为展示各个城市某一个指标的排行，当点击城市时进入该城市的画像页面（第二个页面）；
2、城市概况及其个人画像选择页面：该页面首先展示城市概况，比如：性别比例、收入水平、年龄水平、上网时间分布四个方面可视化展示；再进入数据预览的界面，通过查询框找出具体的个人，点击查看后跳转到个人画像（第三个页面）；
3、个人画像页面：该页面主要展示个人画像和个人信息数据可视化，以及用户特征图、用户消费行为图、用户互联网行为图。再加上一些用户的主要信息如消费次数、消费金额、主要消费方向的成列等。

![](https://s1.ax1x.com/2020/07/14/Ua3nH0.png)

![](https://s1.ax1x.com/2020/07/14/Ua3KEV.png)

![](https://s1.ax1x.com/2020/07/14/Ua3MNT.png)

![](https://s1.ax1x.com/2020/07/14/Ua3334.png)

![](https://s1.ax1x.com/2020/07/14/Ua38gJ.png)

![](https://s1.ax1x.com/2020/07/14/Ua3tD1.png)

![](https://s1.ax1x.com/2020/07/14/Ua3dUK.png)

![](https://s1.ax1x.com/2020/07/14/Ua3cDI.png)

![](https://s1.ax1x.com/2020/07/14/Ua3RVP.png)
