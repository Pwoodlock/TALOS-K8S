

# Getting the status of Pods and Name Spaces 

---

Some generic commands for normal administration of the cluster.  Don't forget that most of the administration is done via Omni Dashboard regarding updating the infrastructure and machine configs etc.   I recommend that you do Talos OS and Kubernetes Updates via that or the Omnictl. 



```
kubectl describe pod wazuh-manager-0 -n <namespace>
kubectl describe pod wazuh-worker-0 -n <namespace>

```


