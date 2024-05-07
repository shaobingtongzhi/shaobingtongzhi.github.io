

```sh
# 拉取镜像
docker pull jenkins/jenkins:lts-jdk8

# 运行容器
docker run --name jenkins -p 8080:8080 -p 50000:50000 -v E:/docker/jenkins/jenkins_home:/var/jenkins_home -d jenkins/jenkins:latest

# 查看初始密码
docker logs jenkins
```


127.0.0.1:8080/job/%E6%99%BA%E5%8D%9A%E7%83%AD%E7%94%B5/build?token=zhiboredianupdate
48HP8-DN98B-MYWDG-T2DCC-8W83P