

```sh
# 拉取镜像
docker pull jenkins/jenkins:lts-jdk8

# 运行容器
docker run --name jenkins -p 8080:8080 -p 50000:50000 -v E:/docker/jenkins/jenkins_home:/var/jenkins_home -d jenkins/jenkins:lts-jdk8

# 查看初始密码
docker logs jenkins
```	