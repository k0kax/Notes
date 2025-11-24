
## 一、同步Github
1.创建Github仓库
![](https://raw.githubusercontent.com/k0kax/PicGo/main/images20251124195215654.png)
2.clone到本地
3.用Obsbian打开该文件夹作为仓库
![](https://raw.githubusercontent.com/k0kax/PicGo/main/images20251124195349307.png)
4.创建.gitngore文件，记录不用上传的文件
```
.obsidian/workspace.json
.obsidian/workspace-mobile.json
```

5.Obsbian安装插件Git
![](https://raw.githubusercontent.com/k0kax/PicGo/main/images20251124195641861.png)
6.配置插件
停止操作一分钟，自动上传
![[Pasted image 20251124195747.png]]
注意事项：
Github有时候会连不上，导致自动上传出现问题，可以用加速器

## 二、附件存储
在设置里找到附件默认存放位置，按需求修改即可
![[Pasted image 20251124195945.png]]
我这里是存在该文件的资格子目录下，普通文件还好，图片的话，最好用图床，markdown引用更加方便
### 三、图床设置
本地图床设置，采用PicGo+Github仓库
1.下载[PicGo](https://github.com/Molunerfinn/PicGo/releases)
2.在Github新建一个开放的仓库
3.配置PicGo
获取token 去GitHub个人页面setting/Developer_Setting/Person Access Token申请一个token
![](https://raw.githubusercontent.com/k0kax/PicGo/main/images20251124200426584.png)

![[Pasted image 20251124200556.png]]
![[Pasted image 20251124200727.png]]
选择qu'xian'qi


参考
   [图片存储](https://www.bilibili.com/video/BV1TCk6BeEJz/?spm_id_from=333.337.search-card.all.click&vd_source=276894b0a5d2380fcbcf1442c30e3620)
   [GitHub同步](https://www.bilibili.com/video/BV1HY5EzCEk5/?spm_id_from=333.337.search-card.all.click&vd_source=276894b0a5d2380fcbcf1442c30e3620)
   [综合信息](https://www.bilibili.com/video/BV1fZCyBYEuT/?spm_id_from=333.337.search-card.all.click&vd_source=276894b0a5d2380fcbcf1442c30e3620)