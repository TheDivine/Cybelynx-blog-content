![k8s the hard way](/kubernetes.webp)

Here's a comprehensive **checklist for setting up a Kubernetes Master Node**. Follow this step-by-step process to ensure a smooth setup:

---

### **Preliminary Setup**

1. **System Requirements:**
   - OS: Ubuntu 20.04+, CentOS 7+, Debian 10+.
   - CPU: Minimum 2 cores.
   - RAM: At least 2 GB (4 GB recommended).
   - Disk: Minimum 10 GB free space.
   - Network: Stable connectivity with a static or reserved IP address.

2. **Disable Swap:**
   ```bash
   sudo swapoff -a
   sudo sed -i '/ swap / s/^/#/' /etc/fstab
   ```

3. **Install Required Packages:**
   - **Update system:**
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```
   - **Install dependencies:**
     ```bash
     sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
     ```

4. **Configure Kernel Modules:**
   ```bash
   sudo modprobe overlay
   sudo modprobe br_netfilter
   ```

   - Persist the modules:
     ```bash
     echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/k8s.conf
     ```

5. **Set Kernel Parameters for Kubernetes:**
   Add the following to `/etc/sysctl.d/k8s.conf`:
   ```bash
   net.bridge.bridge-nf-call-iptables = 1
   net.bridge.bridge-nf-call-ip6tables = 1
   net.ipv4.ip_forward = 1
   ```
   Apply the changes:
   ```bash
   sudo sysctl --system
   ```

---

### **Install Docker and Container Runtime**

6. **Install Docker (Optional if using containerd):**
   - Add Docker’s official GPG key:
     ```bash
     curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
     ```
   - Add the Docker repository:
     ```bash
     sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
     ```
   - Install Docker:
     ```bash
     sudo apt install -y docker-ce docker-ce-cli containerd.io
     ```

7. **Install containerd (Preferred for Kubernetes):**
   - Install:
     ```bash
     sudo apt install -y containerd
     ```
   - Configure containerd:
     ```bash
     sudo mkdir -p /etc/containerd
     containerd config default | sudo tee /etc/containerd/config.toml
     ```
   - Enable `SystemdCgroup` in `/etc/containerd/config.toml`:
     ```toml
     SystemdCgroup = true
     ```
   - Restart containerd:
     ```bash
     sudo systemctl restart containerd
     ```

---

### **Install Kubernetes Components**

8. **Add Kubernetes Repository:**
   ```bash
   curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
   sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
   ```

9. **Install kubeadm, kubelet, and kubectl:**
   ```bash
   sudo apt install -y kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
   ```

10. **Check Installed Versions:**
    ```bash
    kubeadm version
    kubectl version --client
    kubelet --version
    ```

---

### **Initialize the Kubernetes Cluster**

11. **Initialize Master Node:**
    ```bash
    kubeadm init --apiserver-advertise-address=<STATIC_IP> --pod-network-cidr=10.244.0.0/16
    ```
    Replace `<STATIC_IP>` with the master node’s IP.

12. **Set Up kubeconfig for `kubectl`:**
    ```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```

13. **Verify Node Status:**
    ```bash
    kubectl get nodes
    ```

---

### **Install Pod Network**

14. **Deploy a Network Add-on (Choose One):**
    - **Flannel:**
      ```bash
      kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
      ```
    - **Calico:**
      ```bash
      kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
      ```

15. **Verify Pod Network Setup:**
    ```bash
    kubectl get pods --all-namespaces
    ```

---

### **Post-Installation Setup**

16. **Ensure Node Is Ready:**
    Confirm the master node is in the `Ready` state:
    ```bash
    kubectl get nodes
    ```

17. **Enable Scheduling on Master Node (Optional for Single Node Cluster):**
    ```bash
    kubectl taint nodes --all node-role.kubernetes.io/master-
    ```

18. **Backup Cluster Configurations:**
    Save the `kubeadm init` join command for adding worker nodes later.

19. **Check API Server Logs:**
    ```bash
    journalctl -u kubelet -f
    ```

---

### **Summary Checklist**

| Step | Command/Description | Status |
|---|---|---|
| Disable Swap | `swapoff -a` | ✅ |
| Install Docker/Containerd | Install and configure container runtime | ✅ |
| Load Kernel Modules | Configure overlay, br_netfilter, etc. | ✅ |
| Install Kubernetes Components | kubeadm, kubelet, kubectl | ✅ |
| Initialize Master Node | `kubeadm init` | ✅ |
| Configure kubectl | Copy and set permissions for kubeconfig | ✅ |
| Install Pod Network | Apply Flannel or Calico | ✅ |
| Verify Cluster | `kubectl get nodes` and `kubectl get pods --all-namespaces` | ✅ |

Following this checklist will ensure your Kubernetes master node is correctly set up and operational. Let me know if you encounter any issues!
