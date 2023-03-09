---
title: MongoDB

categories: 
- 技术
- 分布式
- 中间件
- 数据库

tags:
- Java
- 分布式存储引擎
- 高并发

description: MongoDB数据库
---



# MongoDB

## 概念

一种非关系型数据库(NoSQL)

应用场景：高并发的、需要低延时的，对事务要求、安全性不是很高的场景。

## 和Reids的区别？

Redis主要把数据存储在内存中，其“缓存”的性质远大于其“数据存储“的性质，其中数据的增删改查也只是像变量操作一样简单；

MongoDB却是一个“存储数据”的系统，增删改查可以添加很多条件，就像SQL数据库一样灵活，这一点在面试的时候很受用。

## 术语对比

![image-20221122105852998](https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20221122105852998.png)

## Windows安装/启动

下载zip格式的mongoDB包

```
https://www.mongodb.com/try/download/community
```

下载之后解压，在根目录下创建文件夹data/db

![image-20221122105904463](https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20221122105904463.png)

进入bin目录下进入cmd

执行如下

```cmd
mongod --dbpath=../data/db  #指定数据库存放位置
```

重新进入bin下的cmd，执行如下：

```cmd
mongo 或者 mongo --host=127.0.0.1 --port=27017 #27017是默认端口号
```

## Linux安装/启动

<img src="https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20221122105922151.png" alt="image-20221122105922151" style="zoom:50%;" />

配置内容如下：

```yaml
systemLog:
	#MongoDB发送所有日志输出的目标指定为文件
	# #The path of the 1log file to which mongod or mongos should send all diagnostic 1ogging information
	destination: file
    #mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
    path: "/mongodb/sing1e/1og/mongod. log"
    #当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
    logAppend: true
storage:
    #mongod实例存储其数据的目录。storage . dbpath设置仅适用于mongod.
    ##The directory where the mongod instance stores its data. Default value is "/data/db".
    dbpath: "/mongodb/sing1e/data/db"
    journal:
        #启用或禁用持久性日志以确保数据文件保持有效和可恢复。
        enabled: true
processManagement:
    #启用在后台运行mongos或mongod进程的守护进程模式。
    fork: true
net:
    #服务实例绑定的IP，默认是localhost
    bindIp: 1oca1host,192.168.0.2
    #绑定的端口，默认是27017
    port: 27017
```

启动：

```cmd
/usr/local/mongodb/bin/mongod -f /mongodb/single/mongod.conf
```

结果应该是Successfully

然后就可以使用可视化工具或者Shell命令行进行连接，如果连接不上，尝试关闭防火墙

```cmd
#查看防火墙状态 systemctl status firewalld 
#临时关闭防火墙 systemctl stop firewalld 
#开机禁止启动防火墙 systemctl disable firewalld
```

关闭数据库：

```cmd
//客户端登录服务，注意，这里通过localhost登录，如果需要远程登录，必须先登录认证才行。 
mongo --port 27017 //#切换到admin库 
use admin //关闭服务 
db.shutdownServer()
```

## 图形化界面

到官网下载Compass，直接运行即可

## 使用

### 数据库

```cmd
use 数据库名称 #选择/创建

show dbs #查看数据库(磁盘里的)

db #查看正在使用的数据库

db.dropDatabase() #删除数据库 db指的是数据库对象，操作的是当前使用的数据库
```

### 集合

```cmd
db.createCollection("My") #创建一个叫My的集合

show collections #查看所有集合

db.集合名.drop() #删除某个集合
```

### 文档

##### **文档的id值必须为字符串!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!不是数字**

#### 查询/插入

##### 普通查询

```cmd
db.集合名.insert({"name":"value"}) #单条数据的插入，若此时集合未被创建，会隐式地创建好集合

db.集合名.insertMany([
    {
        xxx:xxx
    },
    {
        xxx:xxx
    },
])  #插入多条数据

db.集合名.find() #查看集合内容

db.集合名.find({"name":"张三"}) #查询名字为张三的内容

db.集合名.findOne() / db.集合名.findOne({xxx:xxx})#查询一条数据，类似于limit

db.集合名.find({查询的条件},{字段1:1,字段2:0})#只显示部分字段,显示字段1，不显示字段2，以逗号分割
```

##### 统计查询 count

```cmd
db.comment.count() #查询所有数据数量

db.comment.count({条件})
```

##### 分页查询 limt/skip

```cmd
db.comment.find().limit(3) #限制查询数量
```

```cmd
db.comment.find().limit(2).skip(2)
```

##### 排序 sort

```cmd
db.comment.find().sort({xxx:1/0}) #1为升序，-1为降序
```

##### 正则查询 / /

```cmd
db.comment.find({字段:/正则表达式/})
```

##### 比较查询 gt/lt

```cmd
db.comment.find({xxx:{$gt:value}})

gt：大于
gte：大于等于

lt：小于
lte：小于等于
```

##### 包含查询 in/nin

```cmd
db.comment.find({xxx:{$in:["value1","value2"]}}) #包含

db.comment.find({xxx:{$nin:["value1","value2"]}}) #不包含
```

##### 多条件查询 and/or

```cmd
db.comment.find({$and:[{"xxx":"value"},{"xxx2":{$gt:NumberInt(233)}}]})


db.comment.find({$or:[{"xxx":"value"},{"xxx2":{$gt:NumberInt(233)}}]})
```



#### Try catch包裹插入语句

![image-20220714132811240](http://www.clewm.top/MarkDownImages/image-20220714132811240.png)

可以知道哪条数据插入失败

#### 更新

如下是覆盖更新

```cmd
db.comment.update({更新的条件},{更新的内容})
```

如下才是局部更新

```cmd
db.comment.update({更新的条件},{$set:{更新的内容}})
```

默认是只修改找到的第一条数据，若想修改全部

```cmd
db.comment.update({更新的条件},{$set:{更新的内容}},{multi:true})
```

使某个字段自增1

```cmd
db.comment.update({更新的条件},{$inc:{XXX:NumberInt(1)}})
```

#### 删除

```cmd
db.comment.remove({删除的条件})

db.comment.remove({}) #删除全部

#以上是过时的方法，下面是新的
use 数据库名
db.文档名.deleteOne({"uid":"123"})

db.文档名.deleteMany({"sex":"男"}) #删多个满足条件的

db.文档名.deleteMany({}) #删所有
```

### 索引

```cmd
db.comment.getIndexs() #查看索引

db.comment.createIndex(keys,options) #查看索引 options里常用的是name：指定索引名称和unique指定是否是唯一索引
```

#### 单字段索引

```cmd
db.comment.createIndex({"userId":1}) #给userId字段添加一个升序的单字段索引
```

#### 复合索引

```cmd
db.comment.createIndex({"userId":1,"age":-1}) #给userId字段添加一个升序的索引,age为降序的索引
```

#### 其他索引

<img src="https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20221122105945634.png" alt="image-20221122105945634" style="zoom:67%;" />

#### 删除索引

```cmd
db.comment.dropIndex({"userId":1}) #删除userId字段的升序的索引

db.comment.dropIndexes() #删除所有索引
```

### 性能检查

```cmd
db.comment.find(xxx).explain()
```

#### 覆盖查询

类似于MySQL中的覆盖索引

就是查询的字段正是索引的字段



## 整合到SpringBoot

依赖：

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
```

编写配置：

```yaml
# 应用名称
spring:
  application:
    name: mongotest
  data:
    mongodb:
      database: mytest
      host: 127.0.0.1
      port: 27017
#      username: 如果有
#      password: 如果有
server:
  port: 8990
```

创建实体类，并指定文档名

```java
@Document(collection = "user") //可以省略，省略的话默认文档名为实体类的小写名
@Data
public class User {

    //mongoDB里自带的id为字符串！！！！！！！！！！！！
    public String id;

    @Field("name")
    public String name;

    public Integer sex;

//    @Indexed 添加单字段索引
    public Long compId;

    public String phone;
}
```

注入模板类：

```java
    @Autowired
    private MongoTemplate mongoTemplate;
```

### 基本增添：

```java
    @GetMapping("/save")
    public String save() {
        User user = new User();
        user.setName("CleWM");
        user.setSex(1);
        user.setCompId(10001L);
        user.setPhone("18083822909");
        //insert和save的区别？
        //insert: 若新增数据的主键已经存在，则会抛 org.springframework.dao.DuplicateKeyException 异常提示主键重复，不保存当前数据。
        //save: 若新增数据的主键已经存在，则会对当前已经存在的数据进行修改操作
        User insert = mongoTemplate.insert(user);
        return insert.getId().toString();
    }
```

### 基本查询：

```java
    @GetMapping("/get")
    public String get() {
        Query query = new Query();
        query.addCriteria(Criteria.where("name").is("CleWM"));
        List<User> res = mongoTemplate.find(query, User.class);
        return res.get(0).toString();
    }
```

### 基本删除：

```java
mongoTemplate.remove(obj)
    
//根据条件删除
Query query = new Query();
query.addCriteria(Criteria.where("id").is(commentId));
Comment res = mongoTemplate.findAndRemove(query, Comment.class);
```

### 基本修改：

```java
Query query = new Query();
query.addCriteria(Criteria.where("id").is(commentId));
Comment res = mongoTemplate.findAndRemove(query, Comment.class);

Update update = new Update();
update.set("key","value");
mongoTemplate.updateFirst(query,update,Comment.class);
```

### 排序

```java
//实体类的字段需要实现Comparable接口，并重写方法，如下，注意，o和this的位置不能颠倒，否则升序和降序也会反过来。
    @Override
    public int compareTo(@NotNull Comment o) {
        return o.getLikes().compareTo(this.getLikes());
    }
```

```java
    @Override
    public List getList(CommentFuzzySearchDTO commentFuzzySearchDTO) {
        Query query = new Query().with(Sort.by("likes").descending()); //降序排序
        query.addCriteria(Criteria.where("item_id").is(commentFuzzySearchDTO.getItemId()));
        Integer curr = commentFuzzySearchDTO.getCurr();
        Integer pageSize = commentFuzzySearchDTO.getPageSize();
        List<Comment> commentList = mongoTemplate.find(query, Comment.class)
                .stream().skip((curr - 1) * pageSize).limit(pageSize).sorted().collect(Collectors.toList());
        return commentList;
    }
```

### 自增

```java
public Boolean likeInc(String commentId) {
    Query query = new Query();
    query.addCriteria(Criteria.where("_id").is(commentId));
    Update update = new Update();
    update.inc("likes");
    UpdateResult updateResult = mongoTemplate.updateFirst(query, update, Comment.class);
    long modifiedCount = updateResult.getModifiedCount();
    return modifiedCount > 0;
}
```

### 更多使用

参考https://blog.csdn.net/qq_36331657/article/details/116431191

https://blog.csdn.net/weixin_40392053/article/details/120265736



## 副本集

三个节点：

- 主节点，Primary
- 从节点，Slave 
- 选举节点，A也属于Slave，无法成为主节点

### 触发选举条件：

<img src="https://md-1259549904.cos.ap-shanghai.myqcloud.com/img/image-20221122110005391.png" alt="image-20221122110005391" style="zoom:67%;" />
