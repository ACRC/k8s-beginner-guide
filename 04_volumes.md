So, what if we need containers to remember things?

Luckily I have attached it to the openstack cinder api, so we can attach openstack volumes directly to pods.

We can see the available storage classes for volumes with `kubectl get sc` no need for a namespace this time. You can have multiple storage classes for different storage backends or levels of performance, a hdd volume might be cheaper than an ssd volume for example.


```
$ kubectl get sc
NAME                            PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
csi-sc-cinderplugin (default)   cinder.csi.openstack.org   Delete          Immediate           false                  21h
```

This storage class enables dynamic provisioning, meaning, you ask for an amount of storage, and it'll ensure the storage backend gives you what you ask for, and attaches the block storage device to the right place.

To request storage from the storage class you'll need to make a persistent volume claim. I've included the pvc and a new nginx pod in the same manifest:


`04_nginx-pvc.yaml`:

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi

---

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pvc
spec:
  volumes:
    - name: nginx-pvc
      persistentVolumeClaim:
        claimName: nginx-pvc
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
    volumeMounts:
    - mountPath: "/mnt/test"
      name: nginx-pvc

```

apply this:

`kubectl apply -n {namespace} -f 04_nginx-pvc.yaml`

and you'll see a pod come up with a volume attached:

`kubectl describe -n {namespace} pod/nginx-pvc-pod`

If you were to also go into the horizon dashboard for the K8S project, you'll see a new volume created too.


We can see that the volume claim has been created:

```
$kubectl get pvc
NAME                                       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nginx-pvc                                  Bound    pvc-4ce794f1-0177-4720-ae63-6cb73ca35db6   1Gi        RWO            csi-rbd-sc     23s
```

and the volume has been provisioned by ceph based on the spec in the pvc:

```

PS C:\Users\Ethan W\k8s> kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                              STORAGECLASS   REASON   AGE
pvc-4ce794f1-0177-4720-ae63-6cb73ca35db6   1Gi        RWO            Delete           Bound    default/nginx-pvc                                  csi-rbd-sc              103s

```

The access mode is RWO (read,write,once) meaning that the volume can only be mounted to one pod at a time as this storage class provides block storage. You can have different volume types with different access modes, for example an nfs/cephfs volume type which will give a RWX (read,write,many) so that a filesystem can be shared among many pods.

You can check that the volume is mounted correctly by attaching a shell to the pod the volume is mounted in:



```
$ kubectl exec -it pod/nginx-pvc -- bash
root@nginx-pvc:/# df -h
Filesystem                         Size  Used Avail Use% Mounted on
overlay                            442G   18G  402G   5% /
/dev/rbd2                          974M   24K  958M   1% /mnt/test
...

```