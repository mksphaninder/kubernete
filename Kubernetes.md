# Kubernetes

## Kubernetes Architecture.
   
    ```
      Master Node | Worker Node
      Manage cluster| Run your Application
     ____________________________
    |                            |
    |           Cluster          |
    |____________________________|


    API    | Distribute | Scheduler      | Control Manager
    Server |    DB      | Kube-Scheduler | Kube Control Manager
    ____________________________________________________________
    |                                                           |
    |                       Master Node                         |
    |___________________________________________________________|
    ```
- All configuration changes, scaling and details are stored in the distributed DB (etcd).
- Typically we have 3 states of the DB so that it is not lost.
- API Server
    - kubectl communication with cluster is handled.
    - Also, Google cloud console communication with cluster is handled.
- Scheduler
    - Responsible for scheduling pods on nodes.
- Controller Manager
    - Maintains the health of the cluster.
    - It makes sure that the state of the application is same as desired state.

- Worker Node
```
Node Agent | Networking Component | Container Runtime   | PODS
Kubelet    |    Kube-proxy        | (CRI-docker,rkt etc)| Multiple pods running container
_________________________________________________________________________________________
|                                                                                       |
|                                       Worker Node                                     |
|_______________________________________________________________________________________|
```
- Applications run inside pods.
- Kubelet
    - Makes sure it monitors what is happening in the pod and reports it to the master node.
- kube-proxy
    - Helps in exposing services around the nodes and pods.
- Container Runtime
    - Run containers inside of the pods and these need container run-time.
    - Usually Docker is used.
- Does Master node run applications?
    No, Master node only has configuration.
- Can you only run Docker containers?
    No, it can OCI container
- What happens when a master node is down?
    No, it won't go down.

- The structure of kubernetes deployments.
    ```
     pod  |   pod     pod  | pod
    ______________  ____________
    |             ||            | 
    |  Replica Set|| Replica Set|
    |_____________||____________|
     ___________________________
    |                           |
    |         Deployment        |
    |___________________________|
    ```

## Pod 
- In Kubernetes a pod is a throw away unit.
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
- Whenever a pod is deleted (`kubectl delete pod 'pod name'`) everytime a new IP address is assigned.

## Service.
- The role of a service to provide an always available interface to the application which are running inside the pods.
- Service is created when we expose the deployment. (`kubectl expose deployment 'deployment name' --type=LoadBalancer --port='port number'`)
- `kubectl get services` lists the services running in our kubernetes environment.
- Cluster IP service can only be acessed from inside the cluster

## Regions and Zones
- Google cloud has regions across the globe, the actual reason for having regions is latency.
- Availability, we can distribute the applications across multiple regions.
- Legal requirements.
- Even within same regions we have zones, they are physically isolated data centers.

## Running gcloud console locally.
- We need gcloud and kubectl cli tools.
- Gcloud cli
    - `gcloud init` for login
    - `gcloud auth login` to change login at a later point.
- Checking release history
    - `kubectl rollout history deployment 'deployment name'`
    - To record history of deployment `kubectl set image deployment hello-world hello-world=in28min/hello-world-rest-api:0.0.3.RELEASE --record=true`
    - Rolling back to a previous release `kubectl rollout undo deployment hello-world --to-revision=1`

## Storing configs of Kubernetes into YAML.
- `kubectl get service 'service name' -o yaml > 'filename'` this will create a config file and all the config is stored in it.
- We can make all the changes required in this file and save it. To apply the changes `kubectl apply -f 'name of the file'`.
- The changes will be effected immediately.

## Creating a new kubectl deployment using the config file.
- First lets delete the existing deployment so, t do that first we need to know the name of the deployment usually the name is stored inside the `app` variable
    - `kubectl get all -o wide | grep app=` here in this case the label is hello-world
- To delete the existing deployment `kubectl delete all -l app=hello-world`
    - This will delete all the existing pods, services, deployment, replicaset.
- Now to apply the config from deployment.yaml use `kubectl apply -f 'name of the file'`.
- To see the exposed ports use `kubectl get svc --watch` `svc` is short for service.

## Structure for deployment and service config files.
- Both have the similar structure.
    - `apiVersion`
        - says the version we are using.
    - `kind`
        - The kind of deployment(service, deployment etc).
    - `metadata`
        - describes the name of the deployment, namespace & labels used.
    - `spec`
        ```YAML
        spec:
            containers:
                - image: in28min/hello-world-rest-api:0.0.1.RELEASE
                imagePullPolicy: IfNotPresent
                name: hello-world-rest-api
                resources: {}
                terminationMessagePath: /dev/termination-log
                terminationMessagePolicy: File
            dnsPolicy: ClusterFirst
            restartPolicy: Always
            schedulerName: default-scheduler
            securityContext: {}
            terminationGracePeriodSeconds: 30
        ```
        - Describes the pods, replicaset etc.
        - Inside spec we are defining the pod and how it needs to be mapped with the deployment.
        - The important part of the definition of the pod is `spec`(inside of spec).
            - A Pod can contain multiple containers.
            - We can define multiple containers here. using '-' which denotes an array.
            - `template` contains:-
                - `imagePullPolicy`: `ifNotPresent` will pull the image only if the image is absent locally.
                - `name` is the name of the container.
                - `restart policy` if there is a failure in starting of the container then try restarting again and again.
                - `terminationGracePeriodSeconds` Here we are saying give the pod a chance to terminate for 30s.
                - `metadata` inside the metadata the labels of the pods are defined
            - `selector` defined how a pod is mapped to a deployment.
                - `matchLabels` is the selector name we are using for mapping.
                - `replicas` defines No. of Pods
                - `strategy` defined the update strategy.
                - `surge` defines the how the update should take place and how many pods can be unavailable stuff like that.
                - In a service we define the selector to a pod.
                    ```YAML
                    spec:
                        clusterIP: 10.32.14.46
                        externalTrafficPolicy: Cluster
                        ports:
                            - nodePort: 30836
                            port: 8080
                            protocol: TCP
                            targetPort: 8080
                        selector:
                            app: hello-world
                        sessionAffinity: None
                        type: LoadBalancer
                    ```
                    - The important thing to remember is that the service directly maps to a pod, so the `selector` is used to map with the pod.
                    - `ports` define the ports.

