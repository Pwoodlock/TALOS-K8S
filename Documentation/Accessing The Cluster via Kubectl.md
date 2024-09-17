





# 1. Download your cluster Kubeconfig file

---

First we need to log into the Omni Dashboard and then click on the cluster you want to connect to.

Following that, look for the option to download Kubeconfg and download and place it in your 

$USER\.kube\ folder.



---

# 2. Download the required dependencies 

---



First off we need to download an install the Kubernetes CLI tool stack from chocolately or another source. I am going to use Chocolately

Open  a  PowerShell window as administrator and paste the following lines and select A or Y for any of the questions.


```c
choco install kubernetes-cli --version 1.29.1 -y
choco install kubernetes-helm --version 3.14.1 -y
choco install kubelogin --version 1.28.0 -y

```

After you have downloaded and installed you can now check to see if everything is working correctly by running the following command.



```c
kubectl version

```

The output should look something like the following:


```
Client Version: v1.29.1
Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
Server Version: v1.28.0

```



if you navigate via explorer to your users folder which should have a folder called .kube

ie. "C:\Users\Patrick\.kube\wazuh-staging-kubeconfig.yaml"    <===== This is the cluster configuration file!!

Place the Kubeconfig file in there and then in your terminal session, please use this command to link the Kubelogin and Kubectl to your configuration file. 


```c
$Env:KUBECONFIG = "C:\Users\Patrick\.kube\wazuh-staging-kubeconfig.yaml"
```


Now issue a any kubectl command , here is an example:


```c
kubectl get all nodes
```


Now you will have to login in your browser and return back to the terminal and from there you can issue your commands with the cluster.