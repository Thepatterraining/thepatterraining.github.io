---
title: ElasticSearch十二--ES--显式mapping
date: 2021-05-05
tags: ['ES']
category: ES
article: ElasticSearch十二--ES--显式mapping
---

# 显式mapping

## 设置mapping

指令
```
PUT {index}
{
  "mappings":{
    "properties":{                 //设置mapping
      "job_name":{
        "type":"text"
      }
    }                                 
  }
}
```

## 自定义Mapping

可以参考 API 手册，纯手写

也可以复制现有的动态Mapping：
- 创建一个临时的index,写入一些样本数据
- 通过访问 Mapping API 获得该临时文件的动态 Mapping 定义
- 修改后，使用该配置创建你的索引
- 删除临时索引

### index

index 控制当前字段是否被索引。默认为true,如果设置成false，就不会被索引，不能被搜索。

优点：不会建立倒排索引，更节省内存空间
缺点：该字段不能被搜索

指令
```
PUT {index}
{
  "mappings":{
    "properties":{                 //设置mapping
      "job_name":{
        "type":"text",
        "index":false             //这里设置false，这个字段就不能被搜索
      }
    }                                 
  }
}
```

### index options

有四种不同级别的 index options 配置，可以控制倒排索引记录的内容
- docs - 记录 doc id
- freqs - 记录 doc id 和 term frequencies
- positions - 记录 doc id / term frequencies / term position
- offsets - 记录 doc id / term frequencies / term position / character offects

Text类型默认记录 positions ,其他默认为 docs

记录内容越多，占用存储空间越大。

指令
```
PUT {index}
{
  "mappings":{
    "properties":{                 //设置mapping
      "job_name":{
        "type":"text",
        "index_options":"offsets"             //这里设置index_options
      }
    }                                 
  }
}
```

### null value

 如果需要对 NULL 值进行搜搜，那么只有 `Keyword` 类型支持设定 null_value

 指令
```
PUT {index}
{
  "mappings":{
    "properties":{                 //设置mapping
      "job_name":{
        "type":"text",
        "null_value":"NULL"             //这里设置null_value
      }
    }                                 
  }
}
```


### copy_to

copy_to 相当于以前的 _all 。可以满足一些特定的搜索需求。copy_to 将字段的数值拷贝到目标字段，实现类似_all的作用。copy_to的目标字段不会出现在_source中

 指令
```
PUT {index}
{
  "mappings":{
    "properties":{                 //设置mapping
      "job_name":{
        "type":"text",
        "copy_to":"fullName"             //这里设置copy_to
      }
    }                                 
  }
}
```

### 数组类型

ES不提供专门的数组类型。但是任何字段，都可以包含多个相同类型的数值。

指令
```
PUT test_home/_doc/1
{
  "job_name":"php",
  "city":[
    "北京市","上海市"
  ]
}
```

可以看到 city 就是一个数组。但是如果查看 mapping 会发现 city 是一个 text/Keyword 类型。



> 极客时间 ES 学习笔记