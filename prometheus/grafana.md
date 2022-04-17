# grafana 配置

## 参数支持二级联动
```
label_values(dubbo_request_count_total,application)
label_values(dubbo_request_count_total{application="$application"},method)

ps:
Include All option 选型支持查询所有
配合=~在选择All的时候忽略查询条件
```