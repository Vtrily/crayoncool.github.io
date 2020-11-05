# Redis常用命令


Redis常用命令

<!--more-->

## Redis连接

``` redis
redis-cli -h xxx -p 6379 -a password
```

## 获取key

``` redis
get key
```

## 设置过期时间 

``` redis
expire key times
```

## 删除key

``` redis
del key
```

## 选择db

``` redis
select
```

## NOAUTH Authentication required 解决方案

``` redis
auth password
```