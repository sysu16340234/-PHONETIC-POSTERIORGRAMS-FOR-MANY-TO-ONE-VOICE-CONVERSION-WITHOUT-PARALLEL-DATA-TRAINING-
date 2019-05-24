# 论文阅读:"PHONETIC POSTERIORGRAMS FOR MANY-TO-ONE VOICE CONVERSION WITHOUT PARALLEL DATA TRAINING"

## 模型结构

---

### 1. 基本方法:基于DBLSTM的平行数据训练方法

 * 1.1. DBLSTM

DBLSTM(*Deep Bidirectional Long ShortTerm Memory,深度双向长短时记忆网络*)是一种特殊的双向LSTM,它的结构如下:

![](https://github.com/sysu16340234/-PHONETIC-POSTERIORGRAMS-FOR-MANY-TO-ONE-VOICE-CONVERSION-WITHOUT-PARALLEL-DATA-TRAINING-/blob/master/pics/DBLSTM.png?raw=true)

其中每个方块是一个循环单元,t-1,t和t+1分别代表过去时刻,现在时刻和将来时刻的输入输出帧;

 * 1.2. 基本结构

模型的基本结构由两部分组成,一部分是训练模块,一部分是转换模块,具体结构如下:

![](https://github.com/sysu16340234/-PHONETIC-POSTERIORGRAMS-FOR-MANY-TO-ONE-VOICE-CONVERSION-WITHOUT-PARALLEL-DATA-TRAINING-/blob/master/pics/DBLSTMNetWork.png?raw=true)

   - 1.2.1. 训练模块

训练模块首先使用STRAIGHT分析提取源音频和目标音频的频谱包络得到mel倒频谱表示,然后使用DTW对齐源音频和目标音频,最后把成对的MCEP特征作为DBLSTM的训练数据,DBLSTM采用BPTT算法训练;

   - 1.2.2. 转换模块
转换模块从源音频中提取基频F0和分周期部分AP以及MCEP特征,将MCEP特征输入训练好的DBLSTM模块,得到转换后的MCEP特征,然后均衡源音频和目标音频的均值和标准差得到转换后的Log F0,将转换后的F0和AP以及MCEP特征作为STRAIGHT声码器的输入,产生转换后的音频;

由于平行训练数据难以获得以及DTW的误差会很大地影响到最后的输出音频,所以论文不推荐这种方法,而是采用如下的替代方法:

### 2. 推荐方法:使用PPG的语音生成模型

* 1.1. PPG

PPG是时间-类别的矩阵,矩阵的元素代表着某个时间帧的语音属于某个语音类的后验概率,其中的语音类可以是一个特定的单词,发音或句音;

* 1.2.基本结构

![](https://github.com/sysu16340234/-PHONETIC-POSTERIORGRAMS-FOR-MANY-TO-ONE-VOICE-CONVERSION-WITHOUT-PARALLEL-DATA-TRAINING-/blob/master/pics/VCwithPPGs.png?raw=true)

模型由三个部分组成:训练部分1,训练部分2,转换部分;
  - 1.2.1. 训练部分1
  
  训练部分1使用多说话者语料库来训练SI-ASR系统,SI-ASR的输入是从语料库提取的MFCC帧Xt,输出是后验概率Pt =(p(s|Xt)|s = 1, 2, · · · , C),p(s|Xt)是语音类为s的后验概率;
  - 1.2.2. 训练部分2
  
  训练部分2的目的是训练一个DBLSTM来完成PPG到MCEP的转换,训练数据的PPG是由训练部分1中训练好的SI-ASR给出的,从目标音频中提取到的MFCC作为SI-ASR的输入,然后将PPG和目标音频的MCEP作为DBLSTM的训练数据,训练使用如下损失计算方式:
  
  ![](https://github.com/sysu16340234/-PHONETIC-POSTERIORGRAMS-FOR-MANY-TO-ONE-VOICE-CONVERSION-WITHOUT-PARALLEL-DATA-TRAINING-/blob/master/pics/loss.png?raw=true)
  
  其中YT是从目标音频中提取的MCEP特征,YR是真实值;
  
  - 1.2.3 转换模型
  
  转换模型中F0和AP的获得方法和上述相同,而训练部分2中训练好的DBLSTM的输入MCEP特征由训练模型1中训练好的SI-ASR的输出提供,而SI-ASR的输入由源音频中提取到的MFCC提供;最后STRAIGHT声码器使用DBLSTM输出的MCEP以及提取到的F0和AR作为输入;
  
  
---
# 实验:"deep-voice-conversion"

**实验步骤:**

* 执行train1.py训练模块1,使用eval1测试;

* 使用train2.py训练模块2,使用eval2测试;

* 执行convert.py获得结果;

**实验中的问题:**

* 报错:"train1.py: error: the following arguments are required: case"
  
  解决方法:传入参数的正确写法:train1.py [-h] [-ckpt CKPT] [-gpu GPU] case, -ckpt选择checkpoint,-gpu选择GPU,case是此次训练模型的名称,必须指定,否则无法运行;
  
* 报错:"train1.py line 76 error"

  解决方法:看到源码中的第76行,将print('case: {}, logdir: {}'.format(args.case1, args.case, logdir_train1))改为print('case: {}, logdir: {}'.format(args.case, logdir_train1))
  
* 报错:"OSError: [Errno 13] Permission denied: '/data'"

   解决方法:打开hparam目录下的default.yaml文件,将文件路径改为自己需要的路径,例如:
   
   ![](https://github.com/sysu16340234/-PHONETIC-POSTERIORGRAMS-FOR-MANY-TO-ONE-VOICE-CONVERSION-WITHOUT-PARALLEL-DATA-TRAINING-/blob/master/pics/path.png?raw=true)
   
   
* train1.py在训练epoch1时停留在0%

   解决方法:train1需要的训练数据TIMIT没有给出,需要自己下载;

**于第十二周**

* 报错:"NoBackendError"

   解决方法:是由缺少ffmpeg导致,按安装ffmpeg可以解决问题:
   
**于第十三周**

* 问题:无论artic还是bible的样本,convert的结果都不好,含有严重的机械音

   解决方法:尚未发现;
   
   
 **实验结果**
 
 **于第十二周**
 
 trian1:eproch 637:
 
 ![](https://github.com/sysu16340234/-PHONETIC-POSTERIORGRAMS-FOR-MANY-TO-ONE-VOICE-CONVERSION-WITHOUT-PARALLEL-DATA-TRAINING-/blob/master/pics/week12training.png?raw=true)
 
 **于第十三周**
 
 本周初步完成了两个网络的训练,在tensorboard查看训练效果如下:
 
 
 ![](https://github.com/sysu16340234/-PHONETIC-POSTERIORGRAMS-FOR-MANY-TO-ONE-VOICE-CONVERSION-WITHOUT-PARALLEL-DATA-TRAINING-/blob/master/tensorboard.png?raw=true)
 
 train1的loss为0.449,train2的loss为6.4006e-3
 
 本周分别采用artic的音频和bible的音频进行convert:
 
 ![](https://github.com/sysu16340234/-PHONETIC-POSTERIORGRAMS-FOR-MANY-TO-ONE-VOICE-CONVERSION-WITHOUT-PARALLEL-DATA-TRAINING-/blob/master/pics/convert.png?raw=true)
 








  
