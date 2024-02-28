# elastic-stack-deployment-via-eck
Elastic Stack deployment through Elastic Cloud on Kubernetes (ECK)

# elk-stack-via-eck

This document is a guide which provides brief information about Elastic Cloud on Kubernetes (ECK) and steps to deploy ELK Stack on Kubernetes through ECK version 2.10.0.

**Table of Contents**

- [**Elastic Cloud on Kubernetes (ECK) Overview**](#elastic-cloud-on-kubernetes-eck-overview)
- [**Deployment Steps**](#deployment-steps)
- [**Resources**](#resources)

## Elastic Cloud on Kubernetes (ECK) Overview

Built on the Kubernetes Operator pattern, Elastic Cloud on Kubernetes (ECK) extends the basic Kubernetes orchestration capabilities to support the setup and management of Elasticsearch, Kibana, APM Server, Enterprise Search, Beats, Elastic Agent, Elastic Maps Server, and Logstash on Kubernetes.

With Elastic Cloud on Kubernetes you can streamline critical operations, such as:

Managing and monitoring multiple clusters
Scaling cluster capacity and storage
Performing safe configuration changes through rolling upgrades
Securing clusters with TLS certificates
Setting up hot-warm-cold architectures with availability zone awareness

## Deployment Steps

Things to consider before you start:

- For this guide, your Kubernetes cluster is assumed to be already up and running. Before you proceed with the ECK installation, make sure you check the [supported versions](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html).
- If you are using GKE, make sure your user has `cluster-admin` permissions. For more information, check [Prerequisites for using Kubernetes RBAC on GKE](https://cloud.google.com/kubernetes-engine/docs/how-to/role-based-access-control#iam-rolebinding-bootstrap).
- If you are using Amazon EKS, make sure the Kubernetes control plane is allowed to communicate with the Kubernetes nodes on port 443. This is required for communication with the Validating Webhook. For more information, check [Recommended inbound traffic](https://docs.aws.amazon.com/eks/latest/userguide/sec-group-reqs.html).
Refer to [*Install ECK*](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-installing-eck.html) for more information on installation options.

1. cd into the **manifests** directory.

```console
cd manifests
```

2. Install Custom Resource Definitions (CRDs):

```console
kubectl apply -f 01_crds.yaml
```

3. Install the operator with its RBAC and Pod assignment rules:

```console
kubectl apply -f 02_operator.yaml
```

**Note:** The ECK operator runs by default in the elastic-system namespace.

4. Apply Elasticsearch cluster specification:

```console
kubectl apply -f 03_elasticsearch.yaml
```

**Note:** 
If your Kubernetes cluster does not have any Kubernetes nodes with at least 2GiB of free memory, the pod will be stuck in `Pending` state.

5. Get and note the default Elasticsearch user's (`elastic`) password:

```console
PASSWORD=$(kubectl -n elastic-system get secret elastic-es-elastic-user -o go-template='{{.data.elastic | base64decode}}') ; echo $PASSWORD
```

6. Deploy a Kibana instance:

```console
kubectl apply -f 04_kibana.yaml
```

7. Create an ingress resource to access Kibana UI:

```console
kubectl apply -f 05_kibana-ingress.yaml
```

8. Access Kibana UI via [**https://www.example.com/kibana**](https://www.example.com/kibana) and enter `elastic` user credentials to login.

9. On Kibana UI,
    - Go to **Management -> Stack Management -> Security -> API keys**.
    - Click on **+ Create API key**.
    - Name your API key and click on **Create API key** button.
    - On the **Created API key** section, select **Logstash** format and note the API key.

10. Open **06_logstash.yaml** file with a text editor, paste the API key created in the previous step to **spec -> pipelines -> config.string -> output -> elasticsearch -> api_key** section and save the file.

11. Deploy a Logstash instance:

```console
kubectl apply -f 06_logstash.yaml
```

12. Create a configMap for Filebeat configuration:

```console
kubectl apply -f 07_filebeat-configMap-ls.yaml
```

13. Deploy a Filebeat instance:

```console
kubectl apply -f 08_filebeat-ls.yaml
```

## Resources

- [Elastic Cloud on Kubernetes \[2.10\] Overview](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-overview.html)
- [Deploy ECK in your Kubernetes cluster](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-eck.html)
- [Deploy an Elasticsearch cluster](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-elasticsearch.html)
- [Deploy a Kibana instance](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-kibana.html)
- [Use persistent storage](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-persistent-storage.html)
- [Volume claim templates](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-volume-claim-templates.html)
- [How to extract data from filename using logstash?](https://discuss.elastic.co/t/how-to-extract-data-from-filename-using-logstash/257371)
- [Configure SSL](https://www.elastic.co/guide/en/beats/filebeat/current/configuration-ssl.html)
- [Elasticsearch output plugin](https://www.elastic.co/guide/en/logstash/current/plugins-outputs-elasticsearch.html)
- [Logstash Grok Patterns](https://github.com/hpcugent/logstash-patterns/blob/master/files/grok-patterns)

