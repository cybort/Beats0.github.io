---
layout: post
title:  "用steam做图床"
categories: javascript
tags: 图床 跨域请求 tool
author: Beats0
mathjax: true
---

* content
{:toc}

<b>为什么要使用steam云做图床呢?
 - 国内的渣浪微博图床和七牛云都会对用户的图片进行检测限制，有些图片就会被和谐掉
 - steam云有18G，图片基于[Akamai](https://baike.baidu.com/item/Akamai/10008179?fr=aladdin)的,这也意味着图片具有全球CDN加速服务
 - 图片可以在个人资料的`Screenshot`选项中任意更改，便于管理








上传工具有两个，一个是[SteaScree](https://github.com/Foyl/SteaScree)，另一个[SteamScreenshotUploaderGUI](https://github.com/0x6FA3D0/SteamScreenshotUploaderGUI)，两者相比较，我更喜欢SteaScree，应为这个工具可以批量输出，缺点就是可能会不稳定，SteamScreenshotUploaderGUI只能上传单个，但稳定性好

首先用工具输出文件路径，然后再用Steam自带的Steam Screenshots Uploader批量上传图片，最后使用JavaScript跨域请求获得图片链接，注意：图片下载方面，只能使用Chrome浏览器，原因是因为只有Chrome浏览器可以实施[FileSystemAPI](https://developer.mozilla.org/zh-CN/docs/WebGuide/API/File_System/Introduction#%E9%99%90%E5%88%B6)。目前尚不存在专门用于文件/配额管理的浏览器用户界面。实测Firefox只能打印图片链接地址，但不能下载图片

#### SteaScree

[SteaScree](https://github.com/Foyl/SteaScree)

路径: http://www.softpedia.com/get/Internet/Other-Internet-Related/SteaScree.shtml

[steamgroup](https://steamcommunity.com/groups/steascree)

##### 使用

选择steam默认安装路径，选择gameID(可以手动输入ID，前提是你确实是有这款游戏)，

点击`Add screenshots to queue...` 添加图片

点击 `Copy screenshots to game directory` 将图片输出到对应的游戏截图文件夹下

注意这里要退出steam客户端，点击 `Prepare screenshots for uploading...` 生成游戏截图的vtf日志

重新打开steam，选择游戏的截图库，然后使用steam的官方截图上传器上传即可，注意选择图片的私密性

#### SteamScreenshotUploaderGUI

[SteamScreenshotUploaderGUI](https://github.com/0x6FA3D0/SteamScreenshotUploaderGUI)

[SteamGroup](https://steamcommunity.com/sharedfiles/filedetails/?id=878337526)

路径1：https://drive.google.com/file/d/0BwSFV9LmCqmiVjRGTWluOXdIbmc/view

路径2：百度云[http://pan.baidu.com/s/1hrYCQRq](http://pan.baidu.com/s/1hrYCQRq)

##### 使用

English: https://steamcommunity.com/sharedfiles/filedetails/?id=878337526

中文: https://steamcommunity.com/sharedfiles/filedetails/?l=spanish&id=891916460

上传过程和SteamGroup的上传过程差不多

#### 获取图片地址

在浏览器里打开自己的steam个人资料选择Screenshot截图页面(建议以网格视图查看)

输入以下JavaScript，这个函数回调图片链接地址，可以将链接地址打印或转为图片下载(只有chrome能下载图片，下载图片默认关闭)
```js
downloadURI = function (uri, name) {
    var link = document.createElement("a");
    link.download = name;
    link.href = uri;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
    delete link;
};

makeRequest = function (method, url) {
    return new Promise(function (resolve, reject) {
        var xhr = new XMLHttpRequest();
        xhr.open(method, url);
        xhr.onload = function () {
            if (this.status >= 200 && this.status < 300) {
                resolve(xhr.responseText);
            } else {
                reject({
                    status: this.status,
                    statusText: xhr.statusText
                });
            }
        };
        xhr.onerror = function () {
            reject({
                status: this.status,
                statusText: xhr.statusText
            });
        };
        xhr.send();
    });
};


var steamImageUrls = [];

getUrlFromResponse = function (responseText) {
    window.resp = responseText;
    var linkRegex = new RegExp('https://steamuserimages[^\"]+');
    var url = linkRegex.exec(window.resp)[0];
    steamImageUrls.push(url);
    //正则过滤，批量打印原图链接地址
    console.log(/^.*?(?=(?:\?|$))/gi.exec(url)[0]);
    return url;
};

var urls = [];
jQuery('a.profile_media_item.modalContentLink').each(function (index, el) {
    urls.push(el.href);
});

for (var i = 0; i < urls.length; ++i) {
    makeRequest("GET", urls[i])
        .then(function (text) {
            var imageUrl = getUrlFromResponse(text);
            //是否下载当前图片，默认不开启
            // downloadURI(imageUrl, 'image' + i + '.jpg');
        });
}
```

## 参考

 - [SteaScree](https://github.com/Foyl/SteaScree)
 - [0x6FA3D0/SteamScreenshotUploaderGUI Source Code](https://github.com/0x6FA3D0/SteamScreenshotUploaderGUI)
 - [Uploading Custom Screenshots Made Easy](https://steamcommunity.com/sharedfiles/filedetails/?id=878337526)
 - [verysmallrock/steamscreenshotdownloader](https://github.com/verysmallrock/steamscreenshotdownloader)
