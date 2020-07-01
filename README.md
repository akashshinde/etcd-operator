## Etcd Demo Operator

The objective of this Operator is to demonstrate EtcdCluster kind of resource implementation using Kubernetes controller pattern- [Operator](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/). Another objective of this repository is to show how to build the custom controller that encapsulates specific domain/application level knowledge. The Operator is built by using the [operator-sdk framework](https://github.com/operator-framework/operator-sdk)

### Prerequisites

- golang v1.12+.
- set GO111MODULE="on"
- [Install the operator-sdk](https://sdk.operatorframework.io/docs/golang/installation/)
- [crc](https://code-ready.github.io/crc/) or [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)
- [kubectl client](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

### EtcdCluster Resource

Once user applies the EtcdCluster resorce with the name `example-etcdcluster` (kubectl apply -f ./deploy/crds/demo.redhat.com_v1alpha1_etcdcluster_cr.yaml) resource, controller could spin up number of pods mentioned as per the `spec.size` field and with the specified verison number taken from `spec.version`.

e.g. User want to spin up 3 pods with version 3.4.9

```yaml
apiVersion: demo.redhat.com/v1alpha1
kind: EtcdCluster
metadata:
  name: example-etcdcluster
spec:
  size: 3
  version: 3.4.9
  ```

### Trying out the Operator

`git clone https://github.com/akashshinde/etcd-operator.git` 

You would get the etcd-operator directory.
`cd etcd-operator`

We would be trying out the operator locally. By locally we mean that we want to run the operatot logic binary without actually building an image and pushing it to a container registry. Running the operator locally helps in day to day development. You can have a [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) or a [crc](https://code-ready.github.io/crc/) single node local cluster to play with the operator.

Once you have started your local operator, apply the following files

`kubectl apply -f deploy/crds/demo.redhat.com_etcdclusters_crd.yaml`
`kubectl apply -f deploy/role.yaml`
`kubectl apply -f deploy/role_binding.yaml`
`kubectl apply -f deployservice_account.yaml`

The CRD would be registered and yo can check that as well by following the command:

```
avni@localhost etcd-operator (master)=>>   kubectl get crd | grep etcdcluster
etcdclusters.demo.redhat.com
```

After that run the operator locally with `operator-sdk run --local`
You would get the following logs

```
INFO[0000] Running the operator locally in namespace default. 
{"level":"info","ts":1593611264.944664,"logger":"cmd","msg":"Operator Version: 0.0.1"}
{"level":"info","ts":1593611264.9446914,"logger":"cmd","msg":"Go Version: go1.14.3"}
{"level":"info","ts":1593611264.9447021,"logger":"cmd","msg":"Go OS/Arch: linux/amd64"}
{"level":"info","ts":1593611264.9447162,"logger":"cmd","msg":"Version of operator-sdk: v0.17.1"}
{"level":"info","ts":1593611264.9482136,"logger":"leader","msg":"Trying to become the leader."}
{"level":"info","ts":1593611264.9482288,"logger":"leader","msg":"Skipping leader election; not running in a cluster."}
{"level":"info","ts":1593611268.9914746,"logger":"controller-runtime.metrics","msg":"metrics server is starting to listen","addr":"0.0.0.0:8383"}
{"level":"info","ts":1593611268.9921577,"logger":"cmd","msg":"Registering Components."}
{"level":"info","ts":1593611268.9923682,"logger":"cmd","msg":"Skipping CR metrics server creation; not running in a cluster."}
{"level":"info","ts":1593611268.992389,"logger":"cmd","msg":"Starting the Cmd."}
{"level":"info","ts":1593611268.992808,"logger":"controller-runtime.manager","msg":"starting metrics server","path":"/metrics"}
{"level":"info","ts":1593611268.9928758,"logger":"controller-runtime.controller","msg":"Starting EventSource","controller":"etcdcluster-controller","source":"kind source: /, Kind="}
{"level":"info","ts":1593611269.2937565,"logger":"controller-runtime.controller","msg":"Starting Controller","controller":"etcdcluster-controller"}
{"level":"info","ts":1593611269.2938137,"logger":"controller-runtime.controller","msg":"Starting workers","controller":"etcdcluster-controller","worker count":1}
```

This indicates that our operator is up and running.
To trigger an event go ahead and apply the Custom Resource yaml: `kubectl apply -f deploy/crds/demo.redhat.com_v1alpha1_etcdcluster_cr.yaml`

The apllying wold trigger an event since our controller is watching on it and now the operator logs would get

```
{"level":"info","ts":1593611448.3831322,"logger":"controller_etcdcluster","msg":"Reconciling EtcdCluster","Request.Namespace":"default","Request.Name":"example-etcdcluster"}
{"level":"info","ts":1593611449.9101553,"logger":"controller_etcdcluster","msg":"Reconciling EtcdCluster","Request.Namespace":"default","Request.Name":"example-etcdcluster"}
```

Now if you see `kubectl get pods`

```
avni@localhost etcd-operator (master)=>>   kubectl get pods
NAME                                   READY   STATUS              RESTARTS   AGE
example-etcdcluster-567c6cf775-4w6sv   0/1     ContainerCreating   0          3s
example-etcdcluster-567c6cf775-8vdcb   0/1     ContainerCreating   0          3s
example-etcdcluster-567c6cf775-xtkqx   0/1     ContainerCreating   0          3s
avni@localhost etcd-operator (master)=>>   kubectl get pods
NAME                                   READY   STATUS              RESTARTS   AGE
example-etcdcluster-567c6cf775-4w6sv   0/1     ContainerCreating   0          7s
example-etcdcluster-567c6cf775-8vdcb   0/1     ContainerCreating   0          7s
example-etcdcluster-567c6cf775-xtkqx   1/1     Running             0          7s
avni@localhost etcd-operator (master)=>>   kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
example-etcdcluster-567c6cf775-4w6sv   1/1     Running   0          44s
example-etcdcluster-567c6cf775-8vdcb   1/1     Running   0          44s
example-etcdcluster-567c6cf775-xtkqx   1/1     Running   0          44s
```

Check the EtcdClster instance

```
avni@localhost etcd-operator (master)=>>   kubectl get etcdcluster 
NAME                  AGE
example-etcdcluster   2m40s
```

Check the EtcdCluster resource

```
avni@localhost etcd-operator (master)=>>   kubectl get etcdcluster example-etcdcluster -o yaml
apiVersion: demo.redhat.com/v1alpha1
kind: EtcdCluster
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"demo.redhat.com/v1alpha1","kind":"EtcdCluster","metadata":{"annotations":{},"name":"example-etcdcluster","namespace":"default"},"spec":{"size":3,"version":"3.4.9"}}
  creationTimestamp: "2020-07-01T13:50:48Z"
  generation: 1
  name: example-etcdcluster
  namespace: default
  resourceVersion: "30531"
  selfLink: /apis/demo.redhat.com/v1alpha1/namespaces/default/etcdclusters/example-etcdcluster
  uid: c6906595-d3ec-405e-961d-7e3317b50428
spec:
  size: 3
  version: 3.4.9
status:
  message: Cluster created with etcd version 3.4.9
```

You will see that the resource's status has been set by the controller.
Now let's edit the example-etcdcluster resource

```yaml
apiVersion: demo.redhat.com/v1alpha1
kind: EtcdCluster
metadata:
  name: example-etcdcluster
spec:
  size: 5
  version: 3.4.0
  ```

  Now the status of the CR-example-etcdcluster  would change and more pods would be spun up

```
  avni@localhost etcd-operator (master)=>>   kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
example-etcdcluster-656ff54bbb-4fvx7   1/1     Running   0          24s
example-etcdcluster-656ff54bbb-5llz7   1/1     Running   0          16s
example-etcdcluster-656ff54bbb-98x9t   1/1     Running   0          17s
example-etcdcluster-656ff54bbb-n8xd5   1/1     Running   0          24s
example-etcdcluster-656ff54bbb-v5jq2   1/1     Running   0          24s
```

You can check the image in the pod and it's version would have been updated too
`kubectl  get pods example-etcdcluster-656ff54bbb-4fvx7 -o yaml`


### Questions/Feedback

Please feel free to open up an issue for questions/feedbacks/discussions.
Or you can reach out to us and drop a mail at avsharma@redhat.com/akshinde@redhat.com
