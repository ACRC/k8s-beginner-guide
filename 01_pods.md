Pods are the smallest schedulable work unit in Kubernetes, and are the basic building blocks of any complex system you can think of. 

A pod defines one or more container images, to be scheduled and run together on a worker node, along with any network ports that they might expose.

Here's an example of a 'manifest' from the kubernetes docs for an pod running an nginx container.

`01_nginx-pod.yaml`:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

You can run this on the cluster with the command `kubectl apply -n {namespace} -f nginx-pod.yaml`

And after a few seconds you will see it running with:

```
$ kubectl get pod -n demo
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          7s
```

You can get more details about the pod with `kubectl describe -n {namespace} pod/nginx` such as which host it's running on, internal ips, etc.