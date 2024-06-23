> ***我们即将见证一个新的时代！这是最好的时代，也是最坏的时代！***



## 需求背景



> **背景：**

平时会编写博客，并且会把这个博客上传到github上，然后自己买一个域名挂到github上。

我平时编写的博客会有一些图片来辅助说明的，写完之后如果我把图片和文字全部都上传到博客网站，后期图片很多时就会导致网站加载特别慢

所以想把图片存储在一个公共的对象存储平台（腾讯云的cos服务），这样只要上传一个公共访问链接即可，极大的减少存储空间。



> **需求：**

每次写完博客都要手动上传图片，然后把得到的链接在复制到本地的markdown文件中，如果一篇文章的图片特别多，这简直就是灾难！所以我想

1. 给一个文件路径，自动把markdown文件中本地的图片上传到腾讯云的cos平台，并获取公共链接

2. 把本地的文章的链接自动替换为公共链接



> **调研和设计**

腾讯云cos服务是提供这样的接口的，但是接口需要鉴权，所以我把上诉的需求拆解为三部分

- 生成鉴权sign_key
- 调用腾讯云的接口上传图片，并获取链接
- 输入markdown文件，找到本地图片链接，上传并替换为公共链接



---



## **错误示范**

> **我阅读腾讯云的cos文档，需要提供签名，一共需要八个步骤才能生成**

![](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/out-20230411231504053.png)
![](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/out-20230411231505048.png)



> **我把这八个过程每一个过程都复制给chatgpt，让他帮我写**

![](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/out-20230411231505441.png)
![](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/out-20230411231505934.png)![](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/out-20230411231506839.png)



## 总结



> **后面太多我就不演示了，我直接说总结：**

- 在这个过程中给我的代码基本都是可以运行的，我只需要微调一下就可以用，报错直接扔给他也基本都可以解决
- 但是整个过程还是需要我理解每一步，并且把这八个步骤进行汇总和调整
- **比Google搜索要好用一点，但是还没有那么好用，总感觉差点意思**



---



## **正确示范**

> **生成上面的签名之后，我就让他去写上传接口**

![](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/out-20230411231507083.png)
![](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/out-20230411231508211.png)



仔细阅读一下上面的第二张图片，我要求**【****把其中的url的uri替换为文件名****】**，然后它给我代码并解释这是做什么，它说这是上传到**【腾讯云的cos服务】**，要知道我在这之前是完全没有提到过**【腾讯云】、【cos服务】**等字眼的，也就是它只依赖我提供的host**【**[**coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com**](http://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/)**】**这个域名，就判断出我这是腾讯云的cos服务，这就有点厉害了，更厉害的是，当然要求他写一个测试用例：





![](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/out-20230411231507360.png)



> **它直接把我之前费力八个步骤生成的签名直接生成了！这太离谱了！**

- **我以为：**chatgpt不知道腾讯云cos签名的生成过程，然后我阅读文档，把八个步骤重组并依次喂给chatgpt，让他帮我写
- **实际上：**chatgpt不仅知道腾讯云cos，在我要求他上传时，就自动把官方推荐的生成签名的方式给我生成了！



> **这意味着以后大部分网络开源的内容，你甚至不用阅读啃文档，你只要知道一个概念，剩下的就交给他就可以！**



后面的上传和替换markdown的内容我就不截图，后面对于chatgpt就非常简单了，最后附上完整的代码(95%是chatgpt写的)

```python
# -*- coding=utf-8
import hmac
import hashlib
import os
import re
import requests

# 定义 SecretKey 和 KeyTime
# 替换为自己的key和id
secret_key = "xieshijie_key"
key_time = "1680947045;2980950645"
secret_id = "xieshijie_id"
http_method = 'put'


def generate_url_param_list():return ""


def generate_header_list():return ""


def generate_sign_key_val(uri_pathname):# 生成 SignKey
    sign_key = hmac.new(
        secret_key.encode(),
        key_time.encode(),
        hashlib.sha1
    ).hexdigest()

    # 生成 HttpString
    http_string = f"{http_method.lower()}\n{uri_pathname}\n"

    http_string += "\n"

    http_string += "\n"

    # 生成 StringToSign
    string_to_sign = f"sha1\n{key_time}\n{hashlib.sha1(http_string.encode()).hexdigest()}\n"

    # 生成 Signature
    signature = hmac.new(
        sign_key.encode(),
        string_to_sign.encode(),
        hashlib.sha1
    ).hexdigest()

    # 生成签名
    sign_key_val = f"q-sign-algorithm=sha1" \
                   f"&q-ak={secret_id}" \
                   f"&q-sign-time={key_time}" \
                   f"&q-key-time={key_time}" \
                   f"&q-header-list={generate_header_list()}" \
                   f"&q-url-param-list={generate_url_param_list()}" \
                   f"&q-signature={signature}"

    return sign_key_val


def generate_file_name(file_path):# 获取文件名return file_path.split('/')[-1]


def put_request(file_path, sign_key):# 设置请求头
    headers = {
        'host': 'coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com',
        'Authorization': sign_key
    }

    # 设置请求体with open(file_path, 'rb') as f:
        data = f.read()

    uri = generate_file_name(file_path)

    # 发送 PUT 请求
    url = f'https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/{uri}'
    response = requests.put(url, headers=headers, data=data)

    # 判断响应状态码并返回结果if response.status_code == 200:
        print('本地图片上传成功')
        return url
    else:
        print('本地图片上传失败')
        return 'fail'


def replace_local_image_links(file_path):﻿"""
    读取Markdown文件，提取其中的本地图片链接并替换为基于图片名称计算的新链接。
    Args:
        file_path (str): Markdown文件路径。
    """# 判断文件是否是Markdown格式if not file_path.endswith('.md'):
        print('该文件不是Markdown格式。')
        return

    # 读取Markdown文件内容with open(file_path, 'r', encoding='utf-8') as f:
        content = f.read()

    # 匹配所有本地图片链接  匹配形式为：![...](...)
    regex = r"!\[\S+\]\((\S+)\)"
    matches = re.findall(regex, content)

    # 替换本地图片链接为新链接for match in matches:# 判断链接是否是本地路径if not match.startswith('http'):# 获取图片名称和路径if os.path.isabs(match):
                image_name = os.path.basename(match)
                image_path = match
            else:
                image_name = os.path.basename(match)
                image_path = os.path.abspath(os.path.join(os.path.dirname(file_path), match))
            sign_key = generate_sign_key_val('/' + image_name)
            # 上传图片到COS并计算新链接
            new_link = put_request(image_path, sign_key)
            # 替换Markdown文件中的链接
            content = content.replace(match, new_link)

    # 将替换后的Markdown内容写回文件with open(file_path, 'w', encoding='utf-8') as f:
        f.write(content)

    print('本地图片链接替换完成。')


if __name__ == '__main__':
    # 测试替换
    replace_local_image_links("/Users/xieshijie/Desktop/test.md")
```



## 升级plus

整个过程让我对ChatGPT的潜力有了更深的认识。起初，我花费了半天的时间，主要用于阅读腾讯云COS服务的文档以及分步骤生成签名的过程。然而，正确利用ChatGPT实际上可以将这个过程缩短到一个小时，甚至更少。

ChatGPT不仅能高效地生成代码，还能在一些复杂的任务中表现出强大的理解和推理能力。这次体验让我更加坚定地认为，ChatGPT正在以一种前所未有的方式颠覆传统的编程和开发流程，真正实现了十倍的生产力提升。

于是，我果断充了一个GPT-4

![image-20240623141133563](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/image-20240623141133563_50c209126550805c85cc1c164e304875.png)





----


## 更多惊喜

我们的公众号还将定期分享：

- **最新互联网资讯**：让你时刻掌握行业动态。

- **AI前沿新闻**：紧跟技术潮流，不断提升自我。

- **技术分享与职业发展**：助你在职业生涯中走得更远、更稳。

- **程序员生活趣事**：让你在忙碌的工作之余找到共鸣与乐趣。

  

> **关注回复【1024】惊喜等你来拿！**

## 敬请关注【程序员世杰】

![coder_world_618](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/img/2024/coder_world_618-9122505_324d5d2e80e9ad51355871aa6ebda8f0_ba41e3bfe1d1ad09079739513911e407.jpg)

