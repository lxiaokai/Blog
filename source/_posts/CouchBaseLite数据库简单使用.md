---
title: CouchBaseLite数据库简单使用
date: 2018-09-29 10:09:00
tags: [couchbase]
---
### 简介
现在数据库方面的技术很多,也很成熟.但是很多公司也会有很奇怪的需求,明确你要使用什么技术.在数据量很多的时候,不能使用传统数据库的情况下,CouchBaseLite是一个很不错的选择.最起码比本地存储之类的方便,快捷,可以存储数据量比较大的数据,使用方法有点类似数据库.也称作NoSQL的存储方式.
 Couchbase Lite 是一个为满足在线和离线的移动应用所开发的超轻量的，可靠的，并且安全的JSON数据库。即使在最不确定的网络条件下，亦可以给您的移动应用提供富有成效的和可靠的信誉。除此之外，’同步门户’功能亦可以提供协作， 社交互动或者是用户的更新。
 
[官方DemoGitHub地址](https://github.com/couchbase/couchbase-lite-ios)
![image](http://upload-images.jianshu.io/upload_images/2656438-006f4832012982f0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--more-->

### 环境配置
[直通车](https://developer.couchbase.com/documentation/mobile/1.4/installation/ios/index.html)(方便大家直接访问~)
或者按照下面的方法:

- (1)我们先进去[官网](https://www.couchbase.com)
![image](http://upload-images.jianshu.io/upload_images/2656438-006f4832012982f0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- (2)点击进去,然后点击,install instructions(安装说明)

![image](http://upload-images.jianshu.io/upload_images/2656438-c144b025fd75d568.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- (3)就看到我们需要看到的内容啦. 然后选择我们的iOS

![image](http://upload-images.jianshu.io/upload_images/2656438-ba9da090dacce29b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

里面全部有介绍,怎么大家环境和使用方法.大家查看对应的说明搭建好环境就可以尝试使用了.
![image](http://upload-images.jianshu.io/upload_images/2656438-6475734770f76f0b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我的环境配置完成如下图(Xcode 9新建项目报错看最下面)
![image](http://upload-images.jianshu.io/upload_images/2656438-e702a25d71b6a9e1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
还得配置一些-objc
![image](http://upload-images.jianshu.io/upload_images/2656438-b437ccd1ad662b51.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)这些弄完就可以尝试编写了.

### 测试
我是直接基于原来的进行了一些简单的封装,在这里感谢@天梯传说@汪司机,给我很大的帮助.

- (1)在APPdelegate中启动数据库操作,也可以在你们想要的位置
```
/*打开或者创建数据库(一般在APPdelegate中)*/
    [DatabaseSetupUtil databaseOpenOrCreate];
```
- (2)引入文件
![image](http://upload-images.jianshu.io/upload_images/2656438-f71b5584a30932a7.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- (3)建立model层,自己定义了一些方法和变量,可以根据不用的需求定义不同.
.h文件
```
#define UserDataPackageCollectionName @"UserDataPackage"
/**
 用户资料model实体
 作用: 主要是存储一些用户信息
 */
@interface UserDataDBModel : DBModelBase
/*-----可以定义字典或者数组类型,demo不做举例------*/
@property (nonatomic,strong) NSString* resident;     //居民姓名
@property (nonatomic,strong) NSString* mobile;      //居民手机号码
@property (nonatomic,strong) NSString* idno;        //居民身份证号码
@property (nonatomic,strong) NSNumber* status;       //状态:1、未上传，2、已上传
@property (nonatomic,strong) NSString* note;      //备注
@property (nonatomic,strong) NSString* crtime;  //创建时间

//根据实体id获取包数据
+ (UserDataDBModel*)getModelByPackageID:(NSString*)reportId;
//根据上上传状态值获取包数据(根据时间排序)
+ (NSArray*)getModelsByStatus:(NSNumber*)status;
//获取所有包数据(根据时间排序)
+ (NSArray*)getAllModels;
```
.m文件
```
- (instancetype)init
{
    self = [super init];
    if (self) {
        //文档所属的记录集
        self.collection = UserDataPackageCollectionName;
    }
    return self;
}

- (id)mutableCopy {
    UserDataDBModel* model = [[UserDataDBModel alloc] init];
    //这三个是必须的字段
    model._id = [self._id mutableCopy];
    model._rev = [self._rev mutableCopy];
    model.collection = [self.collection mutableCopy];
//中间代码省略...
    
    return model;
}

//实体转词典
- (NSDictionary*)toDictionary {
    NSMutableDictionary* dic = [NSMutableDictionary dictionary];
//中间代码省略...
    }

//词典转实体
+ (UserDataDBModel*)fromDictionary:(NSDictionary*)dictionary {
    UserDataDBModel* ret = [[UserDataDBModel alloc] init];
//中间代码省略...
}
    
   
#pragma mark -- 扩展方法

//根据体检报告id获取体检报告
+ (UserDataDBModel*)getModelByPackageID:(NSString*)reportId {
   //中间代码省略
}

//根据状态值获取体检报告
+ (NSArray*)getModelsByStatus:(NSNumber*)status {
  //中间代码省略
}

//获取所有体检报告
+ (NSArray*)getAllModels {
//中间代码省略
}
```
- (4)ViewController中引入文件
```
#import "DatabaseSetupUtil.h"
#import "UserDataDBModel.h"
```
实现下面方法即可
```
//1.增加数据
- (void)addUserData {
    _model = [[UserDataDBModel alloc] init];
    _model.resident = @"张三";
    _model.mobile = @"123456789111";
    _model.idno = @"546456456575675";
    _model.note = @"假设是资料";
    _model.status = @0;
    _model.crtime = [self getCurrentUTCTime];
}

//保存
- (void)saveDate {
    if(![UserDataDBModel createDocument:_model]) {
        //失败
    }
    else {
        //成功
        NSArray* updateDatas = [UserDataDBModel getModelsByStatus:@0];
        for (UserDataDBModel* model in updateDatas) {
            NSLog(@"文档唯一性id: %@",model._id);
        }
    }
}
```
至于剩下的更新和删除的操作都是很简单的事情,直接DBModelBase父类的方法或者在model中重写一下父类的方法.
### 整体框架
[图片上传失败...(image-ad9ad9-1510651510810)]

下面的主要的.h文件的代码.
DBManager.h
```
/* 数据库 */
@property(strong,nonatomic,readonly) CBLDatabase*
database;

/* 是否启用数据库加密，默认不启用 */
@property(nonatomic) BOOL encrypt;

/**
 *  获取单例管理器对象
 */
+ (DBManager*)shareInstance;

/**
 *  根据数据库存放的路径以及数据库名字创建数据库
 * 参数 path ：数据库存放的路径
 * 参数 name ：数据库存放的路径
 * 返回值 ：如果创建成功或者指定的路径已经存在指定名字的数据库则返回YES,否则返回NO
 */
-(BOOL)createDBWithPath:(NSString*)path  name:(NSString*)name;

/**
 *  根据数据库存放的路径以及数据库名字打开数据库
 * 参数 path ：数据库存放的路径
 * 参数 name ：数据库存放的路径
 * 返回值 ：如果打开成功返回YES,否则返回NO
 */
-(BOOL)openDBWithPath:(NSString*)path  name:(NSString*)name;

/**
 *  删除数据库(必须先打开数据库才能执行删除操作)
 * 参数 path ：数据库存放的路径
 * 参数 name ：数据库存放的路径
 * 返回值 ：如果如果成功或者不存在指定的路径指定名字的数据库则返回YES,否则返回NO
 */
-(BOOL)deleteDB;

/**
 *  执行事务性操作
 * 参数 block ：事务block,一旦该block返回No即会执行回滚操作，返回Yes表示事务执行完成
 * 返回值 ：如果成功则返回YES,否则返回NO
 */
- (BOOL)inTransaction: (BOOL(^)(void))block;

/**
 *  执行事务性操作(异步模式)
 * 参数 block ：事务block,一旦该block返回No即会执行回滚操作，返回Yes表示事务执行完成
 * 参数 competition ：完成回调block ，如果成功则返回YES,否则返回NO
 */
- (void)inTransaction: (BOOL(^)(void))block competition:(void (^)(BOOL result))competition;
```
SingleJobQueue.h
```
/**
 *  获取单例对象
 */
+ (SingleJobQueue*)shareInstance;

/**
 *  以block方式运行工作任务
 *
 *  @param jobBlock 工作任务block
 */
- (void)runWithJobBlock:(void (^)(void))jobBlock;

/**
 *  以block方式异步运行工作任务(不要问我为什么没有同步~)
 *
 *  @param jobBlock 工作任务block
 *  @param competition 完成回调block
 */
- (void)runAsyncJobWithJobBlock:(void (^)(void))jobBlock competition:(void (^)(void))competition;
```
DBModelBase.h
```
@property(strong,nonatomic) NSString* _id; //文档唯一性id
@property(strong,nonatomic) NSString* _rev; //文档版本
@property(strong,nonatomic) NSString* collection; //文档所属的记录集
@property(strong,nonatomic) NSMutableDictionary* extendFields;  //动态扩展字段

//实体转词典,子类必须重写
- (NSDictionary*)toDictionary;

//词典转实体
+ (id)fromDictionary:(NSDictionary*)dictionary;

//转换成服务端JSON
- (NSData*)toServerJSON;

/**
 *  根据指定的实体对象创建一个文档
 * 参数 model ：继承自实体基类的具体实体实例
 * 返回值 ：如果创建成功则返回文档id,否则返回nil
 */
+(NSString*)createDocument:(id)model;

/**
 *  根据指定的实体对象创建一个文档(异步模式)
 * 参数 model ：继承自实体基类的具体实体实例
 * 参数 competition ：完成回调block ，如果创建成功则返回文档id,否则返回nil
 */
+(void)createDocument:(id)model competition:(void (^)(NSString* docId))competition;

/**
 *  根据指定的实体对象创建一个文档(适合于在事务中执行)
 * 参数 model ：继承自实体基类的具体实体实例
 * 返回值 ：如果创建成功则返回文档id,否则返回nil
 */
+(NSString*)createDocumentForTransaction:(id)model;

/**
 *  删除指定id的文档
 * 参数 documentId ：文档id
 * 返回值 ：如果创建成功则返回YES,否则返回NO
 */
+(BOOL)deleteDocumentWithId:(NSString*)documentId;

/**
 *  删除指定id的文档(异步模式)
 * 参数 documentId ：文档id
 * 参数 competition ：完成回调block ，如果创建成功则返回YES,否则返回NO
 */
+(void)deleteDocumentWithId:(NSString*)documentId competition:(void (^)(BOOL result))competition;

/**
 *  删除指定id的文档(适合于在事务中执行)
 * 参数 documentId ：文档id
 * 返回值 ：如果创建成功则返回YES,否则返回NO
 */
+(BOOL)deleteDocumentForTransactionWithId:(NSString*)documentId;


/**
 *  根据指定的实体对象更新符合条件的文档
 * 参数 model ：条件参数
 * 参数 newModel ：新的文档值参数，继承自实体基类的具体实体实例
 * 返回值 ：如果创建成功则返回YES,否则返回NO
 */
+(BOOL)updateDocument:(NSPredicate*)where newModel: (id) newModel;

/**
 *  根据指定的实体对象更新符合条件的文档(异步模式)
 * 参数 model ：条件参数
 * 参数 newModel ：新的文档值参数，继承自实体基类的具体实体实例
 * 参数 competition ：完成回调block ，如果创建成功则返回YES,否则返回NO
 */
+(void)updateDocument:(NSPredicate*)where newModel: (id) newModel competition:(void (^)(BOOL result))competition;

/**
 *  根据指定的实体对象更新符合条件的文档
 * 参数 model ：条件参数
 * 参数 newModel ：新的文档值参数，继承自实体基类的具体实体实例
 * 返回值 ：如果创建成功则返回YES,否则返回NO
 */
+(BOOL)updateDocumentForTransaction:(NSPredicate*)where newModel: (id) newModel;

/**
 *  根据指定的文档id查询文档
 * 参数 documentId ：文档id
 * 返回值 ：如果查询成功则返回继承自实体基类的具体实体实例,否则返回nil
 */
+(id)getDocumentWithId:(NSString*)documentId;

/**
 *  根据指定的文档id查询文档(异步模式)
 * 参数 documentId ：文档id
 * 参数 competition ：完成回调block,如果查询成功则返回继承自实体基类的具体实体实例,否则返回nil
 */
+(void)getDocumentWithId:(NSString*)documentId competition:(void (^)(id doc))competition;

/**
 *  根据指定的文档id查询文档(适合于在事务中执行)
 * 参数 documentId ：文档id
 * 返回值 ：如果查询成功则返回继承自实体基类的具体实体实例,否则返回nil
 */
+(id)getDocumentForTransactionWithId:(NSString*)documentId;

/**
 *  根据指定的实体对象获取符合条件的文档
 * 参数 select ：获取字段参数
 * 参数 where ：条件参数
 * 参数 orderBy ：排序参数
 * 返回值 ：如果查询成功则返回继承自实体基类的具体实体实例数组,否则返回nil
 */
+(NSArray*)getDocumentWithSelect: (nullable NSArray*)select where:(NSPredicate*)where orderBy:(nullable NSArray*)orderBy;

/**
 *  根据指定的实体对象获取符合条件的文档(异步模式)
 * 参数 select ：获取字段参数
 * 参数 where ：条件参数
 * 参数 orderBy ：排序参数
 * 参数 competition ：完成回调block,如果查询成功则返回继承自实体基类的具体实体实例数组,否则返回nil
 */
+(void)getDocumentWithSelect: (nullable NSArray*)select where:(NSPredicate*)where orderBy:(nullable NSArray*)orderBy competition:(void (^)(NSArray*))competition;

/**
 *  根据指定的实体对象获取符合条件的文档(适合于在事务中执行)
 * 参数 select ：获取字段参数
 * 参数 where ：条件参数
 * 参数 orderBy ：排序参数
 * 返回值 ：如果查询成功则返回继承自实体基类的具体实体实例数组,否则返回nil
 */
+(NSArray*)getDocumentForTransactionWithSelect: (nullable NSArray*)select where: (NSPredicate*)where orderBy:(nullable NSArray*)orderBy;

/**
 *  根据指定的实体对象获取符合条件的文档
 * 参数 select : 获取字段参数
 * 参数 where : 条件参数
 * 参数 orderBy : 排序参数
 * 参数 skip : 跳过多少条
 * 参数 limit : 读取多少条
 * 返回值 ：如果查询成功则返回继承自实体基类的具体实体实例数组,否则返回nil
 */
+(NSArray*)getDocumentWithSelect: (nullable NSArray*)select where:(NSPredicate*)where orderBy:(nullable NSArray*)orderBy skip:(NSInteger)skip limit:(NSInteger)limit;

/**
 *  根据指定的实体对象获取符合条件的文档(异步模式)
 * 参数 select : 获取字段参数
 * 参数 where : 条件参数
 * 参数 orderBy : 排序参数
 * 参数 skip : 跳过多少条
 * 参数 limit : 读取多少条
 * 参数 competition ：完成回调block,如果查询成功则返回继承自实体基类的具体实体实例数组,否则返回nil
 */
+(void)getDocumentWithSelect: (nullable NSArray*)select where:(NSPredicate*)where orderBy:(nullable NSArray*)orderBy skip:(NSInteger)skip limit:(NSInteger)limit competition:(void (^)(NSArray*))competition;

/**
 *  根据指定的实体对象获取符合条件的文档(适合于在事务中执行)
 * 参数 select : 获取字段参数
 * 参数 where : 条件参数
 * 参数 orderBy : 排序参数
 * 参数 skip : 跳过多少条
 * 参数 limit : 读取多少条
 * 返回值 ：如果查询成功则返回继承自实体基类的具体实体实例数组,否则返回nil
 */
+(NSArray*)getDocumentForTransactionWithSelect: (nullable NSArray*)select where:(NSPredicate*)where orderBy:(nullable NSArray*)orderBy skip:(NSInteger)skip limit:(NSInteger)limit;
```
上面的主要是封装的部分方法和文件说明
具体的请查看[demo](https://github.com/lxiaokai/CouchbaseLite_NoSQL.git)

### 小结
这个CouchBaseLite个人觉得还是蛮好用的,简单粗暴...,和以往的数据库操作基本一样,也算是比较轻量级的存储了.
上面主要是自己的一些理解的封装,还有很多需要完善,目前只是简单的实现了部分功能,这个后面会逐渐完善的.有什么写错的,请大家指出,一起讨论.
