# wazuh-kubernetes
Wazuh (3.6) cluster on top of Kubernetes (tested with v1.10.3) with a working simple ELK stack.

## Abstract
Wazuh best practices recommend deploying multiple instances of the Wazuh manager so it can support a larger amount of events and can be fault tolerant.
* `Master` node - intended to expose the Wazuh API, manage agents registration
* `Worker` nodes - intended to receive agents events

You should take a quick look at [the Wazuh cluster documentation](https://documentation.wazuh.com/current/user-manual/manager/wazuh-cluster.html) before deploying what's in this repository.

Based on that, we demonstrate that it's possible to deploy the Wazuh cluster in a Kubernetes cluster on top of AWS. Master and worker nodes are all behind an internal AWS elastic load balancer, so the events from agents are dispatched to any available nodes in the cluster. Kubernetes will ensure that the cluster stays highly available.

This repository is using Docker images from the [wazuh-docker](https://github.com/wazuh/wazuh-docker) repository. A simple `kompose convert -f docker-compose.yml` helped a lot to build this Kubernetes example!

## Pre-requisites
* A Kubernetes cluster (tested with v1.10.3) on top of [AWS](https://aws.amazon.com/). Was tested in AWS EKS.
  * You should be able to create Persistent Volumes on top of AWS EBS when using a volumeClaimTemplates in a Kubernetes StatefulSet.
  * You should be able to create a record set in AWS Route 53 from a Kubernetes LoadBalancer.
* Having at least two Kubernetes node, otherwise, Wazuh manager worker nodes won't be able to boot due to the podAntiAffinity policy.

## Wazuh manager cluster deployment
First, you need to deploy Kubernetes YAML files in the [base](base) folder:
```BASH
pushd base

kubectl apply -f aws-gp2-storage-class.yaml
kubectl apply -f wazuh-ns.yaml

popd
```

Once done, deploy the Elasticsearch StatefulSet from the [elasticsearch](elasticsearch) folder:
```BASH
pushd elasticsearch

kubectl apply -f elasticsearch-sts-svc.yaml
kubectl apply -f elasticsearch-svc.yaml

kubectl apply -f elasticsearch-sts.yaml

popd
```

Then, it's preferable you wait for the Elasticsearch Pod to be fully up and initialized before you deploy Kibana. The Kibana Pod will load Wazuh templates in the Elasticsearch installation. You can deploy the Kibana Kubernetes Deployment from the [kibana](kibana) folder:
```BASH
pushd kibana

kubectl apply -f kibana-svc.yaml
kubectl apply -f nginx-svc.yaml

kubectl apply -f kibana-deploy.yaml
kubectl apply -f nginx-deploy.yaml

popd
```

Deploy the Logstash Kubernetes Deployment from the [logstash](logstash) folder after Kibana successfully loaded Wazuh templates in the Elasticsearch installation:
```BASH
pushd logstash

kubectl apply -f logstash-svc.yaml

kubectl apply -f logstash-deploy.yaml

popd
```

Then, you can deploy what's in the [manager_cluster](manager_cluster) folder:
```BASH
pushd manager_cluster

kubectl apply -f wazuh-api-svc.yaml
kubectl apply -f wazuh-manager-cluster-sts-svc.yaml
kubectl apply -f wazuh-manager-svc.yaml

kubectl apply -f wazuh-manager-master-conf.yaml
kubectl apply -f wazuh-manager-worker-0-conf.yaml
kubectl apply -f wazuh-manager-worker-1-conf.yaml

kubectl apply -f wazuh-manager-master-sts.yaml
kubectl apply -f wazuh-manager-worker-0-sts.yaml
kubectl apply -f wazuh-manager-worker-1-sts.yaml

popd
```

Then, all the pieces should be up!
* The Wazuh API should be available at wazuh-api.some-domain.com:55000
* All manager nodes of your Wazuh manager cluster should be reachable at wazuh-manager.some-domain.com:1514
* Kibana and the Wazuh Kibana application should be available at https://wazuh.some-domain.com:443

## Wazuh agents deployment
This repository does not show how to deploy the Wazuh agent in a Kubernetes cluster. Normally, we would use a DaemonSet to deploy the agent on each Kubernetes node. To do that, we would need a Docker image with the Wazuh agent installed on it and then we would need to mount almost every folder of your host inside that container (`/bin`, `/etc`, `/var/log`, etc.). It would be a very complicated task since you cannot simply mount the `/bin` folder of your host in the `/bin` folder of your container. Therefore, creating such Docker image and using it in a Kubernetes DaemonSet is not the ideal way to deploy a Wazuh agent. Instead, you should take a look at the [Wazuh Ansible playbooks project](https://github.com/wazuh/wazuh-ansible) or at the [Wazuh Puppet module project](https://github.com/wazuh/wazuh-puppet) to deploy your Wazuh agents.

## TODO
* Secure the Wazuh API.
* Use an enterprise class Elasticsearch cluster instead of a single node. The [kubernetes-elasticsearch-cluster](https://github.com/pires/kubernetes-elasticsearch-cluster) repository looks like a great place from where to start.
* Make the Logstash deployment highly available.
* Make the Kibana deployment highly available.
