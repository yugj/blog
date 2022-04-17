# 路径

## 获取当前脚本执行路径
```shell script
echo "$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"

```