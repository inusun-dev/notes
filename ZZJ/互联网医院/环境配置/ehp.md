# 开发环境

## Jenkins

Jenkins：开发环境Jenkins

```shell
192.168.59.14
root/root
```

```shell
# 启动语句：
nohup java -DJENKINS_HOME=/var/lib/jenkins -jar /usr/local/src/jenkins-2.319.3-lts.war --logfile=/var/log/jenkins/jenkins.log --httpPort=8080 > /dev/null 2>&1 &

# Jenkins访问地址和账号密码
地址：192.168.59.14:8080
账号：admin
密码：jenkins

```

