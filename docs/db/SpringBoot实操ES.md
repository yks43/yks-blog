## 简介
### 是什么
> Elasticsearch 是一个分布式、可扩展、实时的搜索与数据分析引擎。 它能从项目一开始就赋予你的数据以搜索、分析和探索的能力;
Elasticsearch使用 Java 编写的，它的内部使用 Lucene 做索引与搜索，但是它的目的是使全文检索变得简单， 通过隐藏 Lucene 的复杂性，取而代之的提供一套简单一致的 RESTful API。

### 特性
* 一个分布式的实时文档存储，每个字段 可以被索引与搜索
* 一个分布式实时分析搜索引擎
* 能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据

### 使用场景

-   搜索领域：如百度、谷歌，全文检索等。
-   门户网站：访问统计、文章点赞、留言评论等。
-   广告推广：记录用户行为数据、消费趋势、用户群体进行定制推广等。
-   信息采集：记录应用的埋点数据、访问日志数据等，方便大数据进行分析。



## 基本概念

### 对比DB

| DB                  | ES                  |
| ------------------- | ------------------- |
| Databases           | Indices             |
| Tables              | Types               |
| Rows                | Documents           |
| Columns             | Fields              |
| Schema              | Mapping             |
| Index               | Evething is indexed |
| SQL                 | Query DSL           |
| select * from table | GET http://         |
| update table set    | PUT http://         |

#### 映射（mapping)

>   映射规定了一个Fields中数据的类型，以及是否支持索引和被检索到

## 操作索引示例

### Restful操作示例

#### 创建索引

创建名为 mydlq-user 的索引与对应 Mapping。

```json
PUT /mydlq-user
{
  "mappings": {
    "doc": {
      "dynamic": true,
      "properties": {
        "name": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword"
            }
          }
        },
        "address": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword"
            }
          }
        },
        "remark": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword"
            }
          }
        },
        "age": {
          "type": "integer"
        },
        "salary": {
          "type": "float"
        },
        "birthDate": {
          "type": "date",
          "format": "yyyy-MM-dd"
        },
        "createTime": {
          "type": "date"
        }
      }
    }
  }
}
```



删除索引

删除 mydlq-user 索引。

```json
DELETE /mydlq-user
```

### Java操作示例

```java

```



## 操作文档示例

### Restful操作示例

在索引 mydlq-user 中增加一条文档信息。

```json
POST /mydlq-user/doc
{
    "address": "北京市",
    "age": 29,
    "birthDate": "1990-01-10",
    "createTime": 1579530727699,
    "name": "张三",
    "remark": "来自北京市的张先生",
    "salary": 100
}
```

### Java操作示例

```java

```



## 插入初始化数据

### 单条插入

```json
POST mydlq-user/_doc
{"name":"零零","address":"北京市丰台区","remark":"低层员工","age":29,"salary":3000,"birthDate":"1990-11-11","createTime":"2019-11-11T08:18:00.000Z"}
```



### 批量插入

```java
POST_bulk mydlq-user/_doc
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"刘一","address":"北京市丰台区","remark":"低层员工","age":30,"salary":3000,"birthDate":"1989-11-11","createTime":"2019-03-15T08:18:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"陈二","address":"北京市昌平区","remark":"中层员工","age":27,"salary":7900,"birthDate":"1992-01-25","createTime":"2019-11-08T11:15:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"张三","address":"北京市房山区","remark":"中层员工","age":28,"salary":8800,"birthDate":"1991-10-05","createTime":"2019-07-22T13:22:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"李四","address":"北京市大兴区","remark":"高层员工","age":26,"salary":9000,"birthDate":"1993-08-18","createTime":"2019-10-17T15:00:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"王五","address":"北京市密云区","remark":"低层员工","age":31,"salary":4800,"birthDate":"1988-07-20","createTime":"2019-05-29T09:00:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"赵六","address":"北京市通州区","remark":"中层员工","age":32,"salary":6500,"birthDate":"1987-06-02","createTime":"2019-12-10T18:00:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"孙七","address":"北京市朝阳区","remark":"中层员工","age":33,"salary":7000,"birthDate":"1986-04-15","createTime":"2019-06-06T13:00:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"周八","address":"北京市西城区","remark":"低层员工","age":32,"salary":5000,"birthDate":"1987-09-26","createTime":"2019-01-26T14:00:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"吴九","address":"北京市海淀区","remark":"高层员工","age":30,"salary":11000,"birthDate":"1989-11-25","createTime":"2019-09-07T13:34:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"郑十","address":"北京市东城区","remark":"低层员工","age":29,"salary":5000,"birthDate":"1990-12-25","createTime":"2019-03-06T12:08:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"萧十一","address":"北京市平谷区","remark":"低层员工","age":29,"salary":3300,"birthDate":"1990-11-11","createTime":"2019-03-10T08:17:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"曹十二","address":"北京市怀柔区","remark":"中层员工","age":27,"salary":6800,"birthDate":"1992-01-25","createTime":"2019-12-03T11:09:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"吴十三","address":"北京市延庆区","remark":"中层员工","age":25,"salary":7000,"birthDate":"1994-10-05","createTime":"2019-07-27T14:22:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"冯十四","address":"北京市密云区","remark":"低层员工","age":25,"salary":3000,"birthDate":"1994-08-18","createTime":"2019-04-22T15:00:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"蒋十五","address":"北京市通州区","remark":"低层员工","age":31,"salary":2800,"birthDate":"1988-07-20","createTime":"2019-06-13T10:00:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"苗十六","address":"北京市门头沟区","remark":"高层员工","age":32,"salary":11500,"birthDate":"1987-06-02","createTime":"2019-11-11T18:00:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"鲁十七","address":"北京市石景山区","remark":"高员工","age":33,"salary":9500,"birthDate":"1986-04-15","createTime":"2019-06-06T14:00:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"沈十八","address":"北京市朝阳区","remark":"中层员工","age":31,"salary":8300,"birthDate":"1988-09-26","createTime":"2019-09-25T14:00:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"吕十九","address":"北京市西城区","remark":"低层员工","age":31,"salary":4500,"birthDate":"1988-11-25","createTime":"2019-09-22T13:34:00.000Z"}
{"index":{"_index":"mydlq-user","_type":"doc"}}
{"name":"丁二十","address":"北京市东城区","remark":"低层员工","age":33,"salary":2100,"birthDate":"1986-12-25","createTime":"2019-03-07T12:08:00.000Z"}
```



## 查询示例

### 精确查询（term)



### 匹配查询（match）



### 模糊查询（fuzzy）



### 范围查询（range）



### 通配符查询（wildcard）



### 布尔查询（bool）



## 聚合查询

### Metric 聚合分析



### Bucket 聚合分析

