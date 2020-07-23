# 配置Go开发环境

这篇文章的环境是在`windows10`下

## 安装

先进入Go官网进行下载。[进入下载页面](https://golang.org/dl)

可以查看Go的官方文档，[查看](https://golang.org/doc/)

### 配置环境变量

安装好后，可以在命令行(`cmd`或`powerShell`)中查看Go的命令和状态。

**查看版本**

```
go version
```

**查看环境变量**

```
go env
```

go的环境变量中，有几个比较重要的变量

- **GOROOT**：Go的安装目录
- **GOPATH**：自定义的Go项目目录
- **GOBIN**：通常配置为%GOPATH%/bin

#### 配置系统变量

我们要在系统变量的**PATH**中配置**Go的bin目录**和**Git的bin目录**，配置Git是因为go的很多包和插件都是在github上面的，通常需要拉取下来才能使用，所以需要用到Git。

![](http://img.fosuchao.com/20200428102551.png)

#### 配置用户变量

用户变量中配置上述的GOROOT，GOPATH，GOBIN，按自己的文件目录替换上去即可。

![](http://img.fosuchao.com/20200428102743.png)

配置完成后，再在命令行中输入`go env`即可看见最新配置的Go环境变量。可能需要重启机器才能及时更新。

## 开发工具

Go有很多开发工具，没有统一的说法，看个人喜好和习惯进行安装即可，这里介绍我本人试过的开发工具，分别有三个。

- Sublime Text + Gobuilder/GoSublime
- VsCode + GoTools
- GoLand

另外还有Vim等等开发工具，有很多，通常都是要自己安装一些插件后才能更好的使用。

前两种我配置后，因为有些插件有些问题，代码自动补全的功能都有问题，不知道大家的正不正常。

比较麻烦，我最后选择了GoLand，虽然说比较重型，不过毕竟是jetbrains出品，各种功能都很完善，也没有那么多麻烦的配置。后面会贴上破解的方法。

### Sublime Text + Gobuilder/GoSublime

#### 安装插件GoSublime/Gobuilder

Preferences ->Package Control -> Install Packages，然后等待弹出新的输入框

这里**Gobuilder**是可以安装的，安装好后在User-setting中进行配置GOROOT和GOPATH即可。

**GoSublime**可能会找不到，需要直接到github上下载源代码，下载好后点击

Preferences -> Browse packages打开插件目录。直接将下载好的文件**解压后复制过去**即可。

安装好GoSublime后，可能会报错（没报错就完美了），原因是缺少一些东西，不过我按照网上的操作一顿捣鼓还是没用，大家可以在网上搜搜试试能不能解决这问题，我就放弃了。

```
GoSublime error: MarGo build failed
|                > This is possibly a bug or miss-configuration of your environment.
|                > For more help, please file an issue with the following build output
....
....
```

上面的问题只会影响到代码补全的功能，代码还是可以正常编译和运行的。

比如运行代码

```go
package main

import "fmt"

func main() {
	fmt.Println("hello")
}

```

![](http://img.fosuchao.com/20200428104640.png)

### VsCode + GoTools

vscode确实很强大，各种语言都可以开发，只需要安装上需要的插件即可。不过我还是不习惯用。

在vscode的扩展商店搜索go，然后安装即可。

![](http://img.fosuchao.com/20200428104844.png)

然后想要正常使用就需要安装16个左右的插件才行。

`ctrl + shift + p`弹出搜索框，输入go查找**Install/Update Tools**，然后点击

![](http://img.fosuchao.com/20200428105124.png)

点击后会有如下页面

![](http://img.fosuchao.com/20200428105212.png)

这个列表中的就是Go的所有插件，直接全选安装，不多比比。

点了安装后，会发现，基本都是安装失败。

![](http://img.fosuchao.com/20200428105507.png)

遇到这种情况，还有另外的安装方法

- 设置代理
- 通过go get安装
- 直接下载需要的exe文件复制到%GOPATH%/bin目录

**设置代理**

- 在vscode中设置代理，[详情参考](https://goproxy.io/zh/)

  打开编辑setting，添加以下配置

  ![](http://img.fosuchao.com/20200428110140.png)

- 在系统环境变量中配置GOPROXY

  ![](http://img.fosuchao.com/20200428110308.png)

配置好代理后，再去安装就不会出现上面的问题了。

**go get安装**

下面的列表可能不全，可以直接再github上查找相应的链接，再用这种方法进行安装即可。安装好后，GOPATH的bin目录就会多很多插件的exe文件。

```git
go get -u -v github.com/bytbox/golint
go get -u -v github.com/golang/tools
go get -u -v github.com/lukehoban/go-outline
go get -u -v github.com/newhook/go-symbols
go get -u -v github.com/josharian/impl
go get -u -v github.com/sqs/goreturns
go get -u -v github.com/cweill/gotests
....
```

**exe文件复制**

这个方法其实就是跳过了安装的步骤，将所有exe文件直接复制过去即可，这里没有连接，需要自己去找。

安装好的所有插件后，打开编辑setting.json进行配置。

覆盖配置如下：

```json
"go.buildOnSave": "workspace",
"go.lintOnSave": "package",
"go.vetOnSave": "package",
"go.buildTags": "",
"go.buildFlags": [],
"go.lintFlags": [],
"go.vetFlags": [],
"go.coverOnSave": false,
"go.useCodeSnippetsOnFunctionSuggest": false,
"go.formatOnSave": true,
"go.formatTool": "goreturns",
"go.goroot": "你的GOROOT",
"go.gopath": "你的GOPATH",
"go.gocodeAutoBuild": false
```

配置好就，就可以进行愉快的开发了。

### GoLand

[详情参考](https://shimo.im/docs/dKYCkd8PrX3ckX99/read)