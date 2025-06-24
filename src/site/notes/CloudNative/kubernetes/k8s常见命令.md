---
{"dg-publish":true,"permalink":"/CloudNative/kubernetes/k8s常见命令/","noteIcon":"3"}
---

1. get pods



```sh
kubectl get pods -n mindie
kubectl get nodes -n mindie
kubectl get pods -l key=value
kubectl label node key=vale

#查看上一次重启的日志和原因
kubectl logs <pod-name> -n <namespace> --previous
#多容器pod
kubectl logs <pod-name> -n <namespace> -c <container-name> --previous
kubectl top pod <pod-name> -n <namespace>
#查看事件
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
# 给节点上taint，不允许pod调度到该node，除非pod能tolerate这个taint
kubectl taint nodes node01 node-role.kubernetes.io/unschedulable=:NoSchedule
# 解除taint
kubectl taint nodes <node-name> node-role.kubernetes.io/unschedulable-
# 将节点设置为不可调度，阻止新pod调度上来，不妨碍已有pod
kubectl cordon <node-name>
# 驱逐当前node上已有的pod到其他nodes上
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
# 取消cordon
kubectl uncordon <node-name>



```


查看有taint或者cordon的节点
```sh
# 所有节点的 cordon / taint 状态
for node in $(kubectl get nodes -o name); do
  echo "=== $node ==="
  kubectl describe $node | grep -E "Taints|Unschedulable"
done

```