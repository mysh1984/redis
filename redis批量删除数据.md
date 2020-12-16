```bash
redis-cli -a <PASSWORD> EVAL "for _,k in ipairs(redis.call('keys','<KEYS*>')) do redis.call('del',k) end" 0 
# PASSWORD: redis密码
# KEYS*:    模糊匹配规则

模糊匹配的批量删除.
原子性操作,会造成redis短暂性无法提供服务
redis命令中没有-n 的话全部无法锁死
```


清除所有库所有key数据

```flushall```



清除单个库所有key数据
Redis Flushdb 命令用于清空当前数据库中的所有 key
```flushdb```

