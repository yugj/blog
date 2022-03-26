# 常用命令

## pod查询
1，模糊查询K8S pod列表， -o wide 展示IP信息等

kubectl get pod|grep {your grep value}

2，查询K8S pod详情

kubectl describe pod {pod name}

3 , 查询k8s单个pod日志

kubectl logs --tail 200 -f {pod name}

4，查询k8s多个pod日志

kubectl logs -f -l app={api} --all-containers

ps：api 为详情里面app这个lables值