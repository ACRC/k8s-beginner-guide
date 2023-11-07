Deployments define a set of replica pods, identical to eachother, and these are then spread out across the cluster, for both high availability and horizontal scaling performance.

We can create a deployment for an nginx pod with the following manifest.

`03_nginx-deployment.yaml`

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

`kubectl apply -f 03_nginx-deployment.yaml -n {namespace}`

and we can see that it works :


```
$ kubectl get deploy -n demo
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           83s


sk18181@dl-admin-jump:~/k8s$ kubectl get pod -n demo
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-cbdccf466-7cvbm   1/1     Running   0          85s
```

We can see that the deployment is deployed, and there is a single nginx pod running in this namespace.

But to make use of a deployment, we should have more than one pod, let's scale it up:

`kubectl scale deploy/nginx-deployment --replicas 3 -n {namespace}`

and now there are three:

```
$ kubectl get deploy -n demo
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           5m41s


$ kubectl get pod -n demo
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-cbdccf466-7cvbm   1/1     Running   0          5m57s
nginx-deployment-cbdccf466-9clqp   1/1     Running   0          105s
nginx-deployment-cbdccf466-hzzrz   1/1     Running   0          105s
```

and if you didn't change the labels, all three of these will now also be pointed to by the service you created earlier.