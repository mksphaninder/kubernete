# Kubernetes
- The structure of kubernetes deployments.

     pod  |   pod     pod  | pod
    ______________  ____________
    |             ||            | 
    |  Replica Set|| Replica Set|
    |_____________||____________|
     ___________________________
    |                           |
    |         Deployment        |
    |___________________________|
## Replicaset

- A replica set ensure that a specific number of pods are running at all times.
```kubectl get rs```
- ``kubectl get pods -o wide`` gives some more details about pods.
- ``kubectl delete pods 'podName'`` deletes the pods but if you notice the pod will be automatically created because replica set notices that there are not enough pods running.
- ``kubectl scale deployment 'pod-name' --replicas=3`` will scale the replicas.
- ``kubectl get replicaset`` will get the details of replicaset
- ``kubectl get events --sort-by=metadata.creationTimestamp`` will get the log.
- ``kubectl explain replicaset`` will get details of the command.


## Deployment
-  let's assume we have to update an application from V1 to V2, our aim is to have zero down time.
-  `kubectl set image deployment 'deployment name' 'name of the container'='new image:tag'` if the new image for some reason has error and cannot be deployed kubernetes will still run old container with the old image. There is almost zero-down time when a deployment is made.
- First it creates a new repica set and creates one instance.
- Replica set creates one pod.
    - Now one instance of new image is triggered.
    - Now slowly old replica set will be removed and simultaneously new pods are created.
    - This strategy is called rolling updates. 
- 