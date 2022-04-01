# 清理golang包

## 包位置
```
$GOPATH/pkg/<architecture>
$GOPATH/pkg/mod
$GOPATH/pkg/mod/cache 缓存目录

```

## 内部调试包清理
```shell
删除如下位置包文件
$GOPATH/pkg/<architecture>
$GOPATH/pkg/mod
$GOPATH/pkg/mod/cache 缓存目录

删除go.sum对应记录

执行go mod tidy

若还不行，直接git clone到本地gopath
```

## 相关文章
https://stackoverflow.com/questions/13792254/removing-packages-installed-with-go-get