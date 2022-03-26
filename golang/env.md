# 开发环境

1，安装go sdk 根据项目情况安装，目前使用1.13版本

```
brew install go@1.13

```

2，IDEA 安装GO插件及配置

安装：Plugins 中输入go选择go插件安装

配置：【Languages & Frameworks】【Go】【Go Modules(vgo)】

-   勾选Enable Go Modules(vgo) integration
-   Proxy填入：[https://goproxy.cn](https://goproxy.cn/),direct
-   不勾选 Enable vendoring support automatically

3，Go环境变量配置，目前使用go mod管理

.bash_profile添加如下，修改完后source下

```
#go setting
export GO111MODULE=on
export GOPROXY=https://goproxy.cn,direct
export GOSUMDB=off

```

4，Gitlab上传私钥

步骤：

【头像】【管理账户】【SSH keys】【Add key】

贴入本地ssh key文本

PS：本地ssh key位于~/.ssh/id_rsa.pub

5，查看go环境变量

```
go env

```

补充说明：仓库走代理方式，找不到的包将回源到公司内部仓库，网络层面配置了代理到真实内部仓库  
golang 版本基于git tag管理，认证方式基于git认证方式