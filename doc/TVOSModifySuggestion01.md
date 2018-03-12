<p align="right">**2017年8月15日下午**</p>

## 0. 补充

获取到的数据

首先是记录，形式如下

```json
{
    "id": "TVOS%2FTVOS2%2Fframework%2Fngb-j~TVOS_DEV~I4955b826d032aac46d8f3f5012ee2719317e636e",
    "project": "TVOS/TVOS2/framework/ngb-j",
    "branch": "TVOS_DEV",
    "hashtags": [],
    "change_id": "I4955b826d032aac46d8f3f5012ee2719317e636e",
    "subject": "Headline/CR/PR:  [DRM project] CR Description: Creat DRM Code and Refine DRM case. UnitTest: Pass",
    "status": "NEW",
    "created": "2017-07-11 02:19:35.000000000",
    "updated": "2017-07-11 02:19:35.000000000",
    "mergeable": true,
    "insertions": 897,
    "deletions": 1,
    "_number": 1590,
    "owner": {
      "_account_id": 107,
      "name": "jingang dong",
      "email": "dongjg@novel-supertv.com",
      "username": "dongjingang"
    },
    "labels": {
      "Code-Review": {},
      "Verified": {}
    }
```

这样的记录有六百多条（分布在不同的版块，open,merged,abandoned），每条记录都有一个更详细的json文件，比如上面这个对应如下：

```json
{
  "/COMMIT_MSG": {
    "status": "A",
    "lines_inserted": 11
  },
  "Android.mk": {
    "lines_inserted": 8
  },
  "java/org/ngb/drm/services/ChinaDrmManager.java": {
    "status": "A",
    "lines_inserted": 301
  },
  "java/org/ngb/drm/services/ChinaDrmTeeRetVal.java": {
    "status": "A",
    "lines_inserted": 56
  },
  "jni/Android.mk": {
    "lines_inserted": 16
  },
  "jni/onload.cpp": {
    "lines_inserted": 6,
    "lines_deleted": 1
  },
  "jni/org_ngb_drm_services_ChinaDrmManager.cpp": {
    "status": "A",
    "lines_inserted": 510
  }
}
```

代表着对应项目的对应文件修改的信息



##1.前端部分

###1.1. 整体界面设计

整个界面分为：导航条，主页，侧页

【如果想改得话，这里建议参看docs.docker.com的界面设计】

###1.2.主页（右边显示部分）

针对的是All ，Open，Merged，Abandoned的显示，分为4块,分别如下

【尽量用汉字描述清楚，尤其是每个图代表的含义】

####1.2.1.项目

饼图 鼠标移动显示内容需详细说明（项目名，修改数，占比）【汉字，三行】

饼图下部分显示占比排名前5的项目信息：【这里建议是使用表格】

####1.2.2. 文件

时间-修改图

文件修改次数排名（表格）显示：文件路径，修改量

####1.2.3.用户：

统计用户提交次数  柱状图

提交最多的前5个  表格

####1.2.4.公司：

统计公司提交次数  柱状图【邮箱@后】

提交最多的前5个  表格



###1.3.侧页

侧页展示方式参看docs.docker.com，**展示要新颖**【这里本来我说用北邮人论坛左侧的文件展开形式，leader说太low了，要新颖】

侧页需要展示下面三部分，点击对应的相应条目，主页显示见下面具体介绍

####1.3.1.项目

侧栏显示的是项目名取最后一个/后面的，点击后，在主页上方显示全称

主页部分显示下面几块（对应1.2部分设计）：

branch 饼图

文件

用户

公司

####1.3.2.用户

主页上方显示用户属性

下方显示下面几块（对应1.2部分设计）：

文件

项目

####1.3.3.公司

主页上方显示公司属性

下方显示下面几块（对应1.2部分设计）：

文件

项目

用户

##2.后台部分

1. document部分需要添加后台各部分的结构图


2. Docker的使用

