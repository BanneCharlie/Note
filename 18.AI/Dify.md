阿里云LLM秘钥: sk-d57c475a1d1646528002cdcf660e5112

```shell
cd docker

cp .env.example .env

docker compose up -d

# 关闭所有运行的 docker 容器
docker stop $(docker ps -q)

# 删除所有 docker 容器
docker rm $(docker ps -a -q)
```
