---
{"dg-publish":true,"permalink":"/CloudNative/kubernetes/k8s/","noteIcon":"3"}
---

![Pasted image 20231119163649.png|100%](/img/user/pics/Pasted%20image%2020231119163649.png)

#k8s
## network
> Pods in Kubernetes communicate with each other using their network. When pods are scheduled on different nodes (servers), Kubernetes takes care of networking to enable communication between them. Here's a simplified overview of how pods in different servers communicate:

1. **Cluster Networking:** Kubernetes provides a network overlay that allows communication between pods regardless of the nodes they are running on. This is achieved through a networking solution like Flannel, Calico, Weave, or others, which implement container networking interface (CNI) specifications.
    
2. **Pod IP Address:** Each pod in the cluster is assigned a unique IP address. This IP address is reachable from any other pod in the cluster, regardless of the node on which the pod is running.
    
3. **Service Discovery:** Pods can communicate with each other using their IP addresses or hostnames. Kubernetes provides a DNS service that allows pods to discover other pods by their service names.
    
4. **Kube-proxy:** Kube-proxy is a component of Kubernetes responsible for maintaining network rules on nodes. It helps in forwarding traffic to the appropriate pod regardless of its location in the cluster.
    
5. **Node-to-Node Communication:** If pods are running on different nodes, the communication involves the underlying networking infrastructure of the cluster. The overlay network ensures that packets are routed correctly between nodes, allowing pods to communicate seamlessly.
    
6. **Load Balancing:** In some cases, when a service exposes multiple pods, Kubernetes may set up load balancing to distribute incoming traffic across these pods. This load balancing is transparent to the communicating pods.
    

In summary, Kubernetes abstracts away the complexities of networking when it comes to communication between pods on different nodes. Pods can communicate with each other using standard networking principles, and the underlying Kubernetes networking solution ensures that the communication is seamless and transparent to the applications running in the pods.

k8s部署教程

### 1.安装kubectl，kubelet，kubeadm

### 2.安装CRI(take the containerd for example)


```sh title:"获取指定namespace的pods"
kubectl -n public-resource get pod

```





### 1.路径映射和设备映射
#mount

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["ls", "/dev/mem"]
    devices:
    - name: /dev/mem
      devicePath: /dev/mem

```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: busybox
    command: ["ls", "/mnt/data"]
    volumeMounts:
    - name: data
      mountPath: /mnt/data
  volumes:
  - name: data
    hostPath:
      path: /data
```



```sh
kubectl delete pod -n kube-system pod_name
kubectl delete -f a.yaml
kubectl apply -f a.yaml

# f means follow
kubectl logs -f -n namespace podname

kubectl describe -n namespace podname

kubectl describe svc -n namespace podname

kubectl describe pod -n namespace podname
kubectl describe node


kubectl get pod my-pod -n my-namespace -o yaml

# 查看当前node使用的container runtime， docker or containerd or ？
kubectl describe node node_name | grep Runtime


kubectl get namespaces
kubectl get ns

kubectl label nodes <node-name> <key>=<value>
kubectl get pod my-pod --show-labels
kubectl get node my-node --show-labels

kubectl auth can-i label nodes

```



```sh
# 获取所有的k8s pod状态
kubectl get pods --all-namespaces -o wide
kubectl get pods -A |grep wide


```