---
title: 从零开始搭建一个酷炫的个人博客
excerpt: 从零开始搭建一个酷炫的个人博客，包括git、hexo、fluid主题使用、购买域名、发布文章
updated: 2022-12-27 19:11:09
category_bar: true
date: 2022-11-18 19:48:50

tags:
- demo
- 博客

categories:
- [其他技能]

---

## 一、搭建网站

### git和hexo准备

1. [注册GitHub](https://zhuanlan.zhihu.com/p/103268406)
2. [本地安装Git](https://zhuanlan.zhihu.com/p/103325381)
3. [绑定GitHub并提交文件](https://zhuanlan.zhihu.com/p/103391101)
4. [安装npm和hexo，并绑定github上的仓库](https://zhuanlan.zhihu.com/p/105715224)
5. 注意：上述教程都是Windows系统，Mac系统会更简单！



### 域名准备

1. 购买域名，买的是[腾讯云域名](https://buy.cloud.tencent.com/domain?from=console)，购买完成之后的[域名管理](https://console.cloud.tencent.com/domain/all-domain)
2. [解析域名](https://zhuanlan.zhihu.com/p/103813944)
3. [域名备案](https://console.cloud.tencent.com/beian)

## 二、优化网站

1. 使用的Fluid主题，[Hexo Fluid 用户手册](https://fluid-dev.github.io/hexo-fluid-docs/guide/#%E5%85%B3%E4%BA%8E%E6%8C%87%E5%8D%97)

2. 增加图床，图片可以放在git中一起上传，但是图片多了会拖慢网站打开速度，推荐使用外链图床
   - 采用的是腾讯云的对象存储，直接购买资源包，然后上传图片即可，价格：10元/1年
   - [购买界面](https://cloud.tencent.com/product/cos)
   - [使用界面](https://console.cloud.tencent.com/cos)

3. 增加评论

4. 增加页面统计

5. 变更图标和界面图片

   - 把想要显示的图片放到：`themes/fluid/source/img`文件夹下

   - 在`themes/fluid/_config.yml`配置文件中通过`img/xx.png`来定位图片

```yaml
    # 用于浏览器标签的图标
    # Icon for browser tab
    favicon: /img/icon
```

6. GitHub的网站增加README.md

   - 在根目录 source 文件夹下新建README.md

   - 在根目录的 _config.yml 配置文件里，找到 skip_render 关键字，添加 README.md

```yaml
      # 解释器跳过md渲染为html的过程
      skip_render: README.md
```


## 三、发布第一篇文章



[官方文档](https://hexo.io/zh-cn/docs/writing)



### 新建一篇文章

1. `hexo new post "第一篇文章"`
2. 在博客目录下的/source/\_posts/ 文件夹下，可以看到已经生成了标题为`第一篇文章.md`的博客文件，可以在\_posts/文件夹下创建子目录，用以更好的管理文章
   ![目录结构](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/%E5%85%B6%E4%BB%96%E6%8A%80%E8%83%BD/%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA%E6%94%BB%E7%95%A5/%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA-1.png)
3. Hexo 发布的文章是 Markdown 格式的文件



### 新建一篇草稿

1. `hexo new draft "第一篇草稿"`
2. 在博客目录下的/source/\_drafts/ 文件夹下，可以看到已经生成了标题为`第一篇文草稿.md`的草稿文件，
3. 草稿文章不会被其他人看到，直到通过`hexo publish draft "第一篇草稿"`才会把草稿推送为正式文章，从而被观测到



### 给文章添加分类和标签

```markdown
---
title: 个人博客搭建完整攻略
date: 2022-12-27 14:47:08
tags:
- 博客
- hexo             // 多个标签
  categories:
- [其他技能, 博客搭建]   // 多层级分类，中间用英文逗号分隔
---
```

![添加的分类结构](https://coder-xieshijie-img-1253784930.cos.ap-beijing.myqcloud.com/%E5%85%B6%E4%BB%96%E6%8A%80%E8%83%BD/%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA%E6%94%BB%E7%95%A5/%E4%B8%AA%E4%BA%BA%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA-2.png)



### 设置文章摘要

1. 关闭自动摘要，在主题配置文件/themes/fluid/_config.yml中找到auto_excerpt，设置enable=false

   ```yaml
   index:
     auto_excerpt:
       enable: true
   ```

2. 在文章中指定摘要

   ```markdown
   ---
   title: 这是标题
   excerpt: 这是摘要
   ---
   ```



### 设置文章模板

1. scaffolds/post.md 设置正式文章模板
2. scaffolds/draft.md 设置草稿模板

```markdown
---
title: {{ title }}
date: {{ date }}
tags:
-
categories:
- []
  excerpt: {{ title }}
---
```





### 启动博客服务器

1. 启动并本地测试：`hexo server `
2. 发布到Github上`hexo clean && hexo g && hexo d`

