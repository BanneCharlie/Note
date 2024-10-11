Taro(用户端) + Ruoyi(管理后台) + Mysql(存储数据库) 

OpenAPi3根据后端的接口,自动生成前端的请求;

中间件: Redis(缓存数据库) + RabbitMQ + XXL-job(定时任务)  实现功能的扩展


```shell
# docker 部署 redis
docker run -p 6379:6379 --name redis \
-v /EliteSelection/redis/redis-data:/data \
-v /EliteSelection/redis/redis-config/redis.conf:/etc/redis/redis.conf \
-d redis redis-server /etc/redis/redis.conf
```
