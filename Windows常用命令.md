# 查看端口占用情况

## 根据端口查看占用端口的进程id

```shell
netstat -aon | findstr :8001
```

执行案例



![image-20221228100821658](https://s2.loli.net/2022/12/28/7F9mTxALPrNKUqO.png)

## 根据pid查看服务名称

```shell
tasklist|findstr "13460"
```

执行案例

![image-20221228100903446](https://s2.loli.net/2022/12/28/9OLxWjGhVmJZYMI.png)

最后可以根据服务名称在服务中停止对应服务释放对应端口



























