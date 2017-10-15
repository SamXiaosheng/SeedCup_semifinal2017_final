# 2017种子杯复赛</br>
### 使用语言及运行环境</br>
Python 2.7.13 on Linux</br>
需要模块：numpy,pandas,xgboost,sklearn</br></br>
### 代码相应接口及变量含义</br>
代码使用方法：1、python2 运行xgbost.py文件</br>
&#12288;&#12288;&#12288;&#12288;&#12288;&#12288;&#12288;2、python2 运行get1.py文件</br>
&#12288;&#12288;&#12288;&#12288;&#12288;&#12288;&#12288;3、python2 运行cat.py文件</br>
预测输出文件：pre.txt</br>
get_feature.py文件：</br>
<b>文件作用</b>：特征提取，最终生成相应文件</br>
<b>特征</b>：</br>
&#12288;&#12288;用户行为特征：距离预测日期的前一天，两天，三天，七天，十天，两周的行为特征</br>
&#12288;&#12288;商品特征：商品id，品牌id，类别id，店铺id，价格，这一个月里的浏览，收藏，加购，购买总次数</br>
&#12288;&#12288;用户特征：用户id，用户性别，年龄，会员等级，宝宝性别，宝宝年龄</br></br>
infodata.py文件：</br>
文件作用：数据文件预处理</br></br>
xgbost.py 文件：</br>
文件作用：xgboost模型搭建，训练并预测结果</br>
使用方法：直接python2 运行xgbost.py文件</br>
输出：会输出相应的第一个预测文件</br></br>
user_clear.py 文件：</br>
文件作用：数据文件预处理，修改或删除部分数据</br>
输出：将原始数据分成长度相近的5份精简数据</br></br>
get1.py文件：</br>
文件作用：获得第二个预测文件</br>
使用方法：直接python2 运行</br></br>
cat.py文件：</br>
文件作用：整合两个预测文件，生成最终预测文件</br>
使用方法：直接python2 运行</br>
输出：输出的pre.txt为最终预测文件</br></br>
### 数据特征提取思路</br>
1、用户特征：用户id，用户性别，年龄，会员等级，宝宝性别，宝宝年龄</br>
2、商品特征：商品id，品牌id，类别id，店铺id，价格，这一个月里的浏览，收藏，加购，购买总次数</br>
3、用户行为特征：距离预测日期的前一天，两天，三天，七天，十天，两周的浏览，收藏，加购，购买次数</br></br>
&#12288;&#12288;下图为模型训练后特征打分
![]https://github.com/slaxes/SeedCup_semifinal2017_final/blob/master/img/point.jpg)</br></br>
### 预测模型的选取</br>
&#12288;&#12288;需求分析：</br>
&#12288;&#12288;本次预测为潜在顾客的预测，这是一个弱预测，我们很难做到很高的准确率，所以，我们对模型的偏差要求并不高，而需要尽量压缩方差，所以我们需要一个方差小，偏差大的模型，同时，我们很难判断某个特征对我们的最终结果会产生多大的影响，我们需要一个将权值至0的操作，潜在用户是一个很难判断的事，很容易产生过拟合</br>
&#12288;&#12288;数据分析：</br>
&#12288;&#12288;拿到数据后，在洗数据的过程中，我发现我们能用的有效数据比较少，能够提取出来的特征数据也比较有限，而且特征数据中的关联度并不高</br>
&#12288;&#12288;模型选择：</br>
&#12288;&#12288;中和数据和需求，我排除了神经网络一类的模型，选择boost一类的模型，最终选择了xgboost模型</br></br>
&#12288;&#12288;下图为最终训练后生成的模型：</br>
![](https://github.com/slaxes/SeedCup_semifinal2017_final/blob/master/img/xgboost.jpg)</br></br>
### 对于模型参数的选择以及优化思路</br>
#### 参数选择：</br>
<b><li>数据优化</b>：对数据进行预处理的时候，发现数据的正负样本非常不平衡，大概比例为1:1000。所以对数据进行了平滑处理，缩减负样本数据量，最终正负样本比例为：1:10</br>
<b><li>数据提取</b>：采用滑动时间窗口，最终采用8个窗口，提取大概10w训练数据</br>
<b><li>模型优化</b>：在调参过程中</br>
&#12288;&#12288;（1）当模型树深度设置太大的时候，训练速度太过缓慢，且最终得到的结果不太理想。</br>
&#12288;&#12288;（2）设置最小子叶节点的权值和为1，防止过拟合</br>
&#12288;&#12288;（3）因为数据正负样本比例失调，设置scale_pos_weight，让模型更快收敛</br>
&#12288;&#12288;（4）迭代次数设置5w，当训练数据跌打次数设置太大时，模型预测auc呈现下降次数，最终选择5w次</br>
<b><li>多模型预测</b>：最终提取相应的加购id和模型预测结果进行比对，选取最合适的数据，将多次模型预测数据进行合并，最终生成预测文件</br></br>  
### 比赛整体思路</br>
1、 在拿到数据后，我们小组便先确定好了将会使用的模型。而我们首先对数据做了个预处理，由于behavior文件过长，我们采用mod5的方式将文件分成了长度相近的5份，修改了一些数据（如时间戳改日期）、也删掉了部分数据</br>
2、 注意到各种行为中“加入购物车”对购买相应商品的影响是最大的，我们便按照距8月26日的天数为参数，直接将之前若干天若干项加入购物车的记录转变为8月26日至28日的预测，我们的f1值也达到了0.0266，同时得到了正确预测的组数的范围</br>
3、 不断优化xgboost模型，将8月22日至24日的数据作为测试数据，进行机器学习</br>
4、 为了得到更高的准确率和召回率，我们将得到的两份测试文件进行了整合，最终获得了最优的一组数据</br></br>
### 比赛感想</br>
&#12288;&#12288;在模型选择正确的前提下，由于不同的模型注重的方向不同，其预测出的结果也会有差别。如果将两个模型的数据进行一定程度的整合，既能避免过度学习带来的过拟合，又能提高召回率。所以，对需要得到具体预测记录的任务，采用多个适合的模型进行机器学习或许能成为我们今后机器学习的一种新思路
### 开发者</br>
<li><a href=https://github.com/slaxes>slaxes<a></br>
<li><a href=https://github.com/sunhaohai>sunhaohai<a></br>
