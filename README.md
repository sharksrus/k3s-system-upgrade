# k3s-system-upgrade

## Info

* https://rancher.com/blog/2020/upgrade-k3s-kubernetes-cluster-system-upgrade-controller

This is using Rancher's `System Upgrade Controller`. I simply folled the doc above, but made some tweaks to the files, essentially iamge and k3s versions.


## Instructions

Apply this label only to the primary node.
``` sh
kubectl label node pri1 node-role.kubernetes.io/master=true
```

Apply the following label to all nodes (exclude primary if it is not running workloads)

```sh
kubectl label node pri1.bar.punko.ps worker1 worker2 worker3 worker4 worker5 k3s-upgrade=true
```

Once you have the labels applied, you can apply the 2 yaml files in the repo.

``` sh
kubectl apply -f crd.yaml

namespace/system-upgrade created
serviceaccount/system-upgrade created
clusterrolebinding.rbac.authorization.k8s.io/system-upgrade created
configmap/default-controller-env created
deployment.apps/system-upgrade-controller created

```

``` sh
kubectl apply -f k3s-upgrade.yaml

plan.upgrade.cattle.io/server-plan created
plan.upgrade.cattle.io/agent-plan created

```

you can then check on things with the following:

``` sh
kubectl get all -n system-upgrade

NAME                                                                  READY   STATUS      RESTARTS   AGE
pod/apply-agent-plan-on-worker4-with-57eb7803114-q2xv2   0/1     Completed   0          16m
pod/apply-agent-plan-on-worker5-with-57eb7803114-rfnvw   0/1     Pending     0          13m
pod/apply-server-plan-on-pri1-with-57eb780311464-f2xpm   0/1     Completed   0          16m
pod/system-upgrade-controller-5c49c8b897-bzq2s                        1/1     Running     0          16m

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/system-upgrade-controller   1/1     1            1           16m

NAME                                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/system-upgrade-controller-5c49c8b897   1         1         1       16m

NAME                                                                        COMPLETIONS   DURATION   AGE
job.batch/apply-agent-plan-on-worker4-with-57eb7803114-1b53b   1/1           2m50s      16m
job.batch/apply-agent-plan-on-worker5-with-57eb7803114-5d645   0/1           13m        13m
job.batch/apply-server-plan-on-pri1-with-57eb780311464-9792c   1/1           115s       16m

```

You can check on the job status with the following:

```sh 
kubectl get jobs -n system-upgrade

NAME                                                              COMPLETIONS   DURATION   AGE
apply-agent-plan-on-worker4-with-57eb7803114-1b53b   1/1           2m50s      17m
apply-agent-plan-on-worker5-with-57eb7803114-5d645   0/1           14m        14m
```

You will now see nodes roll through and update to the current version.
