# Arsitektur Cluster

## Diagram

```mermaid
flowchart LR
    U["User / Browser"] --> SG["AWS Security Group<br/>Port 80 HTTP"]
    SG --> EC2["EC2 Ubuntu Free Tier<br/>Public IPv4"]
    EC2 --> K3S["K3s Kubernetes Cluster"]
    K3S --> ING["Traefik Ingress Controller"]
    ING --> SVC["Service<br/>greencampus-service"]
    SVC --> P1["Pod 1<br/>GreenCampus App"]
    SVC --> P2["Pod 2<br/>GreenCampus App"]
    SVC --> P3["Pod N<br/>Auto-scaled by HPA"]
    CM["ConfigMap<br/>Tema SDGs"] --> P1
    CM --> P2
    CM --> P3
    MS["Metrics Server"] --> HPA["Horizontal Pod Autoscaler"]
    HPA --> P1
    HPA --> P2
    HPA --> P3
```

## Penjelasan Komponen

1. **EC2 Ubuntu Free Tier**  
   Menjadi server utama tempat Kubernetes ringan K3s berjalan.

2. **K3s Cluster**  
   Distribusi Kubernetes ringan yang cocok untuk instance kecil. K3s sudah menyediakan Traefik Ingress, ServiceLB, dan metrics-server.

3. **Deployment**  
   Menjalankan aplikasi GreenCampus dalam beberapa replica pod.

4. **ConfigMap**  
   Menyimpan konfigurasi aplikasi, seperti nama aplikasi, tema SDGs, dan pesan edukasi.

5. **Service**  
   Menyediakan alamat internal stabil untuk pod dan melakukan load balancing antar replica.

6. **Ingress**  
   Mengekspos aplikasi ke internet melalui HTTP port 80 menggunakan Traefik.

7. **HPA**  
   Mengatur jumlah pod otomatis berdasarkan penggunaan CPU.

## Alur Request

1. User membuka `http://PUBLIC-IP-EC2/`.
2. Request masuk ke Security Group AWS port 80.
3. Request diterima Traefik Ingress di cluster K3s.
4. Traefik meneruskan request ke Service.
5. Service membagi trafik ke pod aplikasi yang tersedia.
6. Jika beban CPU meningkat, HPA menambah jumlah replica.
