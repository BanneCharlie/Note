```shell
# 将系统中的 libc 库链接为所需的 musl 库（不推荐，但在某些情况下可以使用）解决最终问题
sudo ln -s 
/lib/x86_64-linux-gnu/libc.so.6 /lib/x86_64-linux-gnu/libc.musl-x86_64.so.1

# 后台启动 Xinference
XINFERENCE_MODEL_SRC=modelscope xinference-local --host 0.0.0.0 --port 9997
```

注意：因为我们的Dify是部署到了Docker中，而Xinference服务是在宿主机上，所以Dify是无法直接访问宿主机上的localhost的，需要通过：http://host.docker.internal:9997 访问



