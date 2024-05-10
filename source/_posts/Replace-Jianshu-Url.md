---
title: 更换博客图床
categories:
    - 环境搭建
tags:
    - blog
    - python
    - jsdelivr
date: 2020-01-10 12:50:24
---

<Excerpt in index | 摘要>
我最开始写博客的时候接触到的就是简书，简书的在线编辑文章对于markdown的支持在2年前就很好用了，而且上传图片也比较简单，直接拖到文章里就能生成链接了，也省了一个图床。于是我就先在简书写markdown，再把文章复制一份放到[自己的网站](https://longxuan.ren/)上。平时比较多的情况是直接在文章里贴代码，图可能就在最后放个结果图的链接，最近去看自己网站上的博客的时候，发现图片全都看不了，找了下原因发现简书上传的图片在外站貌似是无法查看了，把简书当图床用的日子到头了_(:з)∠)_
<!-- more --><The rest of contents | 余下全文>

## 寻找合适的图床
有了简书的这个经历，发现通过某个网站当图床都有点不靠谱了，指不定哪天就禁止外链，所有图片就都gg了，所以图床还是自己搭一个比较好。
搜索引擎找了一圈，有三个比较常用的：
1. ~~[七牛云](https://www.qiniu.com/)~~
2. ~~[阿里云](https://www.aliyun.com/)~~
3. [Github](https://github.com/)

### 七牛云
七牛云有免费空间，不过必须要你的域名是备案过的才行，我本来域名是备案了的，备案了俩，上个月阿里客服给我打电话说我不用的另一个域名需要注销，不然会影响主体域名，我手滑把所有都注销了Orz...于是七牛这边就没法用了，除非我重新再备案一波。顺便翻了下别人用七牛云当图床的情况，https的图片外链好像不算免费的，不过我也懒得考证了。

![域名不备案空间会被冻结](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Replace-Jianshu-Url-2020-1-10-11-3-49.png)

### 阿里云
阿里云的OSS看着感觉挺便宜的，那个标准存储40G一年9块钱的样子，好像这是存储费用，如果发生了外网访问产生了流量要另外收费。不过钱不是重点，找了一下使用OSS是否要备案，客服问了也不吱声，直接叫我发工单emmmmm，我可能碰到了假客服。最后在一个叫[阿鲤云百科](https://www.aliyunbaike.com/oss/3198/)的地方翻到这个说明：
![香港的OSS也要备案](https://upload-images.jianshu.io/upload_images/2756183-2a7cd8c888499d1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

香港的OSS都要备案的话，其他的应该也不用问了。查到这我想是不是国内的这些平台都要备案呢，要不还是备案一下？搜索了一下阿里云备案的相关说明，至少要开一个ECS服务器3个月以上。阿里云对老用户真的是一点优惠都木有，没个几百块是搞不定了，关键我还用不着，网站现在放在Github page好好的，这个方案也放弃。
![ECS需要购买3个月以上](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Replace-Jianshu-Url-2020-1-10-11-29-31.png)

### Github 
这个方案放到最后是由于Github仓库平时不用代理clone都费劲，考虑到图片外链希望加载速度快些。不过解决方案却意外的简单，搜索使用Github当图床的时候发现了这个免费的CDN加速[jsdelivr](https://www.jsdelivr.com/)，可以直接使用上传到Github仓库的文件做外链，只需要按要求改下图片链接即可。
![jsdelivr介绍](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Replace-Jianshu-Url-2020-1-10-12-11-31.png)

这是官网说明的链接地址规则：
![转换地址规则](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Replace-Jianshu-Url-2020-1-10-12-16-45.png)

下面是我自己的仓库链接转换前和转换后的链接：
```
//图片上床到Github仓库的地址
https://github.com/Longxr/PicStored/tree/master/blog/xxx.png

//jsdelivr链接地址
https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/xxx.png
```
## 现有文章链接转换
现有的简书的链接有100多个，一个一个替换头都大了，还是用python大法好。在网上找了下[用Python替换简书链接的内容](https://juejin.im/post/5cf86bc9f265da1bb003aebe)，发现挺合适的就改了下博主的脚本，修改内容如下：
1. 原文是直接将简书链接转换成了Github仓库的链接，我是转换成jsdelivr链接。
2. 原文是通过脚本直接上传到Github，我去掉了这部分，直接复制到本地仓库，方便检查是否有问题。
3. 原文是使用图片的uuid作为图片名称，我是改成了当前md的文件名+index的命名方式。

这是做了改动的脚本，如果需要原版可以点开上面的链接查看：
```
import imghdr
import os
import re
import shutil

import requests

## 用户名
user_name = 'Longxr';
## 仓库名
github_repository = 'PicStored';
## 存放图片的git文件夹路径
git_images_folder = 'F:/workspace/GithubCode/PicStored/blog'

# 设置用户代理头
headers = {
    'User-Agent':'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.62 Safari/537.36'
}

# 获取当前目录下所有md文件
def get_md_files(md_dir):
    md_files = [];
    for root, dirs, files in sorted(os.walk(md_dir)):
        for file in files:
            # 获取.md结尾的文件
            if(file.endswith('.md')):
                file_path = os.path.join(root, file)
                print(file_path)
                #忽略排除目录
                need_append = 0
                for ignore_dir in ignore_dir_list:
                    if(ignore_dir in file_path.split('/') == True):
                        need_append = 1
                if(need_append == 0):
                    md_files.append(file_path)
    return md_files

# 获取网络图片
def get_http_image(image_url, file_new_name):
    image_info = {'image_url': '', 'new_image_url': ''}
    image_data = requests.get(image_url.split('?')[0], headers=headers).content
    # 创建临时文件
    tmp_new_image_path_and_name = os.path.join(git_images_folder, file_new_name)
    with open(tmp_new_image_path_and_name, 'wb+') as f:
        f.write(image_data)
    img_type = imghdr.what(tmp_new_image_path_and_name)
    if(img_type == None):
        img_type = ''
    else:
        img_type = '.'+img_type
    # 生成新的名字加后缀
    new_image_path_and_name = tmp_new_image_path_and_name+img_type
    # 重命名图片
    os.rename(tmp_new_image_path_and_name, new_image_path_and_name)
    new_image_url = 'https://cdn.jsdelivr.net/gh/'+ user_name + '/' +github_repository+'/blog/'+ file_new_name + img_type
    image_info = {
        'image_url': image_url,
        'new_image_url': new_image_url
    }
    print('image_info', image_info)

    return image_info


# 获取本地图片
def get_local_image(image_url, file_new_name):
    image_info = {'image_url': '', 'new_image_url': ''}
    try:
        # 获取图片类型
        img_type = image_url.split('.')[-1]
        # 新的图片名和文件后缀
        image_name = file_new_name +'.'+ img_type
        # 新的图片路径和名字
        new_image_path_and_name = os.path.join(git_images_folder, image_name);
        shutil.copy(image_url, new_image_path_and_name)
        # 生成url
        new_image_url = 'https://cdn.jsdelivr.net/gh/'+ user_name + '/' +github_repository+'/blog/'+ file_new_name + img_type
        # 图片信息
        image_info = {
            'image_url': image_url,
            'new_image_url': new_image_url
        }
        print(image_info)
        return image_info
    except Exception as e:
        print(e)

    return image_info
    
# 爬取单个md文件内的图片
def get_images_from_md_file(md_file):
    md_content = ''
    image_info_list = []
    md_file_name = os.path.splitext(os.path.basename(md_file))[0]
    url_index = 0

    with open(md_file, 'r', encoding='UTF-8') as f:
        md_content = f.read()
        image_urls = re.findall(r'!\[.*?\]\((.*?)\)', md_content)
        for image_url in image_urls:
            url_index += 1
            image_new_name_without_type = md_file_name + '_%02d'%url_index

            # 处理本地图片
            if(image_url.startswith('http') == False):
                image_info = get_local_image(image_url, image_new_name_without_type)
                image_info_list.append(image_info)
            # 处理网络图片
            else:
                # 只爬取简书的
                if(image_url.startswith('https://upload-images.jianshu.io')):
                    try:
                        image_info = get_http_image(image_url, image_new_name_without_type)
                        image_info_list.append(image_info)
                    except Exception as e:
                        print(image_url, 'Unable to crawl, skip!')
                        pass
        for image_info in image_info_list:
            md_content = md_content.replace(image_info['image_url'], image_info['new_image_url'])

    with open(md_file, 'w+', encoding='UTF-8') as f:
        f.write(str(md_content.encode('utf-8').decode('utf-8')))

if __name__ == '__main__':
    if(os.path.exists(git_images_folder)):
        pass
    else:
        os.mkdir(git_images_folder)
    # 获取本目录下所有md文件
    md_files = get_md_files(os.path.dirname(os.path.abspath(__file__)))

    # 将md文件依次爬取
    for md_file in md_files:
      # 爬取单个md文件内的图片
      get_images_from_md_file(md_file)

```

转换完后查看没什么问题，就可以提交到Github了，妥了O(∩_∩)O
![转化图片结果](https://cdn.jsdelivr.net/gh/Longxr/PicStored/blog/Replace-Jianshu-Url-2020-1-10-12-34-57.png)

## 相关链接
* [阿里云备案相关说明](https://yq.aliyun.com/articles/720839?spm=a2c4e.11153940.0.0.4ea56b6c1GxVHQ&accounttraceid=e62a502a514e41ed98ee18c3274fa7c9tajr)
* [应对掘金CDN开启防盗链 记一次爬取markdown图片的经历](https://juejin.im/post/5cf86bc9f265da1bb003aebe)
* [Github+jsDelivr+PicGo 打造稳定快速、高效免费图床](https://www.itrhx.com/2019/08/01/A27-image-hosting/)