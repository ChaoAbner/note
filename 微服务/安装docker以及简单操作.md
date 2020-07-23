## ubuntu安装docker后出现错误：

### 权限错误

#### config.json权限错误

WARNING: Error loading config file: /home/ubuntu/.docker/config.json: stat /home/ubuntu/.docker/config.json: permission denied

表示配置文件没有权限读取，要用超级管理员权限

可以为该配置文件添加相关权限

```bash
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "/home/$USER/.docker" -R
```

#### docker.sock权限错误

 docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.39/containers/create: dial unix /var/run/docker.sock: connect: permission denied.

解决办法：

```bash
sudo chomd 666 /var/run/docker.sock
```





### 安装软件

#### 安装java jdk

https://blog.csdn.net/zbj18314469395/article/details/86064849

#### 安装maven

https://blog.csdn.net/cnmilan/article/details/78762499

#### 部署springboot项目

https://segmentfault.com/a/1190000018280290