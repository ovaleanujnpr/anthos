# Integrate a Kubernetes and EKS cluster with Contrail in Google Anthos


Anthos is a portfolio of products and services for hybrid cloud and workload management that runs on the Google Kubernetes Engine (GKE) and users can manage workloads running also on third-party clouds like AWS, Azure and on-premises (private) clusters.
The following diagram shows Anthos components and features and how they provide Anthos's functionality across your environments, from infrastructure management to facilitating application development.

![](https://github.com/ovaleanujnpr/anthos/blob/master/images/anthos-14-components.png)
Reference https://cloud.google.com/anthos/docs/concepts/overview

The scope of this document is integrate a on-prem K8s runing with Contrail together with an EKS with Contrail cluster and a native GKE cluster in GCP Anthos.
So, in my case I will have one GKE on Google Cloud and two attached clusters, Kubernetes with Contrail running on-prem and EKS 4 with Contrail running in AWS.
Acording with the diagram for an attached cluster, Anthos has the following components:

1. Multi-cluster Management: Connect, Environ
2. Configuration Management: Anthos Config Management, Policy Controller
3. Service Management: Anthos Service Mesh

I will test each component from Anthos with my Contrail attached clusters.

Fun fact Anthos is flower in Greek. The reason they chose that is because flowers grow on premise, but they need rain from the cloud to flourish.


As prerequisites, I will need to install GCP cli tolls from Cloud SDK package, [Install Cloud SDK on macOS](https://cloud.google.com/sdk/docs/quickstart-macos), `kubectl` and because we will have three different clusters [`kubectx` and `kubens`](https://github.com/ahmetb/kubectx) are useful tools to managae them easily.

_Note: if `kubectl` version is lower than the [minimum supported Kubernetes version](https://cloud.google.com/kubernetes-engine/docs/release-notes) of Google Kubernetes Engine (GKE), then you need to update it_


## Creating Kubernetes clusters

### Creating on-prem Kubernetes cluster using Contrail SDN


I installed a Kubernetes cluster using Contrail SDN on BMS following the procedure from this [Wiki](https://github.com/ovaleanujnpr/Kubernetes/wiki/Installing-Kubernetes-with-Contrail).

In this case, I installed Kubernetes 1.18.9 on Ubuntu 18.04 OS.

```
$ kubectl get nodes -o wide
NAME          STATUS   ROLES    AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
k8s-master1   Ready    master   19h   v1.18.9   172.16.125.115   <none>        Ubuntu 18.04.5 LTS   4.15.0-118-generic   docker://18.9.9
k8s-master2   Ready    master   19h   v1.18.9   172.16.125.116   <none>        Ubuntu 18.04.5 LTS   4.15.0-118-generic   docker://18.9.9
k8s-master3   Ready    master   19h   v1.18.9   172.16.125.117   <none>        Ubuntu 18.04.5 LTS   4.15.0-118-generic   docker://18.9.9
k8s-node1     Ready    <none>   19h   v1.18.9   172.16.125.118   <none>        Ubuntu 18.04.5 LTS   4.15.0-112-generic   docker://18.9.9
k8s-node2     Ready    <none>   19h   v1.18.9   172.16.125.119   <none>        Ubuntu 18.04.5 LTS   4.15.0-112-generic   docker://18.9.9
```

After the Kubernetes cluster is deployed, I installed Contrail using single yaml file.
```
$ kubectl get po -n kube-system
NAME                                  READY   STATUS    RESTARTS   AGE
config-zookeeper-4klts                1/1     Running   0          19h
config-zookeeper-cs2fk                1/1     Running   0          19h
config-zookeeper-wgrtb                1/1     Running   0          19h
contrail-agent-ch8kv                  3/3     Running   2          19h
contrail-agent-kh9cf                  3/3     Running   1          19h
contrail-agent-kqtmz                  3/3     Running   0          19h
contrail-agent-m6nrz                  3/3     Running   1          19h
contrail-agent-qgzxt                  3/3     Running   3          19h
contrail-analytics-6666s              4/4     Running   1          19h
contrail-analytics-jrl5x              4/4     Running   4          19h
contrail-analytics-x756g              4/4     Running   4          19h
contrail-configdb-2h7kd               3/3     Running   4          19h
contrail-configdb-d57tb               3/3     Running   4          19h
contrail-configdb-zpmsq               3/3     Running   4          19h
contrail-controller-config-c2226      6/6     Running   9          19h
contrail-controller-config-pbbmz      6/6     Running   5          19h
contrail-controller-config-zqkm6      6/6     Running   4          19h
contrail-controller-control-2kz4c     5/5     Running   2          19h
contrail-controller-control-k522d     5/5     Running   0          19h
contrail-controller-control-nr54m     5/5     Running   2          19h
contrail-controller-webui-5vxl7       2/2     Running   0          19h
contrail-controller-webui-mzpdv       2/2     Running   1          19h
contrail-controller-webui-p8rc2       2/2     Running   1          19h
contrail-kube-manager-88c4f           1/1     Running   0          19h
contrail-kube-manager-fsz2z           1/1     Running   0          19h
contrail-kube-manager-qc27b           1/1     Running   0          19h
coredns-684f7f6cb4-4mmgc              1/1     Running   0          93m
coredns-684f7f6cb4-dvpjk              1/1     Running   0          107m
coredns-684f7f6cb4-m6sj7              1/1     Running   0          84m
coredns-684f7f6cb4-nfkfh              1/1     Running   0          84m
coredns-684f7f6cb4-tk48d              1/1     Running   0          86m
etcd-k8s-master1                      1/1     Running   0          94m
etcd-k8s-master2                      1/1     Running   0          95m
etcd-k8s-master3                      1/1     Running   0          92m
kube-apiserver-k8s-master1            1/1     Running   0          94m
kube-apiserver-k8s-master2            1/1     Running   0          95m
kube-apiserver-k8s-master3            1/1     Running   0          92m
kube-controller-manager-k8s-master1   1/1     Running   0          94m
kube-controller-manager-k8s-master2   1/1     Running   0          95m
kube-controller-manager-k8s-master3   1/1     Running   0          92m
kube-proxy-975tn                      1/1     Running   0          108m
kube-proxy-9qzc9                      1/1     Running   0          108m
kube-proxy-fgwqt                      1/1     Running   0          109m
kube-proxy-n6nnq                      1/1     Running   0          109m
kube-proxy-wf289                      1/1     Running   0          108m
kube-scheduler-k8s-master1            1/1     Running   0          94m
kube-scheduler-k8s-master2            1/1     Running   0          95m
kube-scheduler-k8s-master3            1/1     Running   0          90m
rabbitmq-82lmk                        1/1     Running   0          19h
rabbitmq-b2lz8                        1/1     Running   0          19h
rabbitmq-f2nfc                        1/1     Running   0          19h
redis-42tkr                           1/1     Running   0          19h
redis-bj76v                           1/1     Running   0          19h
redis-ctzhg                           1/1     Running   0          19h
```
```
$ kubectl create clusterrolebinding permissive-binding \
  --clusterrole=cluster-admin \
  --user=admin \
  --user=kubelet \
  --group=system:serviceaccounts
```

Grant the cluster-admin RBAC role
```
kubectl auth can-i '*' '*' --all-namespaces
```

### Creating a EKS cluster with Contrail in AWS

For installing an EKS cluster with Contrail in AWS, follow the installation procedure [here](https://ssd-git.juniper.net/sre/contrail-as-the-cni-for-aws-eks)

```
$ kubectl get nodes -o wide
NAME                                            STATUS   ROLES    AGE   VERSION    INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
ip-100-72-0-119.eu-central-1.compute.internal   Ready    infra    30h   v1.16.15   100.72.0.119   <none>        Ubuntu 18.04.3 LTS   4.15.0-1054-aws   docker://17.3.2
ip-100-72-0-216.eu-central-1.compute.internal   Ready    infra    30h   v1.16.15   100.72.0.216   <none>        Ubuntu 18.04.3 LTS   4.15.0-1054-aws   docker://17.3.2
ip-100-72-0-245.eu-central-1.compute.internal   Ready    infra    30h   v1.16.15   100.72.0.245   <none>        Ubuntu 18.04.3 LTS   4.15.0-1054-aws   docker://17.3.2
ip-100-72-1-116.eu-central-1.compute.internal   Ready    <none>   30h   v1.16.15   100.72.1.116   <none>        Ubuntu 18.04.3 LTS   4.15.0-1054-aws   docker://17.3.2
ip-100-72-1-67.eu-central-1.compute.internal    Ready    <none>   30h   v1.16.15   100.72.1.67    <none>        Ubuntu 18.04.3 LTS   4.15.0-1054-aws   docker://17.3.2

$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                READY   STATUS    RESTARTS   AGE
kube-system   cni-patches-2svt6                   1/1     Running   0          44h
kube-system   cni-patches-9mpss                   1/1     Running   0          44h
kube-system   cni-patches-fdbws                   1/1     Running   0          44h
kube-system   cni-patches-ggdph                   1/1     Running   0          44h
kube-system   cni-patches-h46q2                   1/1     Running   0          44h
kube-system   config-zookeeper-68tw7              1/1     Running   0          44h
kube-system   config-zookeeper-9b799              1/1     Running   0          44h
kube-system   config-zookeeper-gfxfw              1/1     Running   0          44h
kube-system   contrail-agent-2dm9q                3/3     Running   0          44h
kube-system   contrail-agent-4bsx7                3/3     Running   0          44h
kube-system   contrail-agent-56c87                3/3     Running   0          44h
kube-system   contrail-agent-b7n68                3/3     Running   0          44h
kube-system   contrail-agent-ckqk7                3/3     Running   0          44h
kube-system   contrail-analytics-alarm-hd99q      4/4     Running   0          44h
kube-system   contrail-analytics-alarm-trbdq      4/4     Running   0          44h
kube-system   contrail-analytics-alarm-znkc2      4/4     Running   0          44h
kube-system   contrail-analytics-dmk77            4/4     Running   0          44h
kube-system   contrail-analytics-nnhsx            4/4     Running   0          44h
kube-system   contrail-analytics-snmp-q82bj       4/4     Running   0          44h
kube-system   contrail-analytics-snmp-q8kh7       4/4     Running   0          44h
kube-system   contrail-analytics-snmp-qnjlc       4/4     Running   0          44h
kube-system   contrail-analytics-xng7q            4/4     Running   0          44h
kube-system   contrail-analyticsdb-jnmg8          4/4     Running   0          44h
kube-system   contrail-analyticsdb-mtp5r          4/4     Running   0          44h
kube-system   contrail-analyticsdb-pkx5t          4/4     Running   0          44h
kube-system   contrail-configdb-4682z             3/3     Running   0          44h
kube-system   contrail-configdb-gqps2             3/3     Running   0          44h
kube-system   contrail-configdb-wbxcp             3/3     Running   0          44h
kube-system   contrail-controller-config-jvgb9    6/6     Running   2          44h
kube-system   contrail-controller-config-qbt74    6/6     Running   3          44h
kube-system   contrail-controller-config-x79xd    6/6     Running   3          44h
kube-system   contrail-controller-control-6mp8r   5/5     Running   0          44h
kube-system   contrail-controller-control-v5p9j   5/5     Running   0          44h
kube-system   contrail-controller-control-v72qt   5/5     Running   0          44h
kube-system   contrail-controller-webui-4gwkm     2/2     Running   0          44h
kube-system   contrail-controller-webui-965bp     2/2     Running   0          44h
kube-system   contrail-controller-webui-crjzb     2/2     Running   0          44h
kube-system   contrail-kube-manager-5mk8l         1/1     Running   0          44h
kube-system   contrail-kube-manager-bn4nx         1/1     Running   1          44h
kube-system   contrail-kube-manager-dvgr9         1/1     Running   1          44h
kube-system   coredns-5fdf64ff8-5tjnq             1/1     Running   0          44h
kube-system   coredns-5fdf64ff8-g4j95             1/1     Running   0          44h
kube-system   kube-proxy-4zw9q                    1/1     Running   1          46h
kube-system   kube-proxy-8d49f                    1/1     Running   1          46h
kube-system   kube-proxy-s5qhd                    1/1     Running   1          46h
kube-system   kube-proxy-xwxqf                    1/1     Running   1          46h
kube-system   kube-proxy-z6wnp                    1/1     Running   1          46h
kube-system   rabbitmq-mjqgb                      1/1     Running   0          44h
kube-system   rabbitmq-s44vd                      1/1     Running   0          44h
kube-system   rabbitmq-tbwwh                      1/1     Running   0          44h
kube-system   redis-fg6xv                         1/1     Running   0          44h
kube-system   redis-gtlsk                         1/1     Running   0          44h
kube-system   redis-vgrz9                         1/1     Running   0          44h
kube-system   upgrade-nodes-75fzq                 1/1     Running   1          44h
kube-system   upgrade-nodes-fcrsq                 1/1     Running   1          44h
kube-system   upgrade-nodes-lfjll                 1/1     Running   1          44h
kube-system   upgrade-nodes-qd64n                 1/1     Running   1          44h
kube-system   upgrade-nodes-t9lc6                 1/1     Running   1          44h
```

```
$ kubectl create clusterrolebinding permissive-binding \
  --clusterrole=cluster-admin \
  --user=admin \
  --user=kubelet \
  --group=system:serviceaccounts
```

Grant the cluster-admin RBAC role
```
kubectl auth can-i '*' '*' --all-namespaces
```

### Creating a GKE cluster in GCP

Before creating the GKE cluster, I need to create a project in GCP.
I will use `gcloud init` command to intialise the SDK:

```
$ gcloud init
```

Follow the process and create a project. Then check the project creation and project id.

```
$ gcloud projects list
```

Choose the project

```
$ gcloud config set project contrail-k8s-289615
```

I need to grant the required IAM roles to the user registering the cluster

```
PROJECT_ID=contrail-k8s-289615
$ gcloud projects add-iam-policy-binding ${PROJECT_ID} \
 --member user:[GCP_EMAIL_ADDRESS] \
 --role=roles/gkehub.admin \
 --role=roles/iam.serviceAccountAdmin \
 --role=roles/iam.serviceAccountKeyAdmin \
 --role=roles/resourcemanager.projectIamAdmin
```

I enable the APIs required for my project

```
gcloud services enable \
 --project=${PROJECT_ID} \
 container.googleapis.com \
 compute.googleapis.com \
 gkeconnect.googleapis.com \
 gkehub.googleapis.com \
 cloudresourcemanager.googleapis.com \
 cloudtrace.googleapis.com \
 anthos.googleapis.com \
 iamcredentials.googleapis.com \
 meshca.googleapis.com \
 meshconfig.googleapis.com \
 meshtelemetry.googleapis.com \
 monitoring.googleapis.com \
 logging.googleapis.com \
 runtimeconfig.googleapis.com
```

Now, I will create the GKE cluster

```
$ export KUBECONFIG=gke-config
$ gcloud container clusters create gke-cluster-1 \
--zone "europe-west2-b" \
--disk-type "pd-ssd" \
--disk-size "150GB" \
--machine-type "n2-standard-4" \
--num-nodes=3 \
--image-type "COS" \
--enable-stackdriver-kubernetes \
--addons HorizontalPodAutoscaling,HttpLoadBalancing,Istio,CloudRun \
--istio-config auth=MTLS_PERMISSIVE \
--cluster-version "1.17.9-gke.1504"
```

```
kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole cluster-admin \
  --user $(gcloud config get-value account)
```

Because I will have to change the context often from one cluster to another, I will merge all the contexs into one configuration

Copy the $HOME/.kube/config from on-prem cluster on my mac as `contrail-config` in `~/.kube`.

Copy the ocp4 and gke configs in the same directory

```
$ cp *-config ~/.kube
$ KUBECONFIG=$HOME/.kube/eks-config:$HOME/.kube/contrail-config:$HOME/.kube/gke-config kubectl config view --merge --flatten > $HOME/.kube/config

$ kubectx gke_contrail-k8s-289615_europe-west2-a_gke-cluster-1
$ kubectx gke=.

$ kubectx arn:aws:eks:eu-central-1:927874460243:cluster/EKS-YC0U0TU5
$ kubectx eks-contrail=.

$ kubectx kubernetes-admin@kubernetes
$ kubectx onprem-k8s-contrail=.
```

Now we have three context representing the clusters

```
$ kubectx
eks-contrail
gke
onprem-k8s-contrail
```

### Configure the GCP account for Anthos

Before registering the clusters we need to create a service account and JSON file containing Google Cloud Service Account credentials for external clusters (On-premises and EKS) to connect to Anthos

```
$ PROJECT_ID=contrail-k8s-289615
$ SERVICE_ACCOUNT_NAME=anthos-connect

$ gcloud iam service-accounts create ${SERVICE_ACCOUNT_NAME} --project=${PROJECT_ID}
```

Bind the gkehub.connect IAM role to the service account:

```
$ gcloud projects add-iam-policy-binding ${PROJECT_ID} \
 --member="serviceAccount:${SERVICE_ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com" \
 --role="roles/gkehub.connect"
```

Create the service account's private key JSON file in current directory. I will need this file for registering the clusters:
```
$ gcloud iam service-accounts keys create ./${SERVICE_ACCOUNT_NAME}-svc.json \
  --iam-account=${SERVICE_ACCOUNT_NAME}@${PROJECT_ID}.iam.gserviceaccount.com \
  --project=${PROJECT_ID}
```

## Multi-cluster Management

### Register an External Kubernetes Cluster to GKE Connect

Connect allows you to connect any of your Kubernetes clusters to Google Cloud. This enables access to cluster and to workload management features, including a unified user interface, Cloud Console, to interact with your cluster. More details [here](https://cloud.google.com/anthos/multicluster-management/connect/overview).

![](https://github.com/ovaleanujnpr/anthos/blob/master/images/image2.png)


To register a non-GKE clusters we need to run the following command

For on-prem Contrail cluster:

```
gcloud container hub memberships register onpremk8s-contrail-cluster-1 \
   --project=${PROJECT_ID} \
   --context=onprem-k8s-contrail \
   --kubeconfig=$HOME/.kube/config \
   --service-account-key-file=./anthos-connect-svc.json
```

When the command finishes a new pod called gke-connect-agent will run in the cluster. This is responsabile to communication with GKE Hub as I decribed above.
```
$ kubectx onprem-k8s-contrail
Switched to context "onprem-k8s-contrail".

$ kubectl get pods -n gke-connect
NAMESPACE      NAME                                               READY   STATUS      RESTARTS   AGE
gke-connect    gke-connect-agent-20200918-01-00-7bc77884d-st4r2   1/1     Running     0          45m
```

_Note: I need SNAT enabled in Contrail to allow gke-connect-agent communication to internet_

The same for the Contrail EKS cluster

```
gcloud container hub memberships register eks-contrail-cluster-1 \
   --project=${PROJECT_ID} \
   --context=eks-contrail \
   --kubeconfig=$HOME/.kube/config \
   --service-account-key-file=./anthos-connect-svc.json
```
The same `gke-connect-agent` will be installed like on on-prem cluster

```
$ kubectx eks-contrail
Switched to context "eks-contrail".

$ kubectl get pods -n gke-connect
NAME                                                READY   STATUS    RESTARTS   AGE
gke-connect-agent-20201002-01-00-5749bfc847-qhvft   1/1     Running   0          43m
```

And the GKE cluster

```
gcloud container hub memberships register gke-cluster-1 \
--project=${PROJECT_ID} \
--gke-cluster=europe-west2-b/gke-cluster-1 \
--service-account-key-file=./anthos-connect-svc.json
```

I can view cluster registration status and all the clusters within my Google project

```
$ gcloud container hub memberships list
NAME                          EXTERNAL_ID
onpremk8s-contrail-cluster-1  78f7890b-3a43-4bc7-8fd9-44c76953781b
eks-contrail-cluster-1        42e532ba-a0d9-4087-baed-647be8bca7e9
gke-cluster-1                 6671599e-87af-461b-aff9-7105ebda5c66
```

To login to the external clusters from the Google Anthos Console I will use a bearer token. For this I will create a Kubernetes service account (KSA) in the cluster.

I am creating and applying first a  node-reader role-based access control (RBAC) role

```
$ cat <<EOF > node-reader.yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
 name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
EOF

$ kubectx contrail

$ kubectl apply -f node-reader.yaml
```

Create and authorise a KSA.

```
$ KSA_NAME=anthos-sa
$ kubectl create serviceaccount ${KSA_NAME}
$ kubectl create clusterrolebinding anthos-view --clusterrole view --serviceaccount default:${KSA_NAME}
$ kubectl create clusterrolebinding anthos-node-reader --clusterrole node-reader --serviceaccount default:${KSA_NAME}
$ kubectl create clusterrolebinding anthos-cluster-admin --clusterrole cluster-admin --serviceaccount default:${KSA_NAME}
```

To acquire the KSA's bearer token, run the following command:

```
$ SECRET_NAME=$(kubectl get serviceaccount ${KSA_NAME} -o jsonpath='{$.secrets[0].name}')
$ kubectl get secret ${SECRET_NAME} -o jsonpath='{$.data.token}' | base64 --decode
```

The output token use it in Cloud Console to Login to the cluster

I will do the same for EKS cluster

```
$ kubectx eks-contrail
$ $ kubectl apply -f node-reader.yaml

$ kubectl create serviceaccount ${KSA_NAME}
$ kubectl create clusterrolebinding anthos-view --clusterrole view --serviceaccount default:${KSA_NAME}
$ kubectl create clusterrolebinding anthos-node-reader --clusterrole node-reader --serviceaccount default:${KSA_NAME}
$ kubectl create clusterrolebinding anthos-cluster-admin --clusterrole cluster-admin --serviceaccount default:${KSA_NAME}

$ SECRET_NAME=$(kubectl get serviceaccount ${KSA_NAME} -o jsonpath='{$.secrets[0].name}')
$ kubectl get secret ${SECRET_NAME} -o jsonpath='{$.data.token}' | base64 --decode
```

The clusters should be visibile in Anthos

![](https://github.com/ovaleanujnpr/anthos/blob/master/images/image23.png)

![](https://github.com/ovaleanujnpr/anthos/blob/master/images/image24.png)

I can view details about it in Kubernetes Engine tab

![](https://github.com/ovaleanujnpr/anthos/blob/master/images/image25.png)

![](https://github.com/ovaleanujnpr/anthos/blob/master/images/image15.png)

![](https://github.com/ovaleanujnpr/anthos/blob/master/images/image26.png)

### Deploy Anthos Apps from GCP Marketplace into Kubernetes on-prem cluster

The first time you deploy an application to a Anthos GKE on-prem cluster, you must also create a namespace called `application-system` for Cloud Marketplace components, and apply an imagePullSecret to the default service account for the namespace.

```
$ kubectl create ns application-system

$ kubens application-system
Context "kubernetes-admin@kubernetes" modified.
Active namespace is "application-system".
```

To pull the images from GCR, we need to create a service account and download the associated JSON token

```
$ PROJECT_ID=contrail-k8s-289615

$ gcloud iam service-accounts create gcr-sa \
	--project=${PROJECT_ID}

$ gcloud iam service-accounts list \
	--project=${PROJECT_ID}

$ gcloud projects add-iam-policy-binding ${PROJECT_ID} \
 --member="serviceAccount:gcr-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
 --role="roles/storage.objectViewer"

$ gcloud iam service-accounts keys create ./gcr-sa.json \
  --iam-account="gcr-sa@${PROJECT_ID}.iam.gserviceaccount.com" \
  --project=${PROJECT_ID}
```

Create a secret with the contents of the token

```
$ kubectl create secret docker-registry gcr-json-key \
--docker-server=https://marketplace.gcr.io \
--docker-username=_json_key \
--docker-password="$(cat ./gcr-sa.json)" \
--docker-email=[GCP_EMAIL_ADDRESS]
```

Patch the default service account within the namespace to use the secret to pull images from GCR instead of Docker Hub

```
$ kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "gcr-json-key"}]}'
```

Annotate the `application-system` namespace to enable the deployment of Kubernetes Apps from GCP Marketplace

```
$ kubectl annotate namespace application-system marketplace.cloud.google.com/imagePullSecret=gcr-json-key
```

GCP Marketplace expects a storage class by name `standard` as the default storage class.

Rename you storage class if it has a different name or create it. [Here](https://github.com/ovaleanujnpr/kubernetes/blob/master/docs/add_local_pv_k8s.md) is how to create a storage class using local volumes.

```
$ cat sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

```
$ kubectl get sc
NAME                 PROVISIONER                    AGE
standard (default)   kubernetes.io/no-provisioner   6m14s
```

This will be utilized by the GCP Marketplace Apps to dynamically provision Persistent Volume (PV) and Persistent Volume Claim (PVC).


Next, we need to create and configure a namespace for the app we will deploy it from GCP Marketplace. We will deploy PostgreSQL.

```
$ kubectl create ns pgsql

$ kubens pgsql

$ kubectl create secret docker-registry gcr-json-key \
 --docker-server=https://gcr.io \
--docker-username=_json_key \
--docker-password="$(cat ./gcr-sa.json)" \
--docker-email=[GCP_EMAIL_ADDRESS]
```

Docker-server key is pointing to https://gcr.io which holds the container images for the GCP Marketplace Apps

Like we did for `application-system` namespace, we need to patch the default service account within the pgsql namespace to use the secret to pull images from GCR instead of Docker Hub

```
$ kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "gcr-json-key"}]}'
```

Also annotate the pgsql namespace to enable the deployment of Kubernetes Apps from GCP Marketplace

```
$ kubectl annotate namespace pgsql marketplace.cloud.google.com/imagePullSecret=gcr-json-key
```

All these steps were a preparation to deploy an app from GCP Marketplace to the on-prem K8s cluster with Contrail

Choose PostgresSQL Server from GCP Marketplace and then click on Configure de start the deployment procedure

![](https://github.com/ovaleanujnpr/anthos/blob/master/images/image17.png)

Choose the external cluster, `onpremk8s-contrail-cluster-1`

![](https://github.com/ovaleanujnpr/anthos/blob/master/images/image27.png)

Select the namespace created, storage class and click Deploy

![](https://github.com/ovaleanujnpr/anthos/blob/master/images/image19.png)

After a minute PostgresSQl is deployed

![](https://github.com/ovaleanujnpr/anthos/blob/master/images/image20.png)

```
$ kubectl get po -n pgsql
NAME                          READY   STATUS      RESTARTS   AGE
postgresql-1-deployer-nzpfn   0/1     Completed   0          91s
postgresql-1-postgresql-0     2/2     Running     0          46s

$ kubectl get pvc
NAME                                                    STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgresql-1-postgresql-pvc-postgresql-1-postgresql-0   Bound    local-pv-e00b14f6   62Gi       RWO            standard       91s
```

In GKE Console we can filter to see the applications deployed on on-prem cluster

![](https://github.com/ovaleanujnpr/anthos/blob/master/images/image21.png)

Access PostgreSQL

Forward PostgreSQL port locally:

```
$ export NAMESPACE=pgsql
$ export APP_INSTANCE_NAME="postgresql-1"
$ kubectl port-forward --namespace "${NAMESPACE}" "${APP_INSTANCE_NAME}-postgresql-0" 5432
Forwarding from 127.0.0.1:5432 -> 5432
Forwarding from [::1]:5432 -> 5432
```

Connect to the database:

```
$ apt -y install postgresql-client-10 postgresql-client-common
$ export PGPASSWORD=$(kubectl get secret "postgresql-1-secret" --output=jsonpath='{.data.password}' | base64 -d)
$ psql (10.12 (Ubuntu 10.12-0ubuntu0.18.04.1), server 9.6.18)
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=#
```

## Configuration Management

### Anthos Config Management

In this part will I use GitOps-style deployment to push workloads across all registered clusters through Anthos Config Management.
GitOps is a way to do Kubernetes cluster management and application delivery.  It works by using Git as a single source of truth for declarative infrastructure and applications. Since Kubernetes uses YAML or JSON files for specifications, it becomes easy to combine these artifacts with code.
Google built a tool called [Config Sync](https://cloud.google.com/kubernetes-engine/docs/add-on/config-sync/overview) which acts as the bridge between an external source code repository and the Kubernetes API server. Anthos Config Management is based on Config Sync to extend it to multicluster scenarios.
I will use a GitHub repository that acts as a single source of truth for deployments and configuration. A component of ACM is installed into each of the registered Anthos clusters to monitor the external repository for any changes and synchronizing them with the cluster.

### Installing the Configuration Management Operator