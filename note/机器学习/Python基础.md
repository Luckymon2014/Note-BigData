### 主要内容
[1. Python基础](#1)
[2. 科学计算库Numpy基础](#2)
[3. 数据分析库Pandas基础](#3)
[4. Python数据可视化库Matplotlib](#4)
[5. Python机器学习裤Scikit-Learn](#5)

---

<h4 id='1'>Python基础</h4>

人生苦短，我用Python！
- 面向对象的解释型语言
- 语法简洁，强制空白符作为语句的缩进
- 功能模块共享时，必须开源
- 具有丰富强大的库，被称为胶水语言，能轻松地把其他语言制作的各种模块连接在一起
- Scikit-learn、SGBoost、TensorFlow、PySpark

环境安装
- 略

列表与元祖
字符串
字典（哈希表）

条件语句
循环
函数

---

<h4 id='2'>科学计算库Numpy基础</h4>

Python的数组运算结构复杂，浪费内存和CPU；不支持二维数组，没有各种运算函数。
因此基于C开发了Numpy库，弥补这些不足。

Numpy两种基本的对象
- ndarray
    - 存储单一数据类型的多维数组，统称为数组
- ufunc
    - 能够对数组进行处理的特殊函数

自动生成数组
- arange
- linspace

---

<h4 id='3'>数据分析库Pandas基础</h4>

Pandas两个常用对象
- Series
    - 定义了Numpy的ndarray对象的接口__array__()，可以用数组处理函数直接处理
    - 除数组功能外，还可以用索引标签，类似于字典
- DataFrame

---

<h4 id='4'>Python数据可视化库Matplotlib</h4>

略

---

<h4 id='5'>Python机器学习裤Scikit-Learn</h4>

基于Python语言，简单高效的数据挖掘和数据分析工具，建立在Numpy、SciPy和matplotlib上

- 简单高效的数据挖掘、数据分析的工具
- 对所有人开方，在很多场景易于复用
- BSD证书下开源

适合于单机版/入门，分布式/企业级用Spark

主要功能
- 测试数据集：sklearn.datasets
    - 乳腺癌、kddcup99、iris、加州房价等开源数据集
- 降维：Dimensionality Reduction
    - 为了特征筛选、统计可视化来减少属性的数量
- 特征提取：Feature extraction
    - 定义文件和图片中的属性
- 特征筛选：Feature selection
    - 为了建立监督学习模型而识别出有真实关系的属性
- 分类：Calssfication
    - 提供常用分类算法决策树、SVM、GBDT等进行有监督训练
- 回归：Regression
    - 提供常用回归算法，如：线性回归、SVR、GBDT等进行回归预测
- 聚类：Clustring
    - 使用KMeans之类的算法去给未标记的数据分类
- 交叉验证：Cross Validation
    - 去评估监督学习模型的性能
- 参数调优：Parameter Tuning
    - 去调整监督学习模型的参数以获取最大效果
- 流型计算：Manifold Learning
    - 去统计和描绘多维度的数据
