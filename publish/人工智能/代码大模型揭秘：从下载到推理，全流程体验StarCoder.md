## 选择模型

### 模型榜单

大模型的发展日新月异，性能强劲的大模型不断涌现，可以实时关注开源大模型的榜单，选择合适自己的大模型

[开源大模型榜单](https://huggingface.co/spaces/HuggingFaceH4/open_llm_leaderboard)

![image-20240326160656604](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/image-20240326160656604.png)

[开源代码大模型榜单](https://huggingface.co/spaces/bigcode/bigcode-models-leaderboard)

![image-20240326160634333](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/image-20240326160634333.png)



### 模型网站

目前主流的下载模型的网站就是

- [huggingface](https://huggingface.co) 全球社区，模型发布的第一社区，需要科学上网
- [modelscope](https://www.modelscope.cn/home) 中文社区，模型不完整，更新滞后于huggingface



因为笔者最近对代码生成大模型十分感兴趣，所以这里就选择一个代码生成模型来下载和生成

这里我选的是starcoder2-7B模型，关于starcoder2模型的详细介绍可以看他们在huggingface上的[model card](https://huggingface.co/bigcode/starcoder2-7b)



## 下载模型

下载模型方式有两种方式

### 使用transformers下载

```python
from transformers import AutoModel, AutoTokenizer
model_name = 'starcoder2-7b'
model = AutoModel.from_pretrained(model_name)
```

这里是使用transformers加载模型，如果模型不存在则会去huggingface上下载模型，下载完成的模型会默认存储到`~/.cache/huggingface/hub`



### 使用huggingface_hub下载

```python
from huggingface_hub import snapshot_download
from tqdm.auto import tqdm

# 使 huggingface_hub 的下载与 tqdm 兼容
tqdm.pandas()

MODEL_URL = "bigcode/starcoder2-7b"
TARGET_PATH = "./starcoder2-7b"
SNAPSHOT_PATH = snapshot_download(MODEL_URL, cache_dir=TARGET_PATH, use_auth_token=True)

print(f"Downloaded successfully! Files are located in: {SNAPSHOT_PATH}")
```

这里使用的是huggingface_hub下载，可以通过`cache_dir`制定下载地址，并且这种方式还支持可视化脚本，类似这种：

```
Downloading README.md: 100%|██████████| 6.98k/6.98k [00:00<00:00, 38.0MB/s]
Fetching 18 files: 100%|██████████| 18/18 [00:01<00:00, 14.72it/s]
Downloaded successfully! Files are located in: ./starcoder2-7b/models--bigcode--starcoder2-7b/snapshots/42df1e961fe8546f0c891cea1d83c3b17631968f/
```



## 模型生成内容

### 下载依赖

[官网requirements.txt](https://github.com/bigcode-project/starcoder2/blob/main/requirements.txt)

```
git+https://github.com/huggingface/transformers.git
accelerate==0.27.1
datasets>=2.16.1
bitsandbytes==0.41.3
peft==0.8.2
trl==0.7.10
wandb==0.16.3
huggingface_hub==0.20.3
```

执行命令 `pip install -r requirements.txt`



### 推理脚本

```python
import torch
from transformers import AutoTokenizer, AutoModelForCausalLM

## 在指定的gpu上推理
gpu_index = 0
device = torch.device(f'cuda:{gpu_index}')

model_name = '/workspace/xieshijie/huggingface-models/starcoder2-7b/models--bigcode--starcoder2-7b/snapshots/42df1e961fe8546f0c891cea1d83c3b17631968f'
## 把模型通过transformer的AutoModelForCausalLM部署到指定GPU
model = AutoModelForCausalLM.from_pretrained(model_name).to(device)
tokenizer = AutoTokenizer.from_pretrained(model_name)

## 推理的输入
input_text = '    public U createFactory(T source) {\n        Supplier<U> tSupplier = SERVICE_MAP.get(source);\n        if (Objects.nonNull(tSupplier)) {\n            return tSupplier.get();\n        }\n\n        log.info(\"not found ServiceFactoryTemplate service, type == {}\", getUClass());\n        '
inputs = tokenizer.encode(input_text, return_tensors="pt", max_length=4096, truncation=True).to(device)
attention_mask = torch.ones(inputs.shape, dtype=torch.long).to(device)

## 推理的输出
outputs = model.generate(inputs, attention_mask=attention_mask, pad_token_id=tokenizer.eos_token_id, max_new_tokens=50)
decoded_output = tokenizer.decode(outputs[0], skip_special_tokens=True)

print(decoded_output)
```

1、本次加载starcoder2-7b模型，使用单个gpu加载，大约使用内容为28-29GB左右，这里没有使用量化等手段去优化内存使用。

2、使用`nvidia-smi`命令查看gpu的相关信息





### 待补全补全的原文

```java
    public U createFactory(T source) {
        Supplier<U> tSupplier = SERVICE_MAP.get(source);
        if (Objects.nonNull(tSupplier)) {
            return tSupplier.get();
        }

        log.info("not found ServiceFactoryTemplate service, type == {}", getUClass());
```



### 模型补全内容

```java
    public U createFactory(T source) {
        Supplier<U> tSupplier = SERVICE_MAP.get(source);
        if (Objects.nonNull(tSupplier)) {
            return tSupplier.get();
        }

        log.info("not found ServiceFactoryTemplate service, type == {}", getUClass());
         return null;
    }

    public Class<U> getUClass() {
        return uClass;
    }

    public void setUClass(Class<U> uClass) {
        this.uClass = uClass;
    }
```



## 更多惊喜

除了这些顶级的面试资料，我们的公众号还将定期分享：

- **最新互联网资讯**：让你时刻掌握行业动态。
- **AI前沿新闻**：紧跟技术潮流，不断提升自我。
- **技术分享与职业发展**：助你在职业生涯中走得更远、更稳。
- **程序员生活趣事**：让你在忙碌的工作之余找到共鸣与乐趣。





## 敬请关注【程序员世杰】

![](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/qrcode_for_gh_3223765a1430_430_899e57eb449c14150b4c0a82ab9b0fb6.jpg)

