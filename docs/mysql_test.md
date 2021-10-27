# Mysql  Varchar255 和 Varchar 小于255整体测试

### 直奔结果：差别不大

> `Mysql` 版本5.7.34，引擎 `InnoDB`

- `test1` 表
```
CREATE TABLE `test1` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `test_name` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3461376 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

- `test2` 表
```
CREATE TABLE `test2` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `test_name` varchar(50) COLLATE utf8mb4_unicode_ci NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3451376 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```


> 插入相同数据35w条，`test_name` 格式为 6177f6683f26c

### 查询测试

```angular2html
test1 表 Varchar(255)
SELECT * FROM test1 WHERE test_name = '6177f6683f263';
时间: 1.023s

test2 表 Varchar(50)
SELECT * FROM test2 WHERE test_name = '6177f6683f263';
时间: 1.008s
```


### 文件占用内存测试
```angular2html
test1 表 Varchar(255)
数据长度：129.64 MB (135,938,048)

test2 表 Varchar(50)
数据长度：129.64 MB (135,938,048)
# 实际上会有一丢丢大小差距
```

### Varchar(2048) 和 Varchar(4096) 测试
```angular2html
test1 表 Varchar(2048)
SELECT * FROM test1 WHERE test_name = '6177f6683f263';
时间: 1.070s

test2 表 Varchar(4096)
SELECT * FROM test2 WHERE test_name = '6177f6683f263';
时间: 1.102s
```

### 总结
> 差距其实并没有想象中那么大，在InnoDB引擎存储下 `Varchar255` 和 `Varchar小于255` 的都是1-2 byte保存长度，排序规则属于 `utf8mb4_unicode_ci` 情况下会将 16383(65535/4)  每个 unicode 占4个bytes长度， 一个表中 `Varchar` 总和最大不能超过`16383` ，也就是如果你的 `test_name(Varcahr10000)` 的话剩下 `Varchar` 字段总和加一起不能超过 `6383` 。
> 条件允许还是越小越好，虽然差距微乎其微。