## 项目部署

```shell
#59服务器商城项目路径
cd applications/java/servers
#其他服务器商城项目都在root下（目前部署在59/210服务器）
#检查端口占用
lsof -i:41005
#nginx重启
./nginx -s reload
#关闭端口服务
kill -9 165706
#商城项目启动
setsid java -jar ehp-medical.jar


#101服务器oms项目路径
cd /home/oms
lsof -i:31005
#关闭端口服务
kill -9 165706
#oms项目启动
setsid java -jar platform-0.2.98.jar
setsid java -jar shop_round-0.0.1-SNAPSHOT.jar
setsid java -jar dkyj-0.0.1-SNAPSHOT.jar	//代客寄药
setsid java -jar wechatthepublic-0.0.10.jar	//附近门店
setsid java -jar thirdpartydocking-0.0.8-SNAPSHOT.jar //百度健康
#商城启动
setsid java -jar servers-1.2.10.jar



setsid java -jar service.oms-0.1.37.jar

setsid java -jar pricezhongtai2-0.0.1-SNAPSHOT.jar

setsid java -jar thirdpartydocking-0.0.9-SNAPSHOT.jar


7533
#新OMS
setsid java -jar platform-0.3.17.jar


cd /home/applications/mall
setsid java -jar servers-1.2.10.jar



#新商城
setsid java -jar dkyj-0.0.1-SNAPSHOT.jar



setsid java -jar shop_round-0.0.1-SNAPSHOT.jar


setsid java -jar ehp-gateway.jar



setsid java -jar
#101服务器微信公众号
cd /root/java
#查看服务
lsof -i:55005
#关闭端口服务
kill -9 xxxx
setsid java -jar service.oms-0.1.36.jar
setsid java -jar oms-0.1.27.jar

setsid java -jar ehp-gateway.jar


#启动公众号项目 启动之后不会关闭
setsid java -jar wechatthepublic-0.0.10.jar &
#启动之后 项目打印控制台
sudo java -jar wechatthepublic-0.0.10.jar &

nohup java -jar wechatthepublic-0.0.10.jar &

											把正确的弄到null    自定义日志错误信息
nohup java -jar wechatthepublic-0.0.10.jar >/dev/null 2>/root/java/logs/wechatthepublic/err.log & 



```

https://blog.csdn.net/qq_33656559/article/details/106549481



https://blog.csdn.net/qq_32265203/article/detai

```
make PREFIX=/root/applications/redis/redis-5.0.7 install
```

/root/applications/redis/redis-5.0.7

./bin/redis-server& ./redis.conf



./mongod -dbpath=/home/applications/mongodb/mongodb/data -logpath=/home/applications/mongodb/mongodb/mongodb.log -logappend -port=27017 -fork



```java
./mongod --config /home/applications/mongodb/mongodb/etc/mongodb.conf
```

GitLab地址 

用户名 jiaxiang     password  jiaxiang





