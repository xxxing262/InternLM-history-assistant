<div align="center">

</div>

# 基于InternLM大模型的中学历史学习助手
![](./attach/logo.jpg)

#### [书生·浦语大模型实战营](https://github.com/InternLM/tutorial)·大作业

> GitHub仓库地址：https://github.com/xxxing262/InternLM-history-assistant.git


# 简介

基于InternLM大模型的中学历史学习助手是一款专为中学生设计的历史学习辅助工具。它利用先进的自然语言处理技术和大数据分析，帮助学生更好地理解和掌握历史知识，帮助学生提高历史学习效率，提升历史知识理解和掌握程度。

# 数据来源

训练数据来源于2022年全国各地中考历史题，包含了**初中历史41个专题、125个考点**的若干考题。

# 项目部署

<details>
<summary id="setup">环境配置</summary>

下载项目仓库。

```sh
git clone https://github.com/xxxing262/InternLM-history-assistant.git

```

进入项目目录。

```sh
cd InternLM-History
```

创建虚拟环境。

```sh
conda create -n history python=3.9
```

进入虚拟环境。

```sh
conda activate history
```

安装依赖项
* 默认cuda版本为11.7，若不是请修改torch版本使与cuda版本对应，否则影响flash-attention安装
* 默认从Gitee下载`Flash-Attention`，若要从github下载，请将`setup.sh`中对应地址改为`https://github.com/Dao-AILab/flash-attention`
* 本步骤受网速和机器性能影响，时间可能在半小时以上

```sh
./setup.sh
```

</details>

<details>
<summary id="setup">启动项目</summary>

```sh
python app.py
```

</details>

# 项目开发

<details>
<summary id="prepare">数据集准备</summary>

数据来源于“[2022年中考历史真题分类-寒假刷题练.docx](datasets/src/2022_junior_middle_classification_all.docx)”，包含了初中历史32个专题、127个考点的若干考题。

#### 数据预处理

在项目根目录下，执行如下脚本，对文档中的题目进行初步拆分和清洗。

```sh
python scripts/0.preprocess_2022_junior_middle_classification_all.py
```

预处理得到的数据见[2022_junior_middle_classification_all.json](datasets/middle/2022_junior_middle_classification_all.json)。

#### 数据清洗

本步骤由**人工**进行，将含有图、表的题目剔除，以及拆分不正确的题目剔除。保证数据质量。

清洗得到的数据见[2022_junior_middle_classification_washed.json](datasets/middle/2022_junior_middle_classification_washed.json)。

#### 数据标准化

对题目数据的格式进行标准化，并使用json格式存储。将所有题目分为三种类型：选择题、填空题、综合题。

其中，选择题的格式如下：

```json
{
    "analysis":"根据所学可知，出土文物是当时真实开情况的遗存，是第一手资料，史料价值最高，D项正确；ABC三项，均是第二手资料，有参考价值，排除。故选D项。",
    "ans":3,
    "choices":[
        "当地传说",
        "地区风俗",
        "经典文献",
        "出土文物"
    ],
    "content":"下列材料都涉及了河姆渡居民生活的历史，其中史料价值最高的是",
    "origin":"2022年江苏连云港",
    "subject_id":1,
    "tid":5,
    "topic_id":2,
    "type":0
}
```
> 其中，`tid`为全局题号，`type`为题目类型（0为选择题），`subject_id`为该题目所属专题的编号，`topic_id`为改题目所属考点的编号，`origin`为题目来源，`content`为题干，`choices`列表为四个选项，`ans`为正确答案的索引，`analysis`为题目解析。

填空题和综合题的格式如下：

```json
{
    "analysis":"【详解】根据所学知识可知，唐太宗在位时，玄奘到天竺取经，在那里，他勤奋学习，成为了著名的佛学大师。后回到长安，口述成书《大唐西域记》；明成祖在位时，为了加强同海外各国的联系，派郑和率领船队出使西洋，增进了中国与亚非国家和地区的友好往来。",
    "ans":"玄奘；郑和",
    "content":"唐朝高僧（ ）（人物）西行前往天竺取经，历经磨难到达天竺，后回到长安，口述成书《大唐西域记》；明成祖派（ ）（人物）率领船队出使西洋，增进了中国与亚非国家和地区的友好往来。##n##",
    "origin":"2022年陕西",
    "subject_id":6,
    "tid":235,
    "topic_id":35,
    "type":1
}
```

> 其中，`tid`为全局题号，`type`为题目类型（1为填空题，2为综合题），`subject_id`为该题目所属专题的编号，`topic_id`为改题目所属考点的编号，`origin`为题目来源，`content`为题干，`ans`为正确答案，`analysis`为题目解析。

最终的题目标准存储格式为：

```json
{
    "subjects": [...]    // 32个专题的名称
    "topics": [...]      // 127个考点的名称
    "test":[             // 题目列表，用上述格式存储
        ...
    ]
}
```

使用如下脚本进行题目标准化：

```sh
python scripts/1.standardize_2022_junior_middle_classification_all.py
```

标准化后的数据见[2022_junior_middle_classification_std.json](datasets/middle/2022_junior_middle_classification_std.json)。

#### 转换为LLM训练数据集

运行如下脚本，转换为json格式的LLM训练数据集。

```sh
python scripts/2.convert_to_2022_junior_middle_history_dataset.py
```

转换好的数据集见[2022_junior_middle_history.json](datasets/2022_junior_middle_history.json)。

同时，按照7：3的比例将数据集切分为训练集和测试集。

```sh
python scripts/3.split_dataset.py
```

训练集见[2022_junior_middle_history_train.json](datasets/2022_junior_middle_history_train.json)，测试集见[2022_junior_middle_history_test.json](datasets/2022_junior_middle_history_test.json)。

</details>

<details>
<summary id="tune">微调模型</summary>

##### 下载模型

```sh
python scripts/4.download_internlm_chat_7b.py
```

##### 微调模型

```sh
xtuner train internlm_chat_7b_qlora_history_e3.py --deepspeed deepspeed_zero2
```

##### 模型转换

```sh
mkdir work_dirs/hf_internlm_chat_7b_history
export MKL_SERVICE_FORCE_INTEL=1

xtuner convert pth_to_hf ./internlm_chat_7b_qlora_history_e3.py ./work_dirs/internlm_chat_7b_qlora_history_e3/epoch_3.pth ./work_dirs/hf_internlm_chat_7b_history
```

##### 将Adapter合并到LLM

```sh
mkdir -p model/internlm-chat-7b-history
xtuner convert merge ./model/internlm-chat-7b ./work_dirs/hf_internlm_chat_7b_history ./model/internlm-chat-7b-history --max-shard-size 2GB
```

##### 测试模型

```sh
xtuner chat model/internlm-chat-7b-history/ \
--prompt-template internlm_chat \
--system "你是中学历史学习助手，内在是InternLM-7B大模型。你的开发者是安泓郡。开发你的目的是为了提升中学生对历史学科的学习效果。你将对中学历史知识点做详细、耐心、充分的解答。"
```

</details>

<details>
<summary>模型量化</summary>

##### 模型转换

```sh
lmdeploy convert internlm-chat-7b model/internlm-chat-7b-history/ --dst-path ./model/internlm-chat-7b-history-turbomind/
```

测试一波：

```sh
lmdeploy chat turbomind ./model/internlm-chat-7b-history-turbomind/ --meta_instruction "你是中学历史学习助手，内在是InternLM-7B大模型。你的开发者是安泓郡。开发你的 目的是为了提升中学生对历史学科的学习效果。你将对中学历史知识点做详细、耐心、充分的解答。"
```

##### 标定minmax

```sh
# 计算 minmax
lmdeploy lite calibrate \
  --model  model/internlm-chat-7b-history/ \
  --calib_dataset "ptb" \
  --calib_samples 128 \
  --calib_seqlen 2048 \
  --work_dir ./model/internlm-chat-7b-history-quant
```

##### 获取量化参数

```sh
lmdeploy lite auto_awq \
  --model  ./model/internlm-chat-7b/ \
  --w_bits 4 \
  --w_group_size 128 \
  --work_dir model/internlm-chat-7b-quant
```

##### 转换为TurboMind格式

```sh
lmdeploy convert  internlm-chat-7b \
    ./model/internlm-chat-7b-history-quant \
    --model-format awq \
    --group-size 128 \
    --dst_path model/InternLM-History-Model-TurboMind-W4A16/internlm-chat-7b-history-turbomind-w4a16
```

</details>
