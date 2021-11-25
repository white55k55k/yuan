# 渊
> 渊 - AI+文言文一站式附庸风雅, 欢迎贡献新的思路 笔记 模型 数据

一个文言诗词的NLP项目们。🌼

* [翻译](#翻译)
    * [现代文到文言文翻译器](#现代文到文言文翻译器)
    * [文言文到现代文翻译器](#文言文到现代文翻译器)

* [断句](#断句)


## 翻译
### 现代文到文言文翻译器
* 可以去[🤗 模型主页](https://huggingface.co/raynardj/wenyanwen-chinese-translate-to-ancient)体验或下载这个模型。
* 使用了这个[翻译句对的数据集](https://github.com/BangBOOM/Classical-Chinese)
* 感兴趣的可以参考[训练的笔记](nbs/zh2cc_translate.ipynb)

在python中推荐使用以下的代码进行inference：
```python
from transformers import (
  EncoderDecoderModel,
  AutoTokenizer
)
PRETRAINED = "raynardj/wenyanwen-chinese-translate-to-ancient"
tokenizer = AutoTokenizer.from_pretrained(PRETRAINED)
model = EncoderDecoderModel.from_pretrained(PRETRAINED)

def inference(text):
    tk_kwargs = dict(
      truncation=True,
      max_length=128,
      padding="max_length",
      return_tensors='pt')
   
    inputs = tokenizer([text,],**tk_kwargs)
    with torch.no_grad():
        return tokenizer.batch_decode(
            model.generate(
            inputs.input_ids,
            attention_mask=inputs.attention_mask,
            num_beams=3,
            max_length=128,
            bos_token_id=101,
            eos_token_id=tokenizer.sep_token_id,
            pad_token_id=tokenizer.pad_token_id,
        ), skip_special_tokens=True)
```
#### 目前版本的案例
目前版本， 按照上述通道的翻译案例：
```python
>>> inference('你连一百块都不肯给我')
['不 肯 与 我 百 钱 。']
>>> inference("他不能做长远的谋划")
['不 能 为 远 谋 。']
>>> inference("我们要干一番大事业")
['吾 属 当 举 大 事 。']
>>> inference("这感觉，已经不对，我努力，在挽回")
['此 之 谓 也 ， 已 不 可 矣 ， 我 勉 之 ， 以 回 之 。']
>>> inference("轻轻地我走了， 正如我轻轻地来， 我挥一挥衣袖，不带走一片云彩")
['轻 我 行 ， 如 我 轻 来 ， 挥 袂 不 携 一 片 云 。']
```

其中可改进处颇多:

* [ ] 目前，模型最长语句是128，可以通过修改tokenizer的max_length参数来调整。也就是会忽略一些现代文的语句。
* [ ] 可以通过去除pad token 的标签设置为-100，这样就不需要传eos token id了。
* [ ] 目前使用现代文预训练的bert-base-chinese作为encoder, 现代文预训练的 gpt2作为decoder。我们完全可以使用文言文+诗词预训练的gpt2作为decoder, 提升效果几乎是肯定的。
* [ ] 算力有限，许多调参细节，基本都没有试过。

### 文言文到现代文翻译器
> 输入文言文， 可以是**断句** 或者 **未断句**的文言文， 模型会预测现代文的表述。

* 欢迎前往[🤗 文言文（古文）到现代文的翻译器模型主页](https://huggingface.co/raynardj/wenyanwen-ancient-translate-to-modern)
* 训练语料是就是九十多万句句对， [数据集链接📚](https://github.com/BangBOOM/Classical-Chinese)。 训练时source序列（古文序列）， 按照50%的概率整句去除所有标点符号。
* 感兴趣的可以参考[训练的笔记](nbs/cc2zh_translate.ipynb),其中可改进处颇多。

#### 推荐的inference 通道
**注意**
* 你必须将```generate```函数的```eos_token_id```设置为102就可以翻译出完整的语句， 不然翻译完了会有残留的语句(因为做熵的时候用pad标签=-100导致)。
目前huggingface 页面上compute按钮会有这个问题， 推荐使用以下代码来得到翻译结果
* 请设置```generate```的参数```num_beams>=3```, 以达到较好的翻译效果
* 请设置```generate```的参数```max_length```256， 不然结果会吃掉句子
```python
from transformers import (
  EncoderDecoderModel,
  AutoTokenizer
)
PRETRAINED = "raynardj/wenyanwen-ancient-translate-to-modern"
tokenizer = AutoTokenizer.from_pretrained(PRETRAINED)
model = EncoderDecoderModel.from_pretrained(PRETRAINED)
def inference(text):
    tk_kwargs = dict(
      truncation=True,
      max_length=128,
      padding="max_length",
      return_tensors='pt')
   
    inputs = tokenizer([text,],**tk_kwargs)
    with torch.no_grad():
        return tokenizer.batch_decode(
            model.generate(
            inputs.input_ids,
            attention_mask=inputs.attention_mask,
            num_beams=3,
            max_length=256,
            bos_token_id=101,
            eos_token_id=tokenizer.sep_token_id,
            pad_token_id=tokenizer.pad_token_id,
        ), skip_special_tokens=True)
```
#### 目前版本的案例
> 当然， 拿比较熟知的语句过来， 通常会有些贻笑大方的失误， 大家如果有好玩的调戏案例， 也欢迎反馈
```python
>>> inference('非我族类其心必异')
['不 是 我 们 的 族 类 ， 他 们 的 心 思 必 然 不 同 。']
>>> inference('肉食者鄙未能远谋')
['吃 肉 的 人 鄙 陋 ， 不 能 长 远 谋 划 。']
# 这里我好几批模型都翻不出这个**输**字（甚至有一个版本翻成了秦始皇和汉武帝）， 可能并不是很古朴的用法， 
>>> inference('江山如此多娇引无数英雄竞折腰惜秦皇汉武略输文采唐宗宋祖稍逊风骚')
['江 山 如 此 多 ， 招 引 无 数 的 英 雄 ， 竞 相 折 腰 ， 可 惜 秦 皇 、 汉 武 ， 略 微 有 文 采 ， 唐 宗 、 宋 祖 稍 稍 逊 出 风 雅 。']
>>> inference("清风徐来水波不兴")
['清 风 慢 慢 吹 来 ， 水 波 不 兴 。']
>>> inference("无他唯手熟尔")
['没 有 别 的 事 ， 只 是 手 熟 罢 了 。']
>>> inference("此诚危急存亡之秋也")
['这 实 在 是 危 急 存 亡 的 时 候 。']
```

## 断句
> 输入一串未断句文言文， 可以断句， 目前支持二十多种标点符号

* 训练好的模型[这里可以下](https://huggingface.co/raynardj/classical-chinese-punctuation-guwen-biaodian)
* 使用了[【殆知阁v2.0数据集】](https://github.com/garychowcmu/daizhigev20)

这里推荐的Inference函数如下

```python
from transformers import AutoTokenizer, BertForTokenClassification
from transformers import pipeline

TAG = "raynardj/classical-chinese-punctuation-guwen-biaodian"
ner = pipeline("ner",module.model,tokenizer=tokenizer)

model = BertForTokenClassification.from_pretrained(TAG)
tokenizer = AutoTokenizer.from_pretrained(TAG)

def mark_sentence(x: str):
    outputs = ner(x)
    x_list = list(x)
    for i, output in enumerate(outputs):
        x_list.insert(output['end']+i, output['entity'])
    return "".join(x_list)
```

案例
```python
>>> mark_sentence("""郡邑置夫子庙于学以嵗时释奠盖自唐贞观以来未之或改我宋有天下因其制而损益之姑苏当浙右要区规模尤大更建炎戎马荡然无遗虽修学宫于荆榛瓦砾之余独殿宇未遑议也每春秋展礼于斋庐已则置不问殆为阙典今寳文阁直学士括苍梁公来牧之明年实绍兴十有一禩也二月上丁修祀既毕乃愓然自咎揖诸生而告之曰天子不以汝嘉为不肖俾再守兹土顾治民事神皆守之职惟是夫子之祀教化所基尤宜严且谨而拜跪荐祭之地卑陋乃尔其何以掲防妥灵汝嘉不敢避其责曩常去此弥年若有所负尚安得以罢輭自恕复累后人乎他日或克就绪愿与诸君落之于是谋之僚吏搜故府得遗材千枚取赢资以给其费鸠工庀役各举其任嵗月讫工民不与知像设礼器百用具修至于堂室廊序门牖垣墙皆一新之""")

'郡邑，置夫子庙于学，以嵗时释奠。盖自唐贞观以来，未之或改。我宋有天下因其制而损益之。姑苏当浙右要区，规模尤大，更建炎戎马，荡然无遗。虽修学宫于荆榛瓦砾之余，独殿宇未遑议也。每春秋展礼于斋庐，已则置不问，殆为阙典。今寳文阁直学士括苍梁公来牧之。明年，实绍兴十有一禩也。二月，上丁修祀既毕，乃愓然自咎，揖诸生而告之曰"天子不以汝嘉为不肖，俾再守兹土，顾治民事，神皆守之职。惟是夫子之祀，教化所基，尤宜严且谨。而拜跪荐祭之地，卑陋乃尔。其何以掲防妥灵？汝嘉不敢避其责。曩常去此弥年，若有所负，尚安得以罢輭自恕，复累后人乎！他日或克就绪，愿与诸君落之。于是谋之，僚吏搜故府，得遗材千枚，取赢资以给其费。鸠工庀役，各举其任。嵗月讫，工民不与知像，设礼器，百用具修。至于堂室。廊序。门牖。垣墙，皆一新之。'
```

### 可能会有的瑕疵
* 有时候两个标点符号连在一起， 会被吃掉一个， 比如```：【```会只有```【```
* 有时候标记的字太强了， 很难学会例外， 比如也字就很霸道， "吾生也有涯，而知也无涯。以有涯随无涯，殆已" 怎么都断不正确

欢迎联系我github的邮箱讨论，或者提交issue，我会尽力帮助你。