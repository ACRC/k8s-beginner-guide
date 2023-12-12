There are many ways to provide external access to services running in the k8s cluster through services. 

However the cleanest way to provide web/http based services to end users is through an ingress controller in k8s.

Ingress controllers run a reverse proxy, listening on a service (usually a virtual floating ip load balancer) and accept incoming web traffic. The cluster uses ingress objects, with rules defined for where to route traffic once it comes into the cluster, based on host headers.

Ingress controllers are defined in kubernetes but it does not implement them itself, they are installed as plugins based on your specific requirements, in this case, I've pre-installed an nginx ingress controller, which will pick up ingress objects from k8s the same way any other ingress controller would. Some alternatives are Traefik, 


This allows you to easily add tls certificates too, since the certs for all of your services can be attached using the one ingress controller.

Use the following to create an nginx pod, a service which points network traffic to that pod, and then an ingress rule:


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

---

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

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  tls:
  - hosts:
      - test.k8s.acrc.bris.ac.uk
    secretName: k8s.acrc.bris.ac.uk
  rules:
  - host: test.k8s.acrc.bris.ac.uk
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: nginx-service
            port:
              number: 80

```


The domain name *.k8s.acrc.bris.ac.uk would need to be pointed to the ingress controller floating ip