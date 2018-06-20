# ![img](http://www.dcjingsai.com/a_new_static/img/user/logo-new2.png?=v1528964803176&date=2018-6-11%2010:58:15):[Boutique Travel Services Predict](http://www.dcjingsai.com/common/cmpt/%E7%B2%BE%E5%93%81%E6%97%85%E8%A1%8C%E6%9C%8D%E5%8A%A1%E6%88%90%E5%8D%95%E9%A2%84%E6%B5%8B_%E6%8E%92%E8%A1%8C%E6%A6%9C.html)



## 比赛成绩

- 2018-02-08  AUC: 0.9764  B榜 **Rank 2**（Stacking：Catboost、Xgboost、LightGBM、Adaboost、RF etc.）
- 2018-02-07  AUC: 0.9589  A榜 **Rank 3**（Weight Average：0.65 * Catboost + 0.35 * XGBoost）
- 2018-01-21  AUC: 0.9733  半程**冠军**   &nbsp;&nbsp;&nbsp;（Single model：Catboost）

## 赛题简介

​	比赛方提供了5万多名用户在境外旅行APP（黄包车）中的浏览行为记录和历史订单记录（具体数据和字段如下五张表所示），其中用户在浏览APP之后有三种可能，购买精品旅游服务，或普通旅行服务，还有部分用户则不会下单。需要分析用户的个人信息、历史记录和浏览行为等，预测用户是否会在**短期内**购买精品旅游服务。（训练集浏览记录一百三十三万条，测试集33万条）

- Tab1 用户个人信息表（用户id、性别、省份、年龄段）
- Tab2 用户浏览记录表（用户id、行为类型、发生时间）
- Tab3 用户历史订单表（用户id、订单id、订单时间、订单类型、旅游城市、国家、大陆）
- Tab4 待预测订单表（id、订单类型 1 精品  0普通）
- Tab4 用户评论数据（用户id、订单id、评分、标签、评论内容）

## 比赛方案

1. 数据预处理：首先对数据进行清洗处理缺失值，浏览记录表中的1-4类无顺序，5-9类有顺序，一方面对**567*9这种补齐8操作**，另一方面发现订单历史记录中的下单时间戳和浏览记录的7操作时间一样，对于历史订单有订单但在浏览记录中对应时间点没有7操作的记录补齐7操作，还有基本信息缺失处理如性别的缺失处理等。
2. 特征工程：特征设计主要从 **（历史订单 + 浏览行为 + 时间特征 + 文本评论）** 这几方面展开，并根据特征方差和特征与label的相关系数&绘图进行特征选择，具体特征在如下。
3. 模型选择：由于其中包括浏览记录是属于**类别特征**，选用对类别特征直接支持且在泛化能力强不易过拟合的Catboost算法，和LightGBM算法。
4. 模型融合：最后模型融合使用Stacking的方式，特征分三份：第一层使用（参数不一样）的10个Catboost、xgboost和lightGBM训练，第二层使用xgboost融合，最后三个stacking结果再次融合，融合方法采用**概率大取更大、小取更小**，通俗的理解是在表现效果 (AUC) 相差不大的多个模型中，去选取对该条样本预测更自信的模型作为最终结果。（全集特征+两份有重合不完全特征80%（相关性强的特征分开））单独Stacking：0.9746，三份stacking融合0.97640，单模型0.9735
5. 由于部分用户浏览记录很少（只有几条），导致这些用户的很多特征维度为空，属于“**冷启动**”问题，单独建立在其历史特征和评论特征维度进行预测。

## 模型设计与模型融合

![model fusion](https://github.com/Bifzivkar/Boutique-Travel-Services-Predict/blob/master/model%20fusion.png)

## 特征工程

​	特征按照比赛时间进展在文件夹feature中，分别为1 ~ 10_extract_feature.py，以下根据特征所属类别（历史订单 + 浏览行为 + 时间特征 + 文本评论 + 交互）进行分类，具体特征提取方法可以看其中注释，另外特征工程运行时间较长，完整的特征文件下载已上传百度网盘：[精品旅行服务预测Rank2特征文件](https://pan.baidu.com/s/1rPwIQL1_FnVnaV8xDtzxiQ)，总结如下：

- **历史订单特征**

  -  历史订单数量
  -  历史出现精品订单 1 的数量和占比
  -  历史出现普通订单 0 的次数和占比
  -  用户最近一次出行是否为精品旅行 1
  -  历史纪录中城市的精品占比
  -  历史订单是否出现过精品订单 1 (leak)

  -  历史订单最近一次是什么类型 0 / 1

  -  历史订单最近一次去的州、国家、城市

- **浏览行为特征**（全部：指用户所有的浏览记录，对应：指该次购买对应的浏览记录）

  - 全部浏览记录中0-9出现的次数

  - 对应浏览记录中0-9出现的次数

  - 全部浏览记录浏览时间
  - 对应浏览记录浏览时间
  - 对应浏览记录是否出现5 6
  - 全部浏览记录是否出现56 67 78 89
  - 对应浏览记录是否出现56 67 78 89
  - 全部浏览记录是否出现567 678 789 566
  - 对应浏览记录是否出现567 678 789 566
  - 全部浏览记录是否出现5678 6789
  - 全部浏览记录是否出现56789
  - 对应浏览记录是否出现56789
  - action中大于6出现的次数
  - 对应点击2-4的和值 与 5-9 的比值
  - 全部点击2-4的和值 与 5-9 的比值
  - 对应浏览记录 1-9 操作所用平均时间
  - 全部浏览记录 1-9 操作所用平均时间
  - 全部action 最后一次 的类型
  - 全部 action 倒数第2-6次操作的类型
  - 最后1 2 3 4 次操作的时间间隔
  - 时间间隔的均值 最小值 最大值 方差
  - action 最后4 5 6 次操作时间的方差 和 均值
  - 对应浏览记录浏览平均时间(可以改成最近几天的)
  - 对应浏览记录 1-9 操作所用平均时间
  - 全部浏览记录 1-9 操作所用平均时间
  - 每日用户action的次数
  - 每日用户action的时间
  - 最近1周的使用次数 eval-auc:0.963724
  - 离最近的1-9的距离(间隔操作次数) 只取 56789 
  - 总体操作 1 2 3 4 5 6 7 8 9 次数的排名 rank
  - 对应操作 1 2 3 4 5 6 7 8 9 次数的排名 rank
  - 用户使用APP的天数，分别是否老用户
  - 用户当前时间距离最近历史订单时间间隔


- **文本评论特征**
  - 评论的长度
  - 评论的标签个数（强特征 涨分1个万）
  - 用户订单评分的统计特征（平均分、方差）
  - 用户评论各类分数的比例，最近一次评论的分数
  - 分用户普通订单评论的平均分和精品订单平均分
  - 使用snownlp对用户评论进行情感分析，统计用户订单评论得分
- **用户特征**
  - 是否新用户
  - 性别 是否男 是否女 是否缺失
  - 所属省份/城市（one-hot encode）
  - 所属年龄段（one-hot encode）
- **时间特征**
  - 以最近的浏览记录作为要预测的用户订单时间
  - 当前时间点的月份、当月第几天、星期几、是否周末
  - 用户历史订单最多的月份、当月第几天、星期几、是否周末
  - 是否为该城市的旅游旺季
  - 季节特征

## 运行环境和代码结构

1. **代码运行环境，包含主要软件和库**

   |  id  | software | version |  software  | version |
   | :--: | :------: | :-----: | :--------: | :-----: |
   |  1   |  ubuntu  |  16.04  |   pandas   | 0.20.2  |
   |  2   |  python  |  3.6.1  |   numpy    | 1.14.1  |
   |  3   | xgboost  |   0.4   |  sklearn   | 0.18.1  |
   |  4   | lightGBM |   0.1   |    tqdm    | 4.15.0  |
   |  5   | catboost |   0.5   | matplotlib |  2.0.2  |


2. **代码结构**
   - 在boutique_travel是整个比赛的代码，数据存储路径在
   - data文件夹用来存放数据，
     - 其中包含提取的特诊集（train6/test6、train_feature1/test_feature1、data_train/data_test）
     - submit文件夹是提交的结果文件，其下面子文件夹包含各个单模型的预测结果
   - feature文件夹存放提取特征的代码
     - 包含1 ~ 10_extract_feature.py 每部分代码对应提取的特征均有注释
   - model文件夹存放模型训练、预测和融合的代码
     - 1_submit.py是概率文件融合和修改预测结果为比赛要求的提交格式
     - 2~6分别是catboost、xgboost、lightGBM等的单模型和5折CV训练预测
     - 7是特征分三分，分别做两层的stacking learning，最后再对表现结果差不多的概率文件结果融合
     - model文件夹存储训练好的模型
   - 如有纰漏，欢迎指正 ^▽^

## License

This project is licensed under the terms of the MIT license.
