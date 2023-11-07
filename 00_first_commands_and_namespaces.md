The easiest way to interact with the kubernetes api is through the `kubectl` command.

To point kubect at the right cluster, you will need a 'kubeconfig' file under `$HOME/.kube/config` for the purposes of this tutorial, I've placed it there for you on dl-admin-gateway. 

At this point, it's probably a good idea to check it's working by running a simple command like `kubectl show no` to get a list of the nodes in the k8s cluster.

If you see this, then you're good to go:

```
~$ kubectl get no
NAME               STATUS   ROLES    AGE   VERSION
k8s-controller-2   Ready    <none>   20h   v1.27.7
k8s-controller-1   Ready    <none>   20h   v1.27.7
k8s-controller-3   Ready    <none>   20h   v1.27.7
```


Almost all kubernetes operations work within an isolated namespace, specified along with the command you're trying to run.


From this point on you should create and work in your own namespace to avoid conflicts with other users on the cluster:

`kubectl create ns demo`

```
$ kubectl get pod -n demo
No resources found in demo namespace.
```