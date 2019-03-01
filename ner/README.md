## NER Recognizer 实体识别模型
命名实体识别在大多数框架中都使用了基于CRF-概率随机场的模型, 
在[CoreNLP](https://stanfordnlp.github.io/CoreNLP/ner.html)的实现中, 使用了自实现的CRF, 
[Hanlp](https://github.com/hankcs/HanLP)中支持了扩展CRF++与自实现的
[Perceptron感知机](https://github.com/hankcs/HanLP/wiki/%E7%BB%93%E6%9E%84%E5%8C%96%E6%84%9F%E7%9F%A5%E6%9C%BA%E6%A0%87%E6%B3%A8%E6%A1%86%E6%9E%B6)的模型, 
本项目将介绍两者模型的训练方式, Demo代码及效果对比.
特别感谢[ChineseNER](https://github.com/buppt/ChineseNER) 收集的来自Boson, MSRA 和人民日报的标注数据 

### CoreNLP框架
#### Traing Data Structure 训练集数据格式
```text
浙	B_product_name
江	M_product_name
在	M_product_name
线	M_product_name
杭	M_product_name
州	E_product_name
4	B_time
月	M_time
2	M_time
5	M_time
日	E_time
讯	O
```

#### Model Training 模型训练
```angular2
java -Xmx2g -cp corenlp-chinese-smoothnlp-*-jar-with-dependencies.jar edu.stanford.nlp.ie.crf.CRFClassifier -prop ner.model.props
```
关于模型配置文件更详细的信息, 可以看[官方FAQ文档](https://nlp.stanford.edu/software/crf-faq.html#a)
和[具体参数说明](https://nlp.stanford.edu/nlp/javadoc/javanlp/edu/stanford/nlp/ie/NERFeatureFactory.html)

*模型输出* : model.ser.gz

------------

### HanLP框架
### CRF (CRF++ Implementation)
Hanlp中支持了[crf++](https://taku910.github.io/crfpp/)的支持, 也给出了[相关文档](https://github.com/hankcs/HanLP/wiki/CRF%E8%AF%8D%E6%B3%95%E5%88%86%E6%9E%90)介绍训练方式, 到那内容中有一些关于crf++ 的细节没有提及, 且HanLP源码中也仅支持了CWS文档格式的接口. 

#### Feature Template 特征模板 
HanLP提供了一个静态写死的[特征模板](https://github.com/hankcs/HanLP/blob/master/src/main/java/com/hankcs/hanlp/model/crf/CRFNERecognizer.java)
```
@Override
    protected String getDefaultFeatureTemplate()
    {
        return "# Unigram\n" +
            // form
            "U0:%x[-2,0]\n" +
            "U1:%x[-1,0]\n" +
            "U2:%x[0,0]\n" +
            "U3:%x[1,0]\n" +
            "U4:%x[2,0]\n" +
            // pos
            "U5:%x[-2,1]\n" +
            "U6:%x[-1,1]\n" +
            "U7:%x[0,1]\n" +
            "U8:%x[1,1]\n" +
            "U9:%x[2,1]\n" +
            // pos 2-gram
            "UA:%x[-2,1]%x[-1,1]\n" +
            "UB:%x[-1,1]%x[0,1]\n" +
            "UC:%x[0,1]%x[1,1]\n" +
            "UD:%x[1,1]%x[2,1]\n" +
            "UE:%x[2,1]%x[3,1]\n" +
            "\n" +
            "# Bigram\n" +
            "B";
    }
```
这里稍微解释一下这份模板的含义, 更详尽的解释, 也可以看crf++给出的官方[文档](https://taku910.github.io/crfpp/#features)
```text
新      a       O
世纪    n       O
一九九八年      t       year  <<- 当前字符
新年    t       O
讲话    n       O

```
|模板 |  特征| 
|-----| -----|
|%x[0,0] | 一九九八年  | 
|%x[-1,0] | 世纪 |
|%x[1,0] | t |
|%x[2,0] | n | 

这里再提供另外一份CoNLL2000 Base-NP的模板
```
# Unigram
U00:%x[-2,0]
U01:%x[-1,0]
U02:%x[0,0]
U03:%x[1,0]
U04:%x[2,0]
U05:%x[-1,0]/%x[0,0]
U06:%x[0,0]/%x[1,0]

U10:%x[-2,1]
U11:%x[-1,1]
U12:%x[0,1]
U13:%x[1,1]
U14:%x[2,1]
U15:%x[-2,1]/%x[-1,1]
U16:%x[-1,1]/%x[0,1]
U17:%x[0,1]/%x[1,1]
U18:%x[1,1]/%x[2,1]

U20:%x[-2,1]/%x[-1,1]/%x[0,1]
U21:%x[-1,1]/%x[0,1]/%x[1,1]
U22:%x[0,1]/%x[1,1]/%x[2,1]

# Bigram
B   # 代表Bigram, 表示当前预测标签与前标签的组合也被做成特征放入模型
```

### TODO
在pku数据集上对比crf++中不同feature template对模型结果的影响. 
并将可配置模板集成到Smoothnlp Maven项目中. 