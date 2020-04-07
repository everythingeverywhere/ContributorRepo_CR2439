# Setting up your Modernapps Ninja Contributor Repository

**Contents:**

- [Step 1: About ModernApps Ninja Contributor Repository and Portfolio]()
- [Step 2: Request your contributor Repository]()
- [Step 3: Configure your contributor portfolio]()
- [Step 4: Post assignments and course completion records to your contributor repository]()
- [Step 5: Add your completion record to the verify service]()
- [Step 6: Update your portfolio to display your course completion certificate]()

## Step 1: About ModernApps Ninja Contributor Repository and Portfolio

1.1 The ModernApps Ninja Contributor Repository Service provides a dedicated repository for any participant in the ModernApps Ninja community. A contributor repository is required to recieve a course of completion for [modernapps.ninja](https://modernapps.ninja) courses that offer certificates of completion. 

Note that it is called a contributor repository and not a student or participant repository because all participants who complete course requirements provide meaningful contributions to the community as a core part of their learning experience and can rightfully consider themselves to be contributors to the community. 

The Contributor repository serves both as a location where participants can practice gitops skills and also post assignments and course completion records, which act as part of the proof of completion required to validate completion certificate requests. 

The Contributor repository also includes a portfolio website that displays a nicely organized page to display your modernapps.ninja course completion certificates, records, and contributions. 

## Step 2: Request your contributor Repository

#### 2.1 In a web browser, navigate to [https://github.com/modernappsninjas/dojo/issues](https://github.com/modernappsninjas/dojo/issues), and select the `New Issue` button to open an issue ticket.

<details><summary>Screenshot 2.1</summary>
<img src="media/2020-04-07-00-48-03.png">
</details>

#### 2.2 Select the `Contributor Repo Request` issue ticket template

<details><summary>Screenshot 2.2</summary>
<img src="media/2020-04-07-02-14-35.png">
</details>

#### 2.3 Fill out the `Contributor Repo Request` form and click `Submit new issue`

Use the title: `Contributor Repo Request for {Github UserID}` - using your github ID

Review the instructions in the request form, fill in your github ID and email address per the instructions in the form, and click `Submit new issue`

<details><summary>Screenshot 2.3</summary>
<img src="media/2020-04-07-02-19-50.png">
</details>

#### 2.4 Wait for your repo to be provisioned

Please note that while we make our best effort to provision your contributor repository as quickly as possible, it could take up to a few days to provision the repo. However, please be aware that requesting a contributor repo is a one-time process, once it is provisioned for you, it will be immediately usable for your ongoing needs.

Once we are able to review your request and provision your contributor repo, we will send an email to the provided email address with details and further instructions for accessing your contributor repository. 

## Step 3: Configure your contributor portfolio

Example:
`cp frontend-deployment_all_k8s.yaml frontend-deployment_ingress.yaml`

1.3.2 Get the URL of your smarcluster with the following command, be sure to replace 'afewell-cluster' with the name of your cluster:

``` bash
vke cluster show afewell-cluster | grep Address
```

<details><summary>Screenshot 1.3.2</summary>
<img src="media/2018-10-20-15-45-19.png">
</details>
<br/>

1.3.3 Edit the `frontend-deployment_ingress.yaml` file, near the bottom of the file in the ingress spec section, change the value for spec.rules.host to URL for your smartcluster as shown in the following snippet:

NOTE: Be sure to replace the URL shown here with the URL for your own smartcluster

``` bash
spec:
  rules:
  - host: afewell-cluster-69fc65f8-d37d-11e8-918b-0a1dada1e740.fa2c1d78-9f00-4e30-8268-4ab81862080d.vke-user.com
    http:
      paths:
      - backend:
          serviceName: planespotter-frontend
          servicePort: 80
```

<details><summary>Click to expand to see the full contents of frontend-deployment_ingress.yaml</summary>

When reviewing the file contents below, observe that it includes a ClusterIP service spec which only provides an IP address that is usable for pod-to-pod communications in the cluster. The file also includes an ingress spec which implements the default VKE ingress controller.

In the following steps after you deploy the planespotter-frontend with ingress controller, you will be able to browse from your workstation to the running planespotter app in your VKE environment even though you have not assigned a nat or public IP for the service.

Ingress controllers act as a proxies, recieving http/s requests from external clients and then based on the URL hostname or path, the ingress controller will proxy the request to the corresponding back-end service. For example mysite.com/path1 and mysite.com/path2 can be routed to different backing services running in the kubernetes cluster.

In the file below, no rules are specified to different paths and so accordingly, all requests sent to the host defined in the spec, your VKE SmartCluster URL, will be proxied by the ingress controller to the planespotter-frontend ClusterIP service also defined in the frontend-deployment_ingress.yaml file

``` bash
---
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: planespotter-frontend
  namespace: planespotter
  labels:
    app: planespotter-frontend
    tier: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: planespotter-frontend
  template:
    metadata:
      labels:
        app: planespotter-frontend
        tier: frontend
    spec:
      containers:
      - name: planespotter-fe
        image: yfauser/planespotter-frontend:d0b30abec8bfdbde01a36d07b30b2a3802d9ccbb
        imagePullPolicy: IfNotPresent
        env:
        - name: PLANESPOTTER_API_ENDPOINT
          value: planespotter-svc
        - name: TIMEOUT_REG
          value: "5"
        - name: TIMEOUT_OTHER
          value: "5"
---
apiVersion: v1
kind: Service
metadata:
  name: planespotter-frontend
  namespace: planespotter
  labels:
    app: planespotter-frontend
spec:
  ports:
    # the port that this service should serve on
    - port: 80
  selector:
    app: planespotter-frontend
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: planespotter-frontend
  namespace: planespotter
spec:
  rules:
  - host: afewell-cluster-69fc65f8-d37d-11e8-918b-0a1dada1e740.fa2c1d78-9f00-4e30-8268-4ab81862080d.vke-user.com
    http:
      paths:
      - backend:
          serviceName: planespotter-frontend
          servicePort: 80
```

</details>
<br/>

1.3.4 Run the updated planespotter-frontend app and verify deployment with the following commands. Make note of the external IP address/hostname shown in the output of `kubectl get services`

``` bash
kubectl create -f frontend-deployment_ingress.yaml
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get ingress
kubectl describe ingress
```

<details><summary>Screenshot 1.3.4</summary>
<img src="media/2018-10-20-16-11-14.png">
</details>

1.3.5 Open a browser and go to the url of your VKE SmartCluster to verify that planespotter-frontend is externally accessible with the LoadBalancer service

<details><summary>Screenshot 5.5.5</summary>
<img src="media/2018-10-20-16-26-46.png">
</details>
<br/>

1.3.6 Clean up the planespotter-frontend components and verify with the following commands:

``` bash
kubectl delete -f frontend-deployment_ingress.yaml
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get ingress
```

<details><summary>Screenshot 5.5.6</summary>
<img src="media/2018-10-20-16-32-19.png">
</details>
<br/>

## Next Steps

This lab provided an introductory overview of Kubernetes operations. Additional topics such as persistent volumes, network policy, config maps, stateful sets and more will be covered in more detail in the ongoing labs.

If you are following the PKS Ninja cirriculum, [click here to proceed to the next lab](../Lab2-PksInstallationPhaseOne). As you proceed through the remaining labs you will learn more advanced details about Kubernetes using additional planespotter app components as examples and then deploy the complete planespotter application on a PKS environment.

If you are not following the PKS Ninja cirriculum and would like to deploy the complete planespotter app on VKE, you can find [complete deployment instructions here](https://github.com/Boskey/run_kubernetes_with_vmware)

### Thank you for completing the Introduction to Kubernetes Lab!

### [Please click here to proceed to Lab2: PKS Installation Phase 1](../Lab2-PksInstallationPhaseOne)
