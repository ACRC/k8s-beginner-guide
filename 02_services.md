You say you want to actually access the webserver running in your pod?

Then you need to create a service that maps to it.

`02_nginx-service.yaml`

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
```

Apply with `kubectl apply -n {namespace} -f 02_nginx-service.yaml`

Instead of applying a manifest, creating services can be done a quick and dirty way with `kubectl expose -n demo pod/nginx --selector name=nginx --type=ClusterIP`

A selector condition is required in order to point the service at the right pods, you can create labels in the pod definition for this.

You will have noticed that this service has type ClusterIP, which means it is allocated an ip which is internal to the K8s cluster, and cannot be accessed externally. 

Other types include Nodeport, which allocates a random network port on every k8s worker node and listens on that, and LoadBalancer, which can assign virtual ips to services, which must be routed back to the L2 network which the k8s workers sit on. 

In this case, none of these types are particularly useful to us, as the K8s workers sit on a private network within an openstack project. We'll look at useful external access later on in the ingress secion.

One way we can connect to our services currently is by port-forwarding to the host you're on with kubectl (of course you will need each need to use a different local port on dl-admin-gateway):

`kubectl port-forward -n {namespace} service/nginx-service {localport}:{containerport}`

`kubectl port-forward -n {namespace} service/nginx-service 8080:80`

and then check it works in a different terminal session:
```
$ curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

Success

Services can act as basic load balancer, spreading load across all the pods which match the selector conditions.

Service names are also dns entries, that can be used by other services running in the namespace. 

For example if you were hosting a rest api, which relies on a Redis database, instead of looking up the ip, and passing that through to your pod, you could make a service called `redis`, and use a config which connects to `redis:6379` which will then be routed to the correct pod(s) through the service.