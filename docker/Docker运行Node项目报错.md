## Docker运行Node项目报错

Dockerfile如下：

![](http://img.fosuchao.com/20200308211952.png)

运行docker

```dcoker
docker image build -t bulletin-board-app:1.0 .
```

![](http://img.fosuchao.com/20200308211920.png)

执行的过程中报错。

解决办法：

编辑或者创建文件 `/etc/docker/daemon.json`,并且输入

```
{
    "dns": ["10.0.0.2", "8.8.8.8"]
}
```

保存后重启docker

```bash
systemctl restart docker
```

问题解决

![](http://img.fosuchao.com/20200308212301.png)