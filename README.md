# TVOS
使用Django搭建的数据可视化项目







[TOC]

<p align="right">**项目负责人：**</p>

<p align="right">**项目开发人员：**</p>

<p align="right">**项目持续周期：**</p>

<p align="right">**版本：v1.2;  时间：17-08-28**</p>

# TVOS项目概要设计说明书

## 1 项目简介

### 1.1目的

分析http://120.25.200:8081上的数据，并将信息可视化展现在搭建的网址上

超级用户，可以直接修改数据库信息

​    访问网址：http://182.61.136/admin/ 

​    用户：tvos

​    密码：



### 1.2 实现

Django+MySQL+ECharts+Vue

运行环境：

1. Django1.11+
2. MySQL5.6+
3. Python3.4+

### 1.3 结构图

#### 1.3.1 系统框架图

![TVOSsys01](https://github.com/moluchase/TVOS/raw/master/img/tvos_sys.png)

#### 1.3.2 具体request-response图

![tvosseq](https://github.com/moluchase/TVOS/raw/master/img/tvos_seq.png)

### 1.4 代码组成

#### 1.4.1 tovs1

tvos1模块主要是由get_data.py, storage_data.py, operate_mysql.py, update_data.py这几个文件组成

其中update_data.py作为入口文件，get_data.py来获取(更新）json文件，storage_data.py来规范化数据，operate_mysql.py来存储数据

#### 1.4.2 tvossite

1. staticfile ：用来存储静态文件
2. tvos：
   1. admin.py：修改超级用户显示界面即执行动作
   2. models.py: 用来声明数据库模型
   3. views.py: 处理后台逻辑
3. tvossite：
   1. settings.py: 整个后台设置文件
   2. urls.py: 网址 -函数 映射文件
   3. wsgi.py: 后台入口文件


## 2 概要设计

### 2.1数据获取

#### 2.1.1各版块记录获取

首先访问网址，使用F12，动态抓包（Network->Headers），获取到如下信息

对于open，Merged，Abandoned，显示界面的json文件如下：

- Open       http://120.25.209:8081/changes/?n=25&O=81
- Merged     http://120.25.209:8081/changes/?q=status:merged&n=25&O=81
- Abandoned  http://120.25.209:8081/changes/?q=status:abandoned&n=25&O=81

#### 2.1.2 记录中详细信息的获取

点击网页上面版块任一条目，进入详细信息，其json文件网址为

http://120.25.209:8081/changes/1637/revisions/205f3f9694e931de9779cdbaa82c5bb881751899/files

分析上面网址，有两个地方是变化的：

- 一个是1637，一个是205f3f9694e931de9779cdbaa82c5bb881751899

​       1637代表的是该条目的编号，在对应版块的json文件中可获取(_number)

- 第二个为commit的编号，通过查看Network中显示的文件，发现在下面的json文件中可获取

​        http://120.25.209:8081/changes/1637/detail?O=404

#### 2.1.3 代码实现

见**附录5.1数据获取**

总共下载了1500多个json文件。

### 2.2 数据分析

####2.2.1 all,open,merged,abandoned版块显示部分

1. 项目-修改条数（饼图）：需要的数据是项目名，以及各项目的修改次数
2. 时间-修改量（折线图）：需要的数据是时间，及其对应的代码增加量，代码删除量
3. 用户-修改条数（条形图）：需要的数据是用户名，及其修改的次数
4. 公司-修改条数（条形图）：需要的数据是公司名，及其修改的次数



![1](https://github.com/moluchase/TVOS/raw/master/img/01.png)
![2](https://github.com/moluchase/TVOS/raw/master/img/02.png)
![3](https://github.com/moluchase/TVOS/raw/master/img/03.png)


#### 2.2.2 侧页显示部分

显示项目名，用户名，公司名

#### 2.2.3 侧页点击事件显示部分

项目条目

1. branch-修改条数（饼图）
2. 时间-修改量（折线图）
3. 用户-修改条数（条形图）
4. 公司-修改条数（条形图）

用户条目

1. 时间-修改条数（折线图）
2. 项目-修改条数（条形图）
3. 公司-修改条（条形图？）

公司条目

1. 时间-修改量（折线图）
2. 项目-修改条数（条形图）
3. 用户-修改条数（条形图）

### 2.3 数据存储

根据上面的数据分析设计数据表结构   

#### 2.3.1 数据库表格设计

**tvos_record**

| 字段名称       | 字段描述 | 字段类型        | 约束          |
| ---------- | ---- | ----------- | ----------- |
| number     | 标号   | int         | PRIMARY KEY |
| project    | 项目名  | varchar(50) | NOT NULL    |
| branch     | 分支   | varchar(20) | NOT NULL    |
| updated    | 更新日期 | date        | NOT NULL    |
| insertions | 增量   | int         | NOT NULL    |
| deletions  | 删量   | int         | NOT NULL    |
| owner      | 修改者  | varchar(16) | NOT NULL    |
| section    | 版块   | varchar(10) | NOT NULL    |
| company    | 公司   | varchar(16) | NOTNULL     |

**tvos_info**

| 字段名称       | 字段描述    | 字段类型         | 备注          |
| ---------- | ------- | ------------ | ----------- |
| filepath   | 修改的文件路径 | varchar(250) | PRIMARY KEY |
| project    | 所属项目名   | varchar(50)  | PRIMARY KEY |
| insertions | 总增量     | int          | NOT NULL    |
| deletions  | 总删量     | int          | NOT NULL    |
| num        | 修改次数    | int          | NOT NULL    |

【这里Django中没有联合主键，会默认新建字段id，约束条件是project 和filepath字段唯一】

#### 2.3.2 存储数据

由上在models.py中设置数据表，如下所示：

```python
class Record(models.Model):
    number=models.IntegerField(primary_key=True)
    project=models.CharField(max_length=50)
    branch=models.CharField(max_length=20)
    updated=models.DateField()
    insertions=models.IntegerField()
    deletions=models.IntegerField()
    owner=models.CharField(max_length=16)
    company=models.CharField(max_length=16)
    section = models.CharField(max_length=10)
    def __str__(self):
        return self.number


class Info(models.Model):
    filepath=models.CharField(max_length=250)
    project=models.CharField(max_length=50)
    insertions=models.IntegerField()
    deletions=models.IntegerField()
    num=models.IntegerField()

    class Meta:
        unique_together = ('filepath', 'project')
```

在settings中导入pymysql

```python
import pymysql
pymysql.install_as_MySQLdb()
```

修改数据库信息（此处需要安装好MySQL环境）：

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'tvosdb',
        'USER': 'root',
        'PASSWORD':'123645',
        'HOST':'localhost',
        'PORT': '',
    }
}
```

终端进入manage.py目录执行下面命令

```
python3 manage.py makemigrations
python3 manage.py migrate
```

会发现Mysql的tvosdb数据库中生成了好几个表，tovs目录下多了一个migrations文件夹

插入数据，这里直接使用pymysql插入数据到MySQL中，具体代码见附录，由**tvos1**项目实现



### 2.4 Django后台

Django框架中一个项目可以有多个app，在app中完成前后端的交互，分为三个部分类似于MVC模式的MTV模式

#### 2.4.1 Model

负责业务对象和数据库的关系映射，作为views.py与数据库的接口

models.py在2.3.2部分已近给出

#### 2.4.2 Template

负责如何把页面展示给用户

这里对应的时templates目录下的html文件，前端html文件全放于此，对于前端中使用到的css,js等文件，需要再app（或者其他目录）下新建static目录，后台会去查找static目录，见**3.1.1**

**此项目并没有使用Django中的T部分，而是前后端分离**

详细见**前端部分**

#### 2.4.3 View

负责业务逻辑，并在适当时候调用Model和Template

这里对应的是views.py文件，在其中定义函数，获取数据，并传入前端界面

函数名和前端访问网址对应，在项目中的urls.py中使用正则匹配获取网址，并对应到view.py中的函数

此处可见附录部分示例

#### 2.4.4 admin

admin作为后台管理界面，提供给超级用户修改数据库的权限

这部分除了提供可视化的数据显示外，还添加了超级用户更新数据库的行为，具体代码见附录



### 2.5 服务器部署

#### 2.5.1 直接部署

1. 安装软件

- 在服务器上安装apache2和mod_wsgi
- 安装Django1.11
- 安装python3.4及以上
- 安装Mysql

2. 将TVOS项目放在服务器指定位置，(/var/www/)

3. 设置网址配置文件

   这里在/etc/apache2/sites-available/下新建conf文件，其中静态文件路径填写静态文件在服务器上的绝对路径

4. 生成静态文件

   这里要到 /var/www/tvossite/目录下执行下面语句(需要在setting.py中设置STATIC_ROOT)

   ```
   python3 manage.py collectstatic
   ```

5. 激活网址

   在3中新建conf文件(mysite.conf)目录下执行下面语句

   ```
   sudo a2ensite mysite.conf
   ```

6. 启动服务器

   ```
   service apache2 restart
   ```

【注】这里可能会出现静态文件丢失的问题，见3.1部分

####2.5.2 Nginx

1. 在服务器上的/home/tvos/djangoBack目录下：初始化目录

   ```
   git init
   ```

2. 将tvossite项目克隆到本地

   ```
   git clone http://47.94.228:10080/tvos/tvossite.git
   ```

3. 安装Django

   ```shell
   sudo apt-get install python3-pip
   sudo pip install Django==1.11
   ```

4. 安装mysql数据库

   ```shell
   sudo apt-get install mysql-server
   ```

   进入mysql的命令是：`mysql -u root -p`    密码为空

   修改密码为125【密码为空，下面pymysql会报错】

   ```shell
   ..>use mysql;
   ..>update user set authentication_string=PASSWORD("123645") where user='root';
   ..>update user set plugin="mysql_native_password"; 
   ..>flush privileges;
   ..>quit;
    ..>service mysql stop 
   ..>service mysql start
   ```

   创建数据库tvosdb

   ```
   create database tvosdb
   ```

   在tvossite的settings.py目录下修改数据库信息

5. 在tvossite下的tvos目录下，删除migrations文件夹

6. 安装python第三方包

   ```
   pip3 install pymysql
   pip3 install django-cors-headers
   ```

7. 在tvossite目录下执行下面语句：

   ```
   python3 manage.py makemigrations tvos
   python3 manage.py migrate
   ```

   到这一步后需要达到数据库中tvosdb有tvos_info等表，且执行`python3 manage.py runserver`正常

8. 导入数据到数据库中

   因为搭建Django项目，需要用到数据库中的数据，将tvos1项目放在`/home/tvos`目录下，然后在终端中运行update_data.py

9. 创建超级用户

   用户名:tvos

   密码：

10. 安装nginx，管理进程的工具

   ```
   apt-get install python3-dev nginx
   pip install supervisor  #可以不用安装，只是方便管理
   ```

   注意上面的pip，不是pip3，因为python3不支持supervisor，只能用python2

11. 部署uwsgi

    ```
    pip3 install uwsgi --upgrade
    ```

    此处执行下面语句正常，表示部署正确，执行后会显示uwsgi路径，后面会用到

    ```
    uwsgi --http :10088 --chdir /home/tvos/djangoBack/tvossite --module tvossite.wsgi
    ```

    current working directory: /home/tvos/djangoBack/tvossite/tvossite

    detected binary path: /usr/local/bin/uwsgi

12. 使用supervisor来管理进程

    这步我没有做，并没有使用此来管理进程

13. 配置Nginx

    ```
    vi /etc/nginx/sites-available/tvossite.conf
    ```

    内容为：

    ```
    server {
        listen      10088;
        server_name 47.94.222.108;
        charset     utf-8;
     
        client_max_body_size 75M;
     
        location /static {
            alias /home/tvos/djangoBack/tvossite/staticfile;
        }
     
        location / {
            uwsgi_pass  unix:/home/tvos/djangoBack/tvossite/tvossite.sock;
            include     /etc/nginx/uwsgi_params;
        }
    }
    ```

    其中静态文件路径

    这个中的sock只是一个路径而已，会自动生成，与uswgi文件中的socket匹配

14. 配置ini文件：

    ```
    [uwsgi]
    socket = /home/tvos/djangoBack/tvossite/tvossite.sock
    chdir = /home/tvos/djangoBack/tvossite
    module = tvossite.wsgi
    master = true

    processes = 2
    threads = 4

    chmod-socket = 666
    #chown-socket = root:www-data

    vacuum = true
    ```

    这个文件对接的是nginx，通过socket对接，然后访问chdir处的Django项目下的wsgi

15. 激活网站：

    建立软连接

    ```
    ln -s /etc/nginx/sites-available/tvossite.conf /etc/nginx/sites-enabled/tvossite.conf
    ```

16. 启动nginx和uwsgi

    ```
    /etc/init.d/nginx restart
    uwsgi --ini mysite_uwsgi.ini
    ```

    ​

17. 踩的坑：

    1. uwsgi运行报错：!!! no internal routing support, rebuild with pcre support !!!

       需要安装nginx的依赖库：zlib，pcre，openssl：

       ```
       sudo apt-get install openssl libssl-dev 
       sudo apt-get install libpcre3 libpcre3-dev
       sudo apt-get install zlib1g-dev 
       ```

       然后卸载uwsgi

       ```
       pip3 uninstall uwsgi
       sudo apt-get remove uwsgi
       #安装uwsgi：
       pip3 install uwsgi -I --no-cache-dir
       ```

    2. 如果全部下来没有错，但是公网访问不了，应该做如下两步：

    首先判断直接搭nginx，公网是否能访问：

    ```
    sudo apt-get install nginx
    /etc/init.d/nginx start
    ```

    在`/etc/nginx/sites-avaiable/default`中修改端口号，然后通过http://ip:端口号  访问，如果能够访问，说明nginx搭建没有问题,执行第二步

    然后关闭nginx，安装uwsgi

    ```
    pip3 install uwsgi
    ```

    在当前目录下新建test.py文件

    ```
    def application(env, start_response):
           start_response('200 OK', [('Content-Type','text/html')])
           #return ["Hello World"] # python2
           return [b"Hello World"] # python3
    ```

    然后执行如下命令：

    ```
    uwsgi --http 0.0.0.0:8888 --wsgi-file test.py
    ```

    通过http://ip:端口号  访问，如果能够访问，说明uwsgi搭建没有问题，那么问题只可能是配置文件的问题了

     如果第一步就不能访问，原因可能是防火墙设置`ufw disable`，或者端口号被禁止

18. 修改权限：


```
chown user file -R  #表示将file目录文件的所有者改为user，-R表示包括子目录，去掉则不包括
chgrp group file -R #表示将文件所属群组改为group，-R表示包括子目录
```


配置参考：

http://code.ziqiangxuetang.com/django/django-nginx-deploy.html

http://blog.csdn.net/shiyuezhong/article/details/39396487

https://stackoverflow.com/questions/21669354/rebuild-uwsgi-with-pcre-support

http://jukezhang.com/2014/11/28/Nginx-uWSGI-Django-MySQL/

#### 2.5.3 Docker

【待】

### 2.6 数据请求部分
#### 2.6.1主页

点击all,abandoned,merged版块所展现的数据

all：http://182.61.16/section/?section=all

abandoned：http://182.61.56/section/?section=abandoned

merged：http://182.61.16/section/?section=merged

参数说明

| 参数名         | 含义                 |
| ----------- | ------------------ |
| dateList    | 日期列表               |
| insertList  | 对应于日期列表的增加列表       |
| deleteList  | 对应于日期列表的删除列表       |
| ownerName   | 用户名列表              |
| ownerNum    | 对应于用户名列表的各用户修改条数列表 |
| companyName | 公司名列表              |
| companyNum  | 对应于公司名列表的各公司修改条数列表 |
| projectName | 项目名列表              |
| projectNum  | 对应于项目名列表的各项目修改条数列表 |

#### 2.6.2侧栏

侧栏显示数据

http://182.6156/nav/  （这个中的信息实际上all版块是可以获取的）

参数说明

| 参数名         | 参数说明  |
| ----------- | ----- |
| projectName | 项目名列表 |
| ownerName   | 用户名列表 |
| companyName | 公司名列表 |

####2.6. 3 侧栏相应的版块

下面的网址是举例，对应的只需要将后面接得参数修改为项目名，用户名，公司名即可

项目：http://182.61.6/navsec/project/?project=TVOS/TVOS2/component/weblink

用户：http://182.61.6/navsec/owner/?owner=jiamin%20wang

公司：http://182.61.6/navsec/company/?company=changhong

项目参数说明（针对项目）

| 参数名         | 含义                 |
| ----------- | ------------------ |
| dateList    | 日期列表               |
| insertList  | 对应于日期列表的增加列表       |
| deleteList  | 对应于日期列表的删除列表       |
| ownerName   | 用户名列表              |
| ownerNum    | 对应于用户名列表的各用户修改条数列表 |
| companyName | 公司名列表              |
| companyNum  | 对应于公司名列表的各公司修改条数列表 |
| branchName  | 分支名列表              |
| branchNum   | 对应于分支名列表的各分支修改条数   |

用户参数说明（针对用户）
| 参数名         | 含义                 |
| ----------- | ------------------ |
| dateList    | 日期列表               |
| insertList  | 对应于日期列表的增加列表       |
| deleteList  | 对应于日期列表的删除列表       |
| projectName | 项目名列表              |
| projectNum  | 对应于项目名列表的各修改条数列表   |
| companyName | 公司名列表              |
| companyNum  | 对应于公司名列表的各公司修改条数列表 |
公司参数说明(针对公司)
| 参数名         | 含义                 |
| ----------- | ------------------ |
| dateList    | 日期列表               |
| insertList  | 对应于日期列表的增加列表       |
| deleteList  | 对应于日期列表的删除列表       |
| ownerName   | 用户名列表              |
| ownerNum    | 对应于用户名列表的各用户修改条数列表 |
| projectName | 项目名列表              |
| projectNum  | 对应于项目名列表的各修改条数列表   |



### 2.7 数据更新

数据更新部分和数据初始化部分合为一起，均在update_data.py中完成

关于更新的操作分为超级用户手动执行（admin部分）和系统自动更新执行（定时更新部分）

数据更新需要更新以下两个部分：

1. data目录中的record和info更新
2. 数据库中的更新

#### 2.7.1 data目录更新

data目录的更新需要执行两个函数：get_record_data和get_record_info_data，即更新record和更新info

更新record文件夹：

1. 查找数据库中最大的updated，即最晚的更新时间，记为result_date
2. 请求指定网址的json文件，判断json文件中每条记录的updated，如果存在updated大于result_date的记录，则执行第3步；否则，返回更新的文件名列表filename_list和result_date
3. 将第2步中的json写入到record文件夹中，文件名添加到更新文件名列表中；将指定网址的参数指向下一值，执行第2步

更新info文件夹：

1. 获得最晚的更新时间result_date
2. 对record文件下的文件夹遍历，如果其中的记录对应的updated大于result_date ，则将其对应的number存入到new_number中，并获取其info文件
3. 返回new_number列表

#### 2.7.2 数据库更新

数据库更新需要执行两个函数：insert_record_with_json和insert_info_with_json，即更新tvos_record表和更新tvos_info表

更新tvos_record表：

1. 接收new_number参数
2. 遍历record文件夹中的json文件，查看其中记录对应的number值，如果number值在new_number列表中，则判断数据库的tvos_record表中是否有主键为number的记录，有则删除，并将json文件中对应的记录添加到数据库中

更新tvos_info表：

1. 删除tvos_info表中所有记录
2. 遍历info文件夹中的文件，生成字典，并将修改次数大于2的记录插入到表中

#### 2.7.3 更新过程中可能出现的问题及考虑

最初的考虑是更新record文件夹，获取更新的文件名，只对更新的文件名作处理，但是各个环节并不是原子性的，比如以下两个问题：

1. 刷新时数据库和data目录不同步
2. 数据库未同步，没有及时刷新而是之后才刷新

此部分保证在执行update_data.py的过程中，不管出现什么样的问题，当问题解决后，再次执行update_data.py时和第一次执行的效果相同

具体如下：

1. 如果record文件夹或者info文件夹中的文件有缺失（人为所致，一般不存在），而又想恢复缺失文件（实际上没有意义），需要将数据库中两个表的记录全部清空（强烈不建议），再次执行update_data.py后效果不变（不过会重新生成info文件夹中的文件，非常慢）

2. 在执行完更新record文件夹的函数后，执行更新info文件夹的函数前，出现异常：

   此时record文件夹中已经更新完毕，因此当问题解决后，再次执行update_data.py后，get_record_data函数返回的更新文件名列表为空，此处表明对于其他三个函数是不能使用filename_list列表作为传入的参数，再次执行update_data.py后，达到的效果不变

3. 在执行完data目录更新后，出现异常：

   此时数据库没有得到更新，由于在get_record_info_data函数中遍历了所有record下的文件，因此该函数返回的参数new_number列表不会为空，而且对比的对象是数据库，再次执行update_data.py后，该参数仍然有效，可被后续函数调用，再次执行update_data.py后，达到的效果不变

4. 执行完更新表tvos_record后，出现异常，或者其他时候出现异常，均不会改变再次执行update_data.py时的效果

关于更新tvos_info表中第一步要删除所有记录的问题：

1. 首先可以获取到info中要更新的json文件，但是tvos_info表中的主键是项目名和文件名，因此在生成的字典中需要查看该条记录是否在tvos_info表中，如果在，则更新number值，否则插入该条记录，这样就是每次插入一条记录都要在这3万多条记录中去查看是否已经存在，太耗时，不可取，不过可以在该表中添加number字段
2. 因为以项目名和文件路径作为主键的记录有100多万条，实际上存入到数据库中的是修改次数大于2的记录，这样是有问题的，

针对以上2点，最好的办法就是删除所有记录，重新分析修改记录条数，再将修改次数大于2的记录添加到数据库中

#### 2.7.4 定时更新

ubuntu系统下，输入`crontab -e`进行编辑，其中crontab命令格式为：

```shell
* * * * * user command
分时日月周  用户  命令

#每周一的8点30更新一次
30 8 * * 1 root python3 文件路径/update_data.py

#如果要将输出的日志也记录下来，可以在任意目录下新建log文件(/opt/tvos/update.log)
30 8 * * 1 root python3 文件路径/update_data.py >> /opt/tvos/update.log
```

参考http://www.jb51.net/article/120484.htm?utm_source=debugrun&utm_medium=referral

## 3 后记

### 3.1错误与解决

#### 3.1.1 Django中静态文件路径

比如把`echarts.min.js`文件放置在templates下和index.html同级目录，按理直接使用

```html
<script src="echarts.min.js"></script>
```

便可，但是访问时却显示：Not Found charts.min.js

**解决：** 在app目录下（这个随意，会自动查找static文件夹）新建static文件夹，然后将js文件存放在其中，再在settings.py文件中声明static目录：

```python
STATIC_PATH=os.path.join(os.path.dirname(__file__),'static').replace('\\','/')
```

在index.html中使用

```html
<script src="/static/echarts.min.js"></script>
```

即可

#### 3.1.2 Python 内存溢出问题

在导入tvos_info表的时候，服务器内存不够，导致抛MemoryError，最后是将变量存入txt文件中，程序一行一行的取出来，再插入到数据库

#### 3.1.3 admin登入界面静态文件丢失

1. 首先找到admin的静态文件目录:`/usr/local/lib/python3.4/dist-packages/django/admin/static/`

   使用下面方法查看：终端进入python

   ```python
   >>>import django
   >>>django.__file__
   ```

   一般在contrib目录下

2. 然后再settings.py中的STATICFILES_DIRS下添加上面的路径

3. 在/ect/apache2/sites-enabled/mysite.conf文件中添加静态路径（即STATIC_ROOT的绝对路径）

#### 3.1.4 mysql远程连接

需要做两部分修改：

1. 修改被连接的数据库的mysql数据库下的user表，将其user=root且Host=localhost的记录修改为Host=%，表示允许所有ip访问

   并执行FLUSH PRIVILEGES 生效

2. 修改etc/mysql目录下的my.cnf文件，将其中的bind-address=127.0.0.1修改为0.0.0.0，即允许所有的ip访问

#### 3.1.5 Apache问题 

**apache AH01630: client denied by server configuration错误解决方法**

出现这个错误的原因是，apache2.4 与 apache2.2 的虚拟主机配置写法不同导致

### 3.2 tips

#### 3.2.1 CHAR  and  VARCHAR

在MySQL中

数据类型CHAR表示定长，一般存储身份证号，手机号等，如果不足，会用空格填充，效率极高

数据类型VARCHAR表示不定长，会有一个字节来存储数据的长度，不足不进行填充，效率低，适合变化长度的数据类型

#### 3.2.2 向sql语句中传递参数

传递一条数据：

```mysql
sql='INSERT INTO record_table VALUES ("%d","%s","%s","%s","%d","%d","%s")'
cursor.execute(sql%(number,project,branch,updated,insertions,deletions,owner))
```

批量导入（一条insert语句插入多条记录，效率高）：

```mysql
sql='INSERT INTO record_table VALUES %s'%'(7,"tvos/das","master","2009-09-17",22,33,"dang")'
```

这里直接使用executemany效率并不是很高

参考：[https://github.com/TsaiZehua/PyMySQL](https://github.com/TsaiZehua/PyMySQL)

#### 3.2.3 mysql语句

desc record_table  显示表结构

alter table record_table add column company varchar(16) not null;  添加列

删除表时，如果该表有外键约束，是删除不掉的，需要使用下面语句去除约束：

```
set foreign_key_checks=0
```



### 3.3 后期安排

1. tvos_info 表的分析问题
2. Docker



## 4 参考

【1】[ Django官方文档](https://docs.djangoproject.com/en/1.11/)

【2】[自强学堂 Django基础教程](http://code.ziqiangxuetang.com/django/django-tutorial.html)

【3】[ECharts](http://echarts.baidu.com/index.html)

【4】[Django+Vue](https://zhuanlan.zhihu.com/p/24893786)

【5】[跨域](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)



## 5 附录

详细数据与代码均在服务器上

### 5.1 数据获取部分代码

#### 5.1.1 获取三个版块的json文件

```python
import json
import requests


#获取三个版块的json文件
def get_record_data():
    #动态抓包获取各版块的json文件网址,存储在字典中
    section_dict={}
    section_dict['open']='http://120.25.200081/changes/?n=25&O=81'
    section_dict['merged']='http://120.25.9:8081/changes/?q=status:merged&n=25&O=81'
    section_dict['abandoned']='http://120.0.9:8081/changes/?q=status:abandoned&n=25&O=81'

    filename_list=[]#获取记录的文件名

    #遍历访问网址获取数据，以json存储在data中
    for key ,value in section_dict.items():
        flag = 1
        num = 0
        while flag:
            json_open = requests.get(value + "&S=" + str(num)).text#获取文件的文本形式
            json_open = json_open.strip().split(']}\'\n')[1]#去除文本中的第一行
            #这里小于5表示空，即next到头了，结束next；否则将数据写入文件
            if len(json_open) < 5:
                flag = 0
            else:
                with open('data/record/'+key+'_' + str(num) + '.json', 'w') as f:
                    f.write(json_open)
                    filename_list.append(key+'_'+str(num))
                    print(key+str(num)+"has wirtten !")
            num += 25
    return filename_list
```

#### 5.1.2 获取记录文件中对应的详细json文件

```python
#获取记录详细信息
def get_record_info_data(filename_list):
    for filename in filename_list:
        with open('data/record/'+filename+'.json','r') as f:
            records_json=json.load(f)
        for record in records_json:
            number=record['_number']
            print(number)
            url_str2='http://120.25.200.39:8081/changes/'+str(number)+'/detail?O=404'
            detail=requests.get(url_str2).text
            detail = detail.strip().split(']}\'\n')  # 去除文本中的第一行
            if len(detail)<2:continue
            detail=detail[1]
            detail='['+detail+']'
            with open ('data/info/'+filename.split('_')[0]+'/detail/detail'+str(number)+'.json','w') as f:
                f.write(detail)
            with open ('data/info/'+filename.split('_')[0]+'/detail/detail'+str(number)+'.json','r') as f:
                detail=json.load(f)
                print(filename+"detail"+str(number)+"has written!")
            revision=detail[0]['current_revision']

            url_str1 = 'http://120.25.200.39:8081/changes/'+str(number)+'/revisions/'+str(revision)+'/files'
            info=requests.get(url_str1).text
            info = info.strip().split(']}\'\n') # 去除文本中的第一行
            if len(info)<2:continue
            info=info[1]
            with open('data/info/' + filename.split('_')[0] +"/" + str(number) + '.json','w') as f:
                f.write(info)
                print(filename+str(number)+"has written!")
```

### 5.2 数据存储部分代码

#### 5.2.1 tvos_record表的数据插入

```python
#将全部record数据插入到tvos_record中
def insert_record_with_json():
    filelist = os.listdir('data/record')  # 获取data/record目录下的全部文件名
    batch_list=[]#存放记录
    db=connect_db()#连接数据库
    for filename in filelist:
        with open('data/record/' + str(filename), 'r') as f:
            record_data = json.load(f)
        #获取section
        if re.match(r'^abandoned.*',filename):section='abandoned'
        elif re.match(r'^merged.*',filename):section='merged'
        else:section='open'

        if len(record_data) < 1: continue
        for record in record_data:
            record_list=[]
            number=record['_number']
            project = record['project']
            branch = record['branch']
            updated= record['updated'][:10]
            insertions=record['insertions']
            deletions=record['deletions']
            owner=record['owner']['name']
            company=re.findall(r'@([a-zA-Z\-]+)',record['owner']['email'])
            if len(company)==0:company='None'#找不到公司名，则用None代替
            else:company=company[0]
            record_list.append(number)
            record_list.append(project)
            record_list.append(branch)
            record_list.append(updated)
            record_list.append(insertions)
            record_list.append(deletions)
            record_list.append(owner)
            record_list.append(company)
            record_list.append(section)
            batch_list.append(record_list)

            # 一次insert插入20条记录
            if len(batch_list) >=20:
                insert_db_batch(db, 'tvos_record',batch_list)
                batch_list = []

    if len(batch_list)>=1:
        insert_db_batch(db,'tvos_record', batch_list)
    close_db(db)

```

####5.2.2 tvos_info表的数据插入

```python
#将全部info数据插入到tvos_info中
def insert_info_with_json():
    dir=['open','merged','abandoned']
    insertions_dict={}
    deletions_dict={}
    num_dict={}
    db=connect_db()
    for d in dir:
        filelist = os.listdir('data/info/'+d)  # 获取data/record目录下的全部文件名

        for filename in filelist:
            num=re.findall(r'([0-9]+).json$',filename)#获取json文件
            if len(num)!=0:
                #查找number对应的project
                sql='SELECT project FROM tvos_record WHERE number='+num[0];
                cursor=db.cursor()
                cursor.execute(sql)
                result =cursor.fetchall()
                for row in result:
                    project=row[0]

                with open('data/info/' +d+'/'+ filename, 'r') as f:
                    info_data = json.load(f)
                #读取json中的路径以及对应的insert和delete
                for info in info_data:
                    project_filepath=project+'#'+info #以#为分隔符，将项目名和路径分开
                    if project_filepath not in num_dict.keys():
                        num_dict[project_filepath]=0
                        deletions_dict[project_filepath]=0
                        insertions_dict[project_filepath]=0
                    num_dict[project_filepath]+=1
                    if 'lines_inserted' in info_data[info]:
                        insertions_dict[project_filepath]+=info_data[info]['lines_inserted']
                    if 'lines_deleted' in info_data[info]:
                        deletions_dict[project_filepath]+=info_data[info]['lines_deleted']

    batch_list=[]
    info_list=[]
    id=0
    for profile in num_dict.keys():
        pro_file=profile.split('#')
        if num_dict[profile]<2:continue   #修改次数小于2的记录不添加到数据库
        id+=1
        #按照tvos_info表插入一条记录
        info_list.append(id)#id
        info_list.append(pro_file[1])#filepath
        info_list.append(pro_file[0])#project
        info_list.append(insertions_dict[profile])#insertions
        info_list.append(deletions_dict[profile])#deletions
        info_list.append(num_dict[profile])#num
        batch_list.append(info_list)
        info_list=[]
        # 一次insert插入20条记录
        if len(batch_list) >= 20:
            insert_db_batch(db,'tvos_info', batch_list)
            batch_list = []

    if len(batch_list) >= 1:
        insert_db_batch(db,'tvos_info', batch_list)
    close_db(db)
```

### 5.3 Django后台搭建部分代码

#### 5.3.1 view.py的部分代码

各版块数据获取

```python
#django模板会自动找到app下面的templates文件夹中的模板文件
#显示字符串
def index(request):
    sec=request.GET.get('section')
    if(sec==None or sec=='all'):
        record=Record.objects.all()#获取全部记录
    else:
        record = Record.objects.filter(section=sec)#获取各版块的记录
    
    project_dict={}
    owner_dict={}
    company_dict={}
    date_list=[]
    insert_list=[]
    delete_list=[]
    project_name=[]
    project_num=[]
    owner_name=[]
    owner_num=[]
    company_name=[]
    company_num=[]

    #读取数据
    for row in record:
        if row.project not in project_dict.keys():
            project_dict[row.project]=0
        if row.owner not in owner_dict.keys():
            owner_dict[row.owner]=0
        if row.company not in company_dict.keys():
            company_dict[row.company]=0
        project_dict[row.project]+=1
        owner_dict[row.owner]+=1
        company_dict[row.company]+=1
        date_list.append(str(row.updated))
        insert_list.append(row.insertions)
        delete_list.append(row.deletions)

    for key,value in project_dict.items():
        project_name.append(key)
        project_num.append(value)
    for key,value in owner_dict.items():
        owner_name.append(key)
        owner_num.append(value)
    for key,value in company_dict.items():
        company_name.append(key)
        company_num.append(value)

    return HttpResponse(json.dumps({'dateList': date_list,
                   'insertList': insert_list, 'deleteList': delete_list,
                   'ownerName': owner_name, 'ownerNum': owner_num,
                   'companyName': company_name, 'companyNum': company_num,
                   'projectName': project_name, 'projectNum': project_num}))
```

#### 5.3.2 admin.py

```python
from django.contrib import admin

from tvos.models import Record,Info
# Register your models here.

def make_published():
    #这部分路径需要修改
    os.system("python3 /Users/XX/floder/07practiceProject/TVOS/code/tvos1/update_data.py")
make_published.short_description="update Mysql"

#这个类表示网页中Record条目如何显示
class RecordAdmin(admin.ModelAdmin):
    #需要显示的字段
    list_display = ['number','project','branch','updated','insertions','deletions','owner','company','section']
    ordering=['-updated']#排序依据
    actions=[make_published]#点击行为

#Info
class InfoAdmin(admin.ModelAdmin):
    list_display = ['id','filepath','project','insertions','deletions','num']
    ordering = ['-num']
    actions = [make_published]
#将上面的类添加进来
admin.site.register(Record,RecordAdmin)
admin.site.register(Info,InfoAdmin)
```






