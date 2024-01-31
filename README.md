# neuvector-demo
A walkthrough for federating Neuvector

Neuvector is a software security tool that can be installed alongside Rancher. When Neuvector is installed into multiple Kubernetes clusters, 
each Neuvector instance can be federated to manage policies and scan results, as well as present an overall view in the Neuvector Manager UI.
This repo walks through how to setup federation and configure other capabilities in Neuvector.



## Install

### Master cluster steps
1. Neuvector can be installed via a Helm chart from the Rancher Marketplace.
2. On the cluster that you want to be the federation master, expose the `neuvector-svc-controller-fed-master` service externally.
3. Login to the Neuvector Manager UI and click on the User account drop down -> Multiple Clusters
4. Click Promote
5. Enter the endpoint that you exposed in step 2 and click Submit. (Note: Your Primary Cluster will be `443` if you exposed via Ingress)
6. After the page reloads, go back in to Multiple Clusters and you should now see the cluster listed in the table. Under the Actions column, click Generate Token and copy the displayed token to your clipboard for future use.

### Managed cluster steps
1. On all downstream Neuvector instances, Neuvector can be installed via a Helm chart from the Rancher Marketplace.
2. On these clusters you will need to expose the `neuvector-svc-controller-fed-managed` service externally.
3. Login to the Neuvector Manager UI and click on the User account drop down -> Multiple Clusters.
4. Click Join
5. For the controller server, enter the endpoint that you exposed in step 3. (Note: Your controller port will `443` if you exposed via Ingress) 
Enter the token you copied in step 6 of Master section and the remaining fields will be populated.
6. Click Submit.

## Configuring Harbor Integration

Harbor supports using Neuvector as an external image scanner. 

### Configuration Steps

1. Enable the registry adapter pod by setting `cve.adapter.enabled=true`
2. By default, the registry adapter pod is configured for HTTPS and you need to provide a TLS certificate via `cve.adapter.certificate` (Note: If you use this, set `cve.adapter.certificate.pemFile=tls.crt`) [example certificate](https://raw.githubusercontent.com/nnewc/neuvector-demo/main/adapter-certificate.yaml)
3. If Harbor is running external to cluster, you will need to expose the registry adapter pod externally. Ingress can be configured with `cve.adapter.ingress`
4. A secret of the form [basic-auth](https://kubernetes.io/docs/concepts/configuration/secret/#basic-authentication-secret) will need to be created and set via `cve.adapter.harbor.secretName` [example secret](https://raw.githubusercontent.com/nnewc/neuvector-demo/main/harbor-credentials.yaml)
5. After those resources are created, log into Harbor and navigate to Administration -> Interrogation Services -> New Scanner 
6. In the dialog box, enter the exposed `neuevector-registry-adapter` endpoint, the credentials found in the secret referenced by `cve.adapter.harbor.secretName`, then click Add.
7. You should now have a Neuvector scanner configured in Harbor.


