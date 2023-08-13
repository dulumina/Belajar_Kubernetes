langkah-langkah umum yang untuk menginstal Kubernetes menggunakan kubeadm dengan lingkungan :

`vm1 = kube-lb-01 (load balance menggunkan haproxy) ip=192.168.56.31`

`vm2 = kube-master-01 (kube master) ip=192.168.56.11`

`vm3 = kube-master-02 (kube master) ip=192.168.56.12`

`vm4 = kube-worker-01 (kube worker) ip=192.168.56.21`


**Langkah 1: Instalasi di Semua Node**

1.  Pastikan Anda telah menginstal sistem operasi yang sesuai (misalnya, Ubuntu atau CentOS) pada setiap VM.
2.  Update sistem dan instal dependensi yang diperlukan:
        
    ```
    sudo apt update
    ```
    ```
    sudo apt install -y apt-transport-https curl
    ```
    
3.  Tambahkan repository Kubernetes dan instal kubeadm, kubelet, dan kubectl:
        
    ```
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    sudo sh -c 'echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list'
    sudo apt update
    sudo apt install -y kubeadm kubelet kubectl
    ```
    
4.  Nonaktifkan swap space:
    
    ```
    sudo swapoff -a
    sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
    ```
    

**Langkah 2: Konfigurasi Load Balancer (kube-lb-01)**

1.  Pada VM kube-lb-01, instal HAProxy:
        
    ```
    sudo apt install -y haproxy
    ```
    
2.  Konfigurasi HAProxy (`/etc/haproxy/haproxy.cfg`) untuk meneruskan lalu lintas ke master node:
        
    ```
    frontend k8s-frontend
        bind 192.168.56.31:6443
        mode tcp
        default_backend k8s-backend

    backend k8s-backend
        mode tcp
        balance roundrobin
        server kube-master-01 192.168.56.11:6443 check
        server kube-master-02 192.168.56.12:6443 check
    ```
    
3.  Restart HAProxy:
        
    ```
    sudo service haproxy restart
    ```
    

**Langkah 3: Inisialisasi Master Nodes (kube-master-01 dan kube-master-02)**

1.  Pada setiap node master (kube-master-01 dan kube-master-02), inisialisasi Kubernetes menggunakan kubeadm di salah satu node:
    
    ```
    sudo kubeadm init --control-plane-endpoint "192.168.56.31:6443" --upload-certs --apiserver-advertise-address=192.168.56.11
    ```
    
    Simpan perintah `kubeadm join` yang dihasilkan.
    
2.  Setelah inisialisasi selesai, ikuti instruksi yang ditampilkan untuk mengatur konfigurasi kubectl.
    
3.  Bergabungkan node lain ke dalam cluster menggunakan perintah yang disimpan dari langkah sebelumnya:
    
    ```
    sudo kubeadm join ...
    ```
    

**Langkah 4: Tambahkan Node Worker (kube-worker-01)**

1.  Pada node worker (kube-worker-01), bergabungkan dengan cluster menggunakan perintah yang Anda simpan sebelumnya:
    
    ```
    sudo kubeadm join ...
    ```
    

**Langkah 5: Verifikasi Cluster**

1.  Kembali ke salah satu node master, verifikasi status cluster:
        
    ```
    kubectl get nodes kubectl get pods -A
    ```
    

Sekarang Anda memiliki klaster Kubernetes yang dijalankan di lingkungan VirtualBox Anda. Pastikan Anda memahami bahwa ini hanya panduan dasar, dan ada banyak hal lain yang dapat dikonfigurasi dan ditingkatkan dalam klaster Kubernetes Anda, seperti menyebarkan Pods, pengaturan jaringan, penyimpanan, dan lainnya.
