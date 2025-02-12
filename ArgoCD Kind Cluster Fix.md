# **ArgoCD Multi-Cluster Setup - Issues & Solutions**

## **Issue 1: `argocd cluster add cluster2` - Context Not Found**
### **Error Message:**
```bash
FATA[0002] Context cluster2 does not exist in kubeconfig
```
### **Cause:**
The context `cluster2` does not exist in the `kubeconfig` file.

### **Solution:**
1. **Check Available Contexts:**
   ```bash
   kubectl config get-contexts
   ```
   **Output:**
   ```
   CURRENT   NAME            CLUSTER         AUTHINFO        NAMESPACE
   *         kind-cluster1   kind-cluster1   kind-cluster1   
             kind-cluster2   kind-cluster2   kind-cluster2   
   ```
   
2. **Use the Correct Context Name:**
   The correct context name is `kind-cluster2`, so use it in ArgoCD:
   ```bash
   argocd cluster add kind-cluster2
   ```

---

## **Issue 2: Connection Refused When Adding Cluster to ArgoCD**
### **Error Message:**
```bash
FATA[0005] rpc error: code = Unknown desc = error getting server version: Get "https://127.0.0.1:38833/version?timeout=32s": dial tcp 127.0.0.1:38833: connect: connection refused
```

### **Cause:**
Kind uses a local Docker network, and `127.0.0.1:38833` is only accessible from the local machine, not from inside the ArgoCD pod.

### **Solution:**
1. **Check the API Server Address for `kind-cluster2`:**
   ```bash
   kubectl config view --minify --context=kind-cluster2 -o jsonpath='{.clusters[0].cluster.server}'
   ```
   **Output:**
   ```
   https://127.0.0.1:38833
   ```
   
2. **Find the Internal IP of `kind-cluster2`'s Control Plane:**
   ```bash
   docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' cluster2-control-plane
   ```
   **Output:**
   ```
   172.18.0.2
   ```
   
3. **Update Kubeconfig to Use Internal IP:**
   ```bash
   kubectl config set-cluster kind-cluster2 --server=https://172.18.0.2:6443
   ```

4. **Verify the Update:**
   ```bash
   kubectl config view --minify --context=kind-cluster2
   ```
   Ensure that the server address is now `https://172.18.0.2:6443`.

5. **Try Adding the Cluster Again:**
   ```bash
   argocd cluster add kind-cluster2
   ```

6. **Verify the Connection:**
   ```bash
   argocd cluster list
   ```
   This should list `kind-cluster2` as a managed cluster in ArgoCD.

---

## **Summary of Fixes**
| Issue | Cause | Solution |
|-------|-------|----------|
| `Context cluster2 does not exist in kubeconfig` | Context name was incorrect | Use `kind-cluster2` instead of `cluster2` |
| `Connection refused (127.0.0.1:38833)` | Kind cluster uses local IP | Replace `127.0.0.1` with the internal Docker IP (e.g., `172.18.0.2`) |
| Unable to add cluster to ArgoCD | ArgoCD cannot reach Kind API server | Update kubeconfig with the internal IP and retry `argocd cluster add` |

Now that these issues are resolved, you can proceed with deploying a demo application in ArgoCD.

