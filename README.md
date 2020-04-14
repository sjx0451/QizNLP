![](https://img.shields.io/pypi/v/QizNLP?logo=pypi)
![](https://img.shields.io/pypi/pyversions/QizNLP?logo=pypi) 
![](https://img.shields.io/pypi/l/QizNLP?logo=pypi)

**目录：**
* [QizNLP简介](#QizNLP简介)
* [安装流程](#安装流程)
* [使用示例](#使用示例)
   * [快速运行（使用默认数据训练）](#1.快速运行（使用默认数据训练）)
   * [使用自有数据](#2.使用自有数据)
* [框架设计思路](#框架设计思路)
* [公共模块](#公共模块)
* [修改适配需关注点](#修改适配需关注点)
   * [生成词表字典](#1、生成词表字典)
   * [数据处理相关](#2、数据处理相关)
   * [run和model的conf参数](#3、run和model的conf参数)
   * [使用分布式](#4、使用分布式)
* [TODO](#todo)
* [参考](#参考)
* [后记](#后记)


## QizNLP简介
QizNLP（Quick NLP）是一个面向NLP四大常见范式（分类、序列标注、匹配、生成），提供基本全流程（数据处理、模型训练、部署推断），基于TensorFlow的一套NLP框架。 
 
设计动机是为了在各场景下(实验/比赛/工作)，可快速灌入数据到模型，验证效果。从而在原型试验阶段可更快地了解到数据特点、学习难度及对比不同模型。 
 
QizNLP的特点如下：

* 封装了训练数据处理函数(TFRecord或Python原生数据两种方式）及词表生成函数。
* 针对分类、序列标注、匹配、生成这四种NLP任务提供了使用该框架进行模型训练的参考代码，可一键运行（提供了默认数据及默认模型）
* 封装了模型导出装载等函数，用以支持推断部署。提供部署参考代码。
* 封装了少量常用模型。封装了多种常用的TF神经网络操作函数。
* 封装了并提供了分布式训练方式（利用horovod）

讲在前头：

框架设计并非追求面面俱到，因为每个人在实践过程中需求是不一样的（如特殊的输入数据处理、特殊的训练评估打印保存等过程）。故框架仅是尽量将可复用功能封装为公共模块，然后为四大范式（分类/序列标注/匹配/生成）提供一个简单的训练使用示例，供使用者根据自己的情况进行参考修改。框架原则上是重在灵活性，故不可避免地牺牲了部分易用性。虽然这可能给零基础的初学者带来一定困难，但框架的设计初衷也是希望能作为NLP不同实践场景中的一个编码起点（相当于初始弹药库），并且能在个人需求不断变化时也能灵活进行适配及持续使用。
  
## 安装流程
项目依赖
```
python>=3.6
1.8<=tensorflow<=1.12
```
已发布pypi包，可直接通过```pip```下载
```shell script
pip install QizNLP
```

安装完毕后，进入到你自己创建的工作目录，输入以下命令：
```shell script
qiznlp_init
```
回车后，会在当前工作目录生成相关文件：
```bash
.
├── model	# 各个任务的模型代码示例
│   ├── cls_model.py
│   ├── mch_model.py
│   ├── s2l_model.py
│   └── s2s_model.py
├── run		# 各个任务的模型训练代码示例
│   ├── run_cls.py
│   ├── run_mch.py
│   ├── run_s2l.py
│   ├── run_s2s.py
├── deploy	# 模型载入及部署的代码示例
│   ├── example.py
│   └── web_API.py
└── data	# 各个任务的默认数据
    ├── train.toutiao.cls.txt
    ├── valid.toutiao.cls.txt
    ├── test.toutiao.cls.txt
    ├── train.char.bmes.txt
    ├── dev.char.bmes.txt
    ├── test.char.bmes.txt
    ├── mch_example_data.txt
    └── XHJ_5w.txt
```
注意：如果不是通过pip安装此项目而是直接从github上克隆项目源码，则进行后续操作前需将包显式加入python路径中：
```
export PYTHONPATH=$PYTHONPATH:克隆的qiznlp所在目录
```
## 使用示例
#### 1.快速运行（使用默认数据训练）
```shell script
cd run

# 运行分类任务
python run_cls.py

# 运行序列标注任务
python run_s2l.py

# 运行匹配任务
python run_mch.py

# 运行生成任务
python run_s2s.py
```
各任务默认数据说明

|任务|训练代码|调用的模型代码|默认模型|默认数据|备注|来源| 
|---|---|---|---|---|---|---|
|分类|run_cls.py|model_cls.py|TransEncoder+MHAtt_Pooling|train、valid、test.toutiao.cls.txt|头条新闻分类|https://github.com/luopeixiang/textclf|
|序列标注|run_s2l.py|model_s2l.py|BiLSTM+CRF|train、dev、test.char.bmes.txt|ResumeNER简历数据|https://github.com/jiesutd/LatticeLSTM|
|匹配|run_mch.py|model_mch.py|ESIM|mch_example_data.txt|ChineseSTS相似文本语义|https://github.com/IAdmireu/ChineseSTS|
|生成|run_s2s.py|model_s2s.py|Transformer|XHJ_5w.txt|小黄鸡闲聊5万|https://github.com/candlewill/Dialog_Corpus|
#### 2.使用自有数据
根据输入数据文本格式修改```run_*.py```中的```preprocess_raw_data()```函数，决定如何读取自有数据。
目前除```run_s2l.py```中，均已有```preprocess_raw_data()```的函数参考示例，其中默认文本格式如下：
* ```run_cls.py```：默认输入文本格式的每行为：```句子\t类别```
* ```run_mch.py```：默认输入文本格式的每行为：```句子1\t句子2```(正样本对，框架自行负采样)
* ```run_s2s.py```：默认输入文本格式的每行为：```句子1\t句子2```

然后在```run_*.py```中指定```train()```的数据处理函数为```preprocess_raw_data```，以```run_cls.py```为例：
```python
# 参数分别为：模型ckpt保存路径、自有数据文件路径、数据处理函数、训练batch size
rm_cls.train('cls_ckpt_1', '../data/cls_example_data.txt', preprocess_raw_data=preprocess_raw_data, batch_size=512)
```
具体更多细节可自行查阅代码，相信你能很快理解并根据自己的需求进行修改以适配自有数据 :)

这里有个彩蛋，第一次运行自有数据会不成功，需要对应修改```model_*.py```中conf与字典大小相关的参数，请按报错提示进行修改就ok。详情请参考下文：字典生成中的[提醒](#1、生成词表字典)
### 框架设计思路
——只有大体了解了框架设计才能更自由地进行适配 :)

一个NLP任务可以分为：数据选取及处理、模型输入输出设计、模型结构设计、训练及评估、推断或部署。
由此抽象出一些代码模块：
* run模块负责维护以下实例对象及方法：
	* TF中的sess/graph/config/saver
	* 分词方式、字典实例
	* 分布式训练（hvd）相关设置
	* 一个model实例
	* 模型的保存与恢复方法
	* 训练、评估方法
	* 基本的推断方法
	* 原始数据处理方法（数据集切分、分词、构造字典）


* model模块负责维护：
	* 输入输出设计
	* 模型结构设计
	* 输入输出签名暴露的接口
	* 从pb/meta恢复模型结构方法
	
model之间的主要区别是维护了自己特有的输入输出（即tf.placeholder的设计），故有以下实践建议：

**何时不需要新建model？**  
原则上只要输入输出不变，只有模型结构改变，则直接在原有model中增加代码并在初始化时选择新的结构。  
这种情况经常出现在针对某个任务的结构微调及试错过程中。

**何时需要新建model？**  
当输入输出有调整。  
例如想额外考虑词性特征，输入中要加入词性信息字段。或者要解决一个全新的任务。

**model与run的关系？**  
一般一个model对应一个专有run。新建model后则应新建一个相应run。  
原因主要考虑到run的训练评估过程需要与model的输入输出对齐。同时，model的不同输入可能也依赖于run进行特别的数据处理（如分词还是分字，词表大小，unk特殊规则等）

**model与run有哪些数据交互？**  
不同任务的主要区别包括如何对文本进行向量化（即token转id），需要设计分词、字典、如何生成向量、如何对齐到网络的placeholder。  
这里让model负责该向量化方法，run会将自己的分词、字典传过去。并且该向量化方法会被其它许多地方调用。举例：
* 生成向量方式会被应用于run的数据处理（生成tfrecord或原生py的pkl数据），以及对原始数据进行推断时的预处理
* 生成向量后对齐到placeholder的方式则会被应用在run的训练及推断。

### 公共模块
qiznlp包的公共模块文件如下：  
（因为是基本不需更改的基础模块，故没有在```qiznlp_init```命令初始化随其他文件一起复制到当前目录， 通过```qiznlp.common.*```调用）
```bash
└── common
    ├── modules  # 封装少量常用模型及各种TF的神经网络常用操作函数。
    ├── tfrecord_utils.py  # 封装TFRecord数据保存及读取操作。
    ├── train_helper.py  # 封装train过程中对数据的处理及其它相关方法。
    └── utils.py  # 基础类，个人对python中一些数据IO的简单封装，框架的许多IO操作调用了里面封装的方法，建议详细看看。
```

### 修改适配需关注点
##### 1、生成词表字典
```utils.Any2id```类封装了字典相关的功能,可通过传入文件进行初始化，在run中示例如下
```python
token2id_dct = {
	'word2id': utils.Any2Id.from_file(f'{curr_dir}/../data/toutiaoword2id.dct', use_line_no=True),
	}
# use_line_no参数表示直接使用字典文件中的行号作为id
```
如果传入的字典文件为空，则需要在run的数据处理函数中进行字典的构建，并保存到文件，方便下次直接读取
```python
# 循环中不断调用
token2id_dct['word2id'].to_count(cuted_sentent.split(' '))  # 迭代统计token信息，句子已分词，空格分隔

# 结束循环后构造字典
# 参数包括 预留token、最小词频及最大词表大小
token2id_dct['word2id'].rebuild_by_counter(restrict=['<pad>', '<unk>'], min_freq=1, max_vocab_size=20000)
token2id_dct['word2id'].save(f'{curr_dir}/../data/toutiaoword2id.dct')  # 保存到文件
```
提醒：在对自有数据进行训练时，由于模型的初始化比训练数据的处理要早，所以model中conf的vocab size/label size等相关参数只能先随便指定。同时等训练数据分词、构造字典完毕后，才能确定这些参数。  
目前解决方式是运行两次run：第一次运行构造字典完毕后，会检查模型vocab size等相关参数是否一致，不一致报错并返回信息。之后手工修正model中conf的相关参数后，再次运行。  
（当然也可以使用自己预定义的字典文件，然后在model中conf设置好正确的相关参数后直接运行run）

##### 2、数据处理相关
```preprocess_raw_data```返回训练、验证、测试数据的元组：
```
def preprocess_raw_data():
    # ...
    return train_items, dev_items, test_items  # 分别对应训练/验证/测试

    # 其中验证和测试可为None，此时模型将不进行相应验证或测试，如： 
    # return train_items, dev_items, None  # 不进行测试
```

##### 3、run和model的conf参数
run中的conf示例如下：
```
# run_cls.py
conf = utils.dict2obj({
    'early_stop_patience': None,  # 根据指标是否早停
    'just_save_best': True,  # 仅保存指标最好的模型（减少磁盘空间占用）
    'n_epochs': 20,  # 训练轮数
    'data_type': 'tfrecord',  # 训练数据处理成TFRecord
    # 'data_type': 'pkldata',  # 训练数据处理成py原生数据
})
# 前两者的具体指标通过修改train时相关方法的参数确定
```
model中conf示例如下：
```
# model_cls.py
conf = utils.dict2obj({
    "vocab_size": 14180,  # 词表大小，也就是上文所述构建字典后需注意对齐的参数
    'label_size': 16,  # 类别数量，需注意对齐
    "embed_size": 300,
    'hidden_size': 300,
    'num_heads': 6,
    'num_encoder_layers': 6,
    'dropout_rate': 0.2,
    'lr': 1e-3,
    'pretrain_emb': None,  # 不使用预训练词向量
    # 'pretrain_emb': np.load(f'{curr_dir}/pretrain_word_emb300.npy'),  # 使用预训练词向量(np格式)
})
```
具体参数可根据个人任务情况进行增删改。
##### 4、使用分布式
框架提供的分布式功能基于horovod（一种同步数据并行策略），即将batch数据分为多个小batch，分配到多机或多卡来训练。

前提：```pip install horovod```  
限制：只能用TFRecord数据格式（因为需利用其提供的分片shard功能）。且生成TFRecord的过程不好用多个worker,故实践建议分两次运行，一次正常运行生成数据及字典，再次运行才进行分布式训练  
操作步骤：需要先按照正常方式运行一遍，以```run_cls.py```为例，终端运行：
```
python run_cls.py
```
其中run的初始化为：
```
rm_cls = Run_Model_Cls('trans_mhattnpool')  # 默认初始化方式
```
等通过终端日志确定已生成完TFRecord数据后，```ctrl+c```退出  
继而修改初始化参数```use_hvd=True```：
```
rm_cls = Run_Model_Cls('trans_mhattnpool', use_hvd=True)
```
并在终端按照horovod要求的格式运行命令：
```
horovodrun -np 2 -H localhost:2 python run_cls.py
# -np 2 代表总的worker数量为2
# -H localhost:2 代表使用本机的2块GPU
# 注意此时需要在代码中预先设置好可见的GPU，如：os.environ['CUDA_VISIBLE_DEVICES'] = '0,1'
```
提醒：分布式训练中```train()```指定的```batch_size```参数即为有效（真实）batch size，内部会将batch切分为对应每个机或卡的小batch。故分布式训练实践中可在```train()```中直接指定大一点的```batch_size```。


## TODO
* 完善对model和run模块的单独说明
* 完善对公共module模块相关说明
* 完善对deploy模块相关说明
* 继续增加各任务默认模型
* 继续完善框架，保持灵活性的同时尽量增加易用性
* README.md英文化
* 增加更多其他任务（如多轮检索和生成、少样本学习等）

## 参考
[tensor2tensor](https://github.com/tensorflow/tensor2tensor)

## License
Mozilla Public License 2.0 (MPL 2.0)

## 后记
框架形成历程  
最早是在研究T2T官方transformer时，将transformer相关代码独立出来，作为弹药库。  
后续增加了自己写的S2S的beam_search代码（支持一些多样性方法），以及TF模型的导出部署代码。开始有点框架的味道。  
之后在解决各种任务类型时，不断重构，追求较灵活的设计方案，终得到现版本。
  
深知目前还有许多可改进的地方，希望大家能一起加入和改进！
