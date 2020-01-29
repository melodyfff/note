---
title: 【docker】安装部署 mongodb
tags: [docker,linux]
date: 2020-01-29
---

[Hub Docker](https://hub.docker.com/_/mongo?tab=description)

## 运行
```bash
# 运行
docker run -d --name mongodb \
    -v /root/mongo/db:/data/db \
    -p 27017:27017 \
    -e MONGO_INITDB_ROOT_USERNAME=admin \
    -e MONGO_INITDB_ROOT_PASSWORD=admin \
    mongo:4.2.3
```

## 连接
```bash
# 连接
docker exec -it mongodb mongo
docker exec -it mongodb sh -c 'exec mongo -u admin -p admin'
```

## 常用操作
```bash
# 认证
> db.auth("admin","admin")

# 创建用户
> db.createUser({ user:'user',pwd:'123456',roles:[ { role:'userAdminAnyDatabase', db: 'admin'}]});

# 查看数据库
>  show dbs

# 创建数据库
> use hello
> db.hello.insert({"hello":"world"})
> show dbs

# 删除数据库
> use hello
> db.dropDatabase()

# 创建集合
> db.createCollection(name, options) # name: 要创建的集合名称 

> db.createCollection("test")
> db.createCollection("mycol", { 
 capped : true, # （可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。当该值为 true 时，必须指定 size 参数
 autoIndexId : true, # （可选）如为 true，自动在 _id 字段创建索引。默认为 false
 size : 6142800, #（可选）为固定集合指定一个最大值，以千字节计（KB）
 max : 10000 # （可选）指定固定集合中包含文档的最大数量
 } )
> show collections

# 删除集合
> db.collection.drop()

> db.hello1.drop()

# 插入文档
> db.COLLECTION_NAME.insert(document)

> db.col.insert({title: '测试', 
    description: '测试',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
})
> db.col.find()

# 更新文档
> db.collection.update(
   <query>, # update的查询条件，类似sql update查询内where后面的。
   <update>, # update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
   {
     upsert: <boolean>, # 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
     multi: <boolean>, # 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新
     writeConcern: <document> # 可选，抛出异常的级别
   }
)

> db.col.update({'title':'测试'},{$set:{'title':'MongoDB'}})

# 删除文档
> db.collection.remove(
   <query>, #（可选）删除的文档的条件。
   {
     justOne: <boolean>, # （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
     writeConcern: <document> # （可选）抛出异常的级别。
   }
)
> db.col.remove({'title':'测试'}) # 会删除所有匹配
> db.COLLECTION_NAME.remove(DELETION_CRITERIA,1) # 只删除一条
```

## 数据导出与导入
**宿主机上**
```bash
# 导出db
docker exec mongodb sh -c 'exec mongodump --authenticationDatabase admin --username xxx --password xxx -d <database_name> --gzip --archive' > /some/path/on/your/host/all-collections.archive
# 导出单表
docker exec mongodb sh -c 'exec mongodump --authenticationDatabase admin --username xxx --password xxx -d <database_name> --collection col --gzip --archive' > /some/path/on/your/host/one-collections.archive

# 恢复暂时进入容器恢复
```

**进入容器中**
```bash
docker exec mongodb
# 导出
# 导出db
mongodump -h 127.0.0.1:27017 --authenticationDatabase admin -u "admin" -p "admin" -d hello -o test
# 导出单表
mongodump --authenticationDatabase admin -u "admin" -p "admin" -d hello --collection col -o test1

# 导入 --drop 覆盖已存在
# 导入db
mongorestore --authenticationDatabase admin -uadmin -padmin -d hello /test/hello/ --drop
# 导入单表
mongorestore --authenticationDatabase admin -uadmin -padmin -d hello --collection col /test1/hello/col.bson --drop


# 归档
mongodump -h 127.0.0.1:27017 --authenticationDatabase admin -u "admin" -p "admin" -d hello --gzip --archive=./h.archive

mongorestore -h 127.0.0.1:27017 --authenticationDatabase admin -u "admin" -p "admin" --gzip --archive=h.archive --drop
```