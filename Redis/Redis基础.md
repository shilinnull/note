# Redis基础

```SQL
SET key value [expiration EX seconds|PX milliseconds] [NX|XX]
```

`[]`相当于一个独立的单元，表示可选项 (可有可无的)其中`|`表示“或者”的意思，多个只能出现一个
`[]` 和 `[]` 之间,是可以同时存在的

如果 key 不存在，创建新的键值对。
如果 key存在，则是让新的value覆盖旧的value.可能会改变原来的数据类型原来这个key的`ttl`(生存时间)也会失效

![image-20260119203825330](./Redis基础.assets/image-20260119203825330.png)

`FLUSHALL` 可以把 redis 上所有的键值对都带走
