### **Docker Compose for an Nginx Server**

- Logs are stored on the host machine in the `./nginx/logs` folder, ensuring persistence across container restarts.
- A custom bridge network is created with a specified subnet `172.20.8.0/24`, and the Nginx container is assigned a static IP of `172.20.8.7`.
- The `restart: always` policy ensures the container automatically restarts in case of failure.

### **Output**
![Screenshot](https://github.com/neehadumar/DevOpsCaseStudy/blob/main/docker-kubernetes/Screenshot%202025-03-23%20at%2019.46.20.png)
---

### **Kubernetes Command to Identify Pod Restart Reason**
To determine the reason for a pod restart in the "internal" project within the "production" namespace, use the following command:

```sh
kubectl describe pod <pod-name> -n production
```

---

### **Possible Reasons for Java-App Pod Restarts**
Given the resource quota allocated to the Java application, restarts may occur due to the following reasons:

#### **1. Memory Limit Exceeded**
- The pod's memory limit is set to **1500MiB**, while the Java application's maximum heap size (`-Xmx`) is **1000MiB**.
- This leaves insufficient space for non-heap memory (e.g., Metaspace, threads, native memory), potentially causing Out of Memory (OOM) errors and triggering pod restarts.

#### **2. CPU Resource Throttling**
- The pod has a CPU limit of **2000 millicores (2 cores)**.
- If the application demands more CPU, Kubernetes will throttle it, leading to performance degradation, failed health checks, or timeouts, which may cause restarts.

To investigate memory or CPU-related issues, run:

```sh
kubectl describe pod java-app-7d9d44ccbf-lmvbc -n production
```

---

### **Resolving Memory Limit Issues**

#### **Increase the Pod’s Memory Limit**
If feasible, increase the memory allocation to provide sufficient headroom for both heap and non-heap memory. Since the Java application is set to `-Xmx 1000M`, you should consider increasing the pod’s memory limit to **2000Mi or 2500Mi**.

Update the pod’s resource specification:

```yaml
resources:
  requests:
    memory: "1000Mi"
  limits:
    memory: "2500Mi"
```

#### **Adjust Java Memory Settings (`-Xmx` and `-Xms`)**
If increasing the pod’s memory limit is not an option, modify the `-Xmx` value to balance heap and non-heap memory allocation. Lowering `-Xmx` to **800M** could ensure sufficient space for other memory needs, reducing the likelihood of OOM errors.

By implementing these adjustments, you can enhance the stability of the Java application within the Kubernetes environment.

