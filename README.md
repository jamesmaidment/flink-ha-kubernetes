# flink-ha-kubernetes
Flink running in HA mode in Kubernetes

Inspired by the following work:

- [Running Apache Flink on Kubernetes](https://jobs.zalando.com/tech/blog/running-apache-flink-on-kubernetes/) by Tobias Bahls
- ["Flink in Containerland"](https://www.youtube.com/watch?v=w721NI-mtAA)  by Patrick Lucas

## Bootstrap zetcd / Zookeeper

In order to run Flink in high-availability mode, Zookeeper is required.

If you wish to avoid the potential overhead of running Zookeeper in Kubernetes, included is a deployment for [zetcd](https://github.com/etcd-io/zetcd) as a dropin replacement for Zookeeper.

In order to deploy zetcd before starting the Flink cluster, run:

`kubectl create -f zetcd-deployment.yaml`

Alternatively you can set up Zookeeper in Kubernetes using the following instructions https://kubernetes.io/docs/tutorials/stateful-application/zookeeper/

If you are running your own Zookeeper, update the `configmap.yaml` value `high-availability.zookeeper.quorum` to point to your Zookeeper cluster.

## Deploy Flink HA job cluster

This directory contains a predefined K8s service, two template files for the job manager and the task managers, a configmap for both services and the zetcd deployment.

The K8s service is used to provide a stable DNS entrypoint to the job manager.

In order to use the template files, please replace the `${VARIABLES}` in the file with concrete values.
The files contain the following variables:

- `${FLINK_IMAGE_NAME}`: Name of the image to use for the container
- `${FLINK_JOB_PARALLELISM}`: Degree of parallelism with which to start the Flink job and the number of required task managers

One way to substitute the variables is to use `envsubst`.
See [here](https://stackoverflow.com/a/23622446/4815083) for a guide to install it on Mac OS X.

Alternatively, copy the template files (suffixed with `*.template`) and replace the variables.

In order to deploy the job manager run:

`FLINK_IMAGE_NAME=<IMAGE_NAME> FLINK_JOB_PARALLELISM=<PARALLELISM> envsubst < job-manager-deployment.yaml.template | kubectl create -f -`

Now you should see the `flink-job-manager` deployment being started by calling `kubectl get deployment`.

You should then create the job manager service:

`kubectl create -f job-manager-service.yaml`

Then you should start the task manager deployment:

`FLINK_IMAGE_NAME=<IMAGE_NAME> FLINK_JOB_PARALLELISM=<PARALLELISM> envsubst < task-manager-deployment.yaml.template | kubectl create -f -`

At last, create the job to run on the cluster:

`FLINK_IMAGE_NAME=<IMAGE_NAME> envsubst < job.yaml.template | kubectl create -f -`
