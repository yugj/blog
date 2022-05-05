# 库存扣除

## 加锁更新存库
select for update方式

问题：并发性能低
## 乐观锁更新
在更新的时候，使用（CAS+版本号更新）+重试条件（重试次数或者重试时间限制）乐观锁的方式更新库存。此时，如果，客户A和客户B同时读取到库存剩余100，在更新的时候，有一个操作会失败。
```sql
select stock_remaining,version from stock where id = xx
updage stock set stock_remaining = xxx,version  where id = xxx and versiong=xxxxx 
```
可以保证数据的一致性；通过重试次数和重试时间的条件控制，可以防止过多的重试带来的数据库压力。

问题：高并发，如秒杀对DB压力较大

## redis扣减
预先将库存数据同步到redis，基于redis单线程特性和高并发特性，直接在redis扣减

理论上存在redis挂了导致数据不一致问题，但在大多情况下，可将redis作为稳定中间件使用
## 参考
https://www.shouxicto.com/article/2946.html