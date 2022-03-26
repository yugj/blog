# maven

## 常用命令
### 批量设置版本号
进到root目录

1， 设置新的版本号

mvn versions:set -DnewVersion=1.1.3

2，当新版本号设置不正确时可以撤销新版本号的设置

mvn versions:revert

3，确认新版本号无误后提交新版本号的设置

mvn versions:commit