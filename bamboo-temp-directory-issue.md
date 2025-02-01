Issue: Bamboo Failed to Create Temp Directory
Description
The Bamboo Pod (bamboo-58d7ff8778-rmtml) failed to create the temporary directory at /var/atlassian/application-data/bamboo/temp. This issue prevented Bamboo from starting properly.


Error Message
"Bamboo failed to create the temp directory: /var/atlassian/application-data/bamboo/temp"

Root Cause
The issue was caused by one or more of the following:

Insufficient permissions: The Bamboo container did not have write permissions for the /var/atlassian/application-data/bamboo directory.

Missing directory: The temp directory did not exist, and Bamboo was unable to create it.

Storage issues: The PersistentVolumeClaim (PVC) might not have been properly provisioned or was out of storage space.

Steps to Reproduce
1.Deploy the Bamboo Pod using the provided Kubernetes manifest.

2.Check the Pod logs:
```
kubectl logs bamboo-58d7ff8778-rmtml -n bamboo
```
3.Observe the error message:
"Bamboo failed to create the temp directory: /var/atlassian/application-data/bamboo/temp"


<b>Solution</b<
To resolve the issue, follow these steps:

1. Verify PersistentVolumeClaim (PVC)
Ensure the PVC (bamboo-shared-home) is properly provisioned and bound:
```kubectl get pvc -n bamboo
```

2. Check Directory Permissions
Verify that the Bamboo container has write permissions for the /var/atlassian/application-data/bamboo directory:
- 1.Exec into the running Pod:
```kubectl exec -it bamboo-58d7ff8778-rmtml -n bamboo -- /bin/sh
```

- 2.Check the directory permissions:
```
ls -ld /var/atlassian/application-data/bamboo
```

- 3.If necessary, update the permissions:
```
chmod 777 /var/atlassian/application-data/bamboo
```

3. Create the temp Directory Manually
If the temp directory does not exist, create it manually:

```
kubectl exec -it bamboo-58d7ff8778-rmtml -n bamboo -- /bin/sh
mkdir -p /var/atlassian/application-data/bamboo/temp
chmod 777 /var/atlassian/application-data/bamboo/temp
```


4. Validate Storage Capacity
Ensure the PVC has sufficient storage space:

```
kubectl describe pvc bamboo-shared-home -n bamboo
```

5. Restart the Pod
Restart the Pod to apply the changes:
```
kubectl delete pod bamboo-58d7ff8778-rmtml -n bamboo
```
The ReplicaSet will automatically recreate the Pod.



6. Verify Fix
Check the Pod logs to confirm the issue is resolved:
```
kubectl logs bamboo-58d7ff8778-rmtml -n bamboo
```

Preventative Measures
To avoid this issue in the future:

1.Use an initContainer to create the temp directory and set permissions during Pod initialization:
```
initContainers:
  - name: init-bamboo
    image: busybox
    command: ["sh", "-c", "mkdir -p /var/atlassian/application-data/bamboo/temp && chmod 777 /var/atlassian/application-data/bamboo/temp"]
    volumeMounts:
      - name: bamboo-shared-home
        mountPath: /var/atlassian/application-data/bamboo

```


2.Ensure the PVC is configured with sufficient storage and correct permissions.


















