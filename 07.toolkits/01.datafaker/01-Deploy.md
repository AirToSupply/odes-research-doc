### 一、源码安装

1.拉取Datafaker master代码

```shell
git clone https://github.com/gangly/datafaker.git
```

2.进去源码目录中执行

```shell
python3 setup.py install
```

3.查看Datafaker 是否安装成功

```shell
datafaker --version
0.7.6
```

4.修改用户

```shell
sudo chown -R zhongtai:zhongtai datafaker源码目录
```



对于不同的数据库需要用到不同的python包，若在执行过程中报包缺失问题。 请pip安装对应包

| 数据库              | python包                 | 备注                             |
| ------------------- | ------------------------ | -------------------------------- |
| mysql/tidb          | mysql-python/mysqlclient | windows+python3请使用mysqlclient |
| oracle              | cx-Oracle                | 同时需要下载orale相关库          |
| postgresql/redshift | psycopg2                 | 根据sqlachemy选择对应包          |
| Hbase               | happybase,thrift         |                                  |
| es                  | elasticsearch            |                                  |
| hive                | pyhive                   |                                  |
| kafka               | kafka-python             |                                  |


### 二、测试

##### 在mysql中创建学生表

------

```sql
create table stu (
	id int unsigned auto_increment primary key COMMENT '自增id',
	name varchar(20) not null comment '学生名字',
	school varchar(20) not null comment '学校名字',
	nickname varchar(20) not null comment '学生小名',
	age int not null comment '学生年龄',
	class_num int not null comment '班级人数',
	score decimal(4,2) not null comment '成绩',
	phone bigint not null comment '电话号码',
	email varchar(64) comment '家庭网络邮箱',
	ip varchar(32) comment 'IP地址',
	address text comment '家庭地址'
) engine=InnoDB default charset=utf8;
```



##### 编写元数据meta.txt，这是个学生表描述。 其中meta.txt文件内容为：

```shell
id||int||自增id[:inc(id,1)]
name||varchar(20)||学生名字
school||varchar(20)||学校名字[:enum(file://names.txt)]
nickname||varchar(20)||学生小名[:enum(鬼泣, 高小王子, 歌神, 逗比)]
age||int||学生年龄[:age]
class_num||int||班级人数[:int(10, 100)]
score||decimal(4,2)||成绩[:decimal(4,2,1)]
phone||bigint||电话号码[:phone_number]
email||varchar(64)||家庭网络邮箱[:email]
ip||varchar(32)||IP地址[:ipv4]
address||text||家庭地址[:address]
```



##### meta.txt文件中每行数据为元数据的一个字段描述，以||分割为三列，若以#开头，则忽略该行。

- 第一列：字段名
- 第二列：表字段类型
- 第三列：字段注释，其中包含构造规则标识

name不加标记则会随机产生20字符内的字符串，可以加上改为：`学生名字[:name]`

##### 其中学校名字[:enum(file://names.txt)]表示从本地文件names.txt中读取枚举数据，表示学校名称只能从下面这5所学校中随机产生。names.txt内容如下：

```shell
清华中学
人和中心
广东中学
猪场
旧大院
```



后面将详细介绍构造规则说明

注意：meta.txt和names.txt需要放在同一个目录下，再运行datafaker命令 如果没有enum类型从文件读取数据，则不需要names.txt文件



##### 从本地文件meta.txt中读取元数据，以,,分隔符构造10条数据，打印在屏幕上

------

```shell
$ datafaker rdb mysql+mysqldb://root:root@localhost:3600/test?charset=utf8 stu 10 --outprint --meta meta.txt --outspliter ,,

1,,鲍红,,人和中心,,高小王子,,3,,81,,55.6,,13197453222,,mwei@gmail.com,,192.100.224.255,,江苏省西宁市梁平朱路I座 944204
2,,刘东,,清华中学,,高小王子,,3,,31,,52.4,,15206198472,,lili@kong.cn,,203.0.190.6,,内蒙古自治区嘉禾市兴山呼和浩特街E座 706421
3,,匡静,,人和中心,,歌神,,9,,84,,72.51,,18944398099,,zouchao@gmail.com,,203.1.53.166,,安徽省永安市沈河惠州街x座 345415
4,,王宇,,猪场,,逗比,,6,,89,,19.3,,18628114285,,na58@cai.net,,169.4.126.215,,山西省梧州县朝阳何路y座 846430
5,,陆桂芝,,猪场,,逗比,,8,,99,,92.22,,13304570255,,na55@ti.cn,,168.136.127.200,,江苏省英县徐汇尹街C座 908240
6,,顾阳,,猪场,,歌神,,9,,32,,43.14,,18025578420,,linping@pr.net,,174.50.222.39,,黑龍江省惠州县梁平大冶街Z座 611736
7,,杨洁,,人和中心,,鬼泣,,6,,35,,81.25,,13654306263,,minzhong@xiaxia.cn,,100.57.79.2,,湖北省琳市沙湾汪街V座 544660
8,,申璐,,人和中心,,鬼泣,,6,,14,,73.61,,13866020503,,changxiulan@chaoxia.cn,,198.248.254.56,,陕西省合山县东丽宁德街Q座 810017
9,,申强,,广东中学,,逗比,,7,,48,,90.65,,13915915013,,ysun@chao.cn,,169.210.122.39,,甘肃省冬梅县城北六安街Z座 619755
10,,李丹丹,,旧大院,,鬼泣,,3,,67,,87.63,,18899812516,,xiulanmo@qin.cn,,192.52.218.133,,湖南省宜都县萧山澳门街E座 791911
generated records : 10
printed records : 10
time used: 0.458 s
```



##### 从本地文件meta.txt中读取元数据，以,,分隔符构造10条数据，写入mysql

```shell
$ datafaker rdb mysql+mysqldb://root:root@localhost:3600/test?charset=utf8 stu 10  --meta meta.txt
```

若要再次运行，需要修改meta.txt文件中为`id[:inc(id,11)]`起始值为11或更大值，不然数据库会报主键重复错误。



##### 构造规则优先级：

解析器将优先选择第三列的带规则标记的字段注释进行解析，如果不带标记，则选择第二列的字段类型进行解析。

这种好处是：

1）对应已经创建的数据表，用户可以用desc tablename 或者show full columns from tablename，将表shema查询复制下来，对用字段类型构造数据不满足的情况下，在注释里面进行打标机进行特殊处理

2）对于新表，在create table创建表时直接在注释里面打上标记。这种情况不用指定元数据文件。



##### 写hive：产生1000条数据写入hive的test库，stu表中

其中yarn为用户名，需要hive版本支持acid，不然请生成本地文件，然后上传到hdfs

```shell
datafaker hive hive://yarn@localhost:10000/test stu 1000 --meta data/hive_meta.txt
```



##### 写文件：产生10条json格式数据写入到/home目录out.txt中

------

```shell
datafaker file /home out.txt 10 --meta meta.txt --format json
```



#####  写kafka：从本地meta.txt参数数据，以1秒间隔输出到kafka的topic hello中

------

```shell
$ datafaker kafka localhost:9092 hello 1 --meta meta.txt --interval 1
{"school": "\u4eba\u548c\u4e2d\u5fc3", "name": "\u5218\u91d1\u51e4", "ip": "192.20.103.235", "age": 9, "email": "chaokang@gang.cn", "phone": "13256316424", "score": 3.45, "address": "\u5e7f\u4e1c\u7701\u5b81\u5fb7\u5e02\u6d54\u9633\u5468\u8defu\u5ea7 990262", "class_num": 24, "nickname": "\u9017\u6bd4", "id": 1}
{"school": "\u4eba\u548c\u4e2d\u5fc3", "name": "\u6768\u4e3d", "ip": "101.129.18.230", "age": 3, "email": "min60@hv.net", "phone": "18183286767", "score": 22.16, "address": "\u8fbd\u5b81\u7701\u592a\u539f\u5e02\u53cb\u597d\u6c55\u5c3e\u8defG\u5ea7 382777", "class_num": 30, "nickname": "\u6b4c\u795e", "id": 2}
{"school": "\u6e05\u534e\u4e2d\u5b66", "name": "\u8d75\u7ea2", "ip": "192.0.3.34", "age": 9, "email": "fxiao@gmail.com", "phone": "18002235094", "score": 48.32, "address": "\u5e7f\u897f\u58ee\u65cf\u81ea\u6cbb\u533a\u65ed\u5e02\u6c88\u5317\u65b0\u6731\u8defc\u5ea7 684262", "class_num": 63, "nickname": "\u6b4c\u795e", "id": 3}
{"school": "\u6e05\u534e\u4e2d\u5b66", "name": "\u5f20\u7389\u6885", "ip": "198.20.50.222", "age": 3, "email": "xiulanlei@cw.net", "phone": "15518698519", "score": 85.96, "address": "\u5b81\u590f\u56de\u65cf\u81ea\u6cbb\u533a\u6d69\u53bf\u767d\u4e91\u4e4c\u9c81\u6728\u9f50\u8857s\u5ea7 184967", "class_num": 18, "nickname": "\u9017\u6bd4", "id": 4}
{"school": "\u732a\u573a", "name": "\u674e\u6842\u5170", "ip": "192.52.195.184", "age": 8, "email": "fxiao@konggu.cn", "phone": "18051928254", "score": 97.87, "address": "\u9ed1\u9f8d\u6c5f\u7701\u54c8\u5c14\u6ee8\u53bf\u6c38\u5ddd\u6d2a\u8857E\u5ea7 335135", "class_num": 46, "nickname": "\u9ad8\u5c0f\u738b\u5b50", "id": 5}
{"school": "\u4eba\u548c\u4e2d\u5fc3", "name": "\u5434\u60f3", "ip": "192.42.234.178", "age": 3, "email": "uliang@yahoo.com", "phone": "14560810465", "score": 6.32, "address": "\u5b81\u590f\u56de\u65cf\u81ea\u6cbb\u533a\u516d\u76d8\u6c34\u5e02\u5357\u6eaa\u7f57\u8857M\u5ea7 852408", "class_num": 12, "nickname": "\u9b3c\u6ce3", "id": 6}
^Cgenerated records : 6
insert records : 6
time used: 6.285 s
```



消费端验证:

[![数据消费](https://github.com/gangly/datafaker/raw/master/doc/img/kafka.png)](https://github.com/gangly/datafaker/blob/master/doc/img/kafka.png)

json嵌套或任意数据结构(可不是jon)

```shell
datafaker kafka localhost:9092 hello 10 --metaj meta.txt
```

请使用--metaj指定元数据文件meta.txt：

```shell
{
    "name": [:name],
    "age": [:age],
    "school": {
        "sch_name": [:enum(file://../data/names.txt)],
        "sch_address": [:address],
        "scores": [
            {
                "class": [:enum(Math, English)],
                "score": [:decimal(4,2,1)]
            },
            {
                "class": [:enum(Chinese, Computer)],
                "score": [:decimal(4,2,1)]
            }
        ]
    }
}
```

datafaker会替换meta.txt内容中带标记的字符串，并保留原格式，包括tab和空格，产生如下结果：

```shell
{
    "name": 驷俊,
    "age": 95,
    "school": {
        "sch_name": 旧大院,
        "sch_address": 湖北省济南市上街宁德路I座 557270,
        "scores": [
            {
                "class": Math,
                "score": 83.28
            },
            {
                "class": Computer,
                "score": 52.37
            }
        ]
    }
}
```



如果要使用正确格式的json,将元数据文件内容压缩

```json
{"name":[:name],"age":[:age],"school":{"sch_name":[:enum(file://../data/names.txt)],"sch_address":[:address],"scores":[{"class":[:enum(Math,English)],"score":[:decimal(4,2,1)]},{"class":[:enum(Chinese,Computer)],"score":[:decimal(4,2,1)]}]}}
```



##### 写hbase

------

```shell
datafaker hbase localhost:9090 test-table 10 --meta data/hbase.txt
```



需要开启hbase thrift服务,不能为thrift2 例子中，创建一张表test-table, 列族为Cf 元数据文件hbase.txt内容为

```shell
rowkey||varchar(20)||sdflll
Cf:name||varchar(20)||学生名字
Cf:age||int||学生年龄[:age]
```



其中第一行必须为rowkey, 可带参数，rowkey(0,1,4)表示将rowkey值和后面第一列，第五列值用_连接

后面行为列族中的列名，可以创建多个列族



##### 写入ES

------

```shell
datafaker es localhost:9200 example1/tp1 100 --auth elastic:elastic --meta meta.txt
```



其中localhost:9200为es的连接方式，多个host用逗号分隔。如host1:9200,host2:9200

example1/tp1为index和type，以/分隔

elastic:elastic为账号和密码，若没有，则可不带该参数



#####  数据写入oracle

------

```shell
datafaker rdb oracle://root:root@127.0.0.1:1521/helowin stu 10 --meta meta.txt
```

sqlalchemy连接串必须为oracle:形式