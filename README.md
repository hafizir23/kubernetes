# SDG GreenCampus Kubernetes di AWS Free Tier

Proyek ini adalah rancangan dan implementasi cluster Kubernetes untuk aplikasi bertema Sustainable Development Goals (SDGs). Aplikasi yang dipilih adalah **GreenCampus**, dashboard sederhana untuk edukasi dan pemantauan aksi hemat energi kampus.

Tema SDGs yang digunakan:

- **SDG 7: Affordable and Clean Energy**
- **SDG 11: Sustainable Cities and Communities**
- **SDG 13: Climate Action**

Platform:

- AWS Free Tier
- 1 EC2 Ubuntu `t2.micro` atau `t3.micro`
- Kubernetes ringan menggunakan K3s
- Traefik Ingress bawaan K3s
- Fallback Traefik ringan di `optional/traefik-simple.yaml` jika installer Traefik bawaan K3s gagal di instance kecil
- ServiceLB bawaan K3s
- Metrics Server bawaan K3s untuk HPA

> Catatan biaya: Amazon EKS memiliki biaya control plane per jam. Karena tugas memperbolehkan custom EC2 cluster, rancangan ini memakai K3s di EC2 supaya lebih cocok untuk Free Tier.

## Struktur Repository

```text
.
├── README.md
├── architecture.md
├── demo-script.md
├── terminal-workflow.md
└── k8s
    ├── 00-namespace.yaml
    ├── 01-configmap.yaml
    ├── 02-deployment.yaml
    ├── 03-service.yaml
    ├── 04-ingress.yaml
    └── 05-hpa.yaml
```

Folder opsional:

```text
optional/
└── traefik-simple.yaml
```

## Latar Belakang

Banyak lingkungan kampus belum memiliki cara sederhana untuk menampilkan edukasi hemat energi dan indikator kecil terkait aksi ramah lingkungan. GreenCampus dibuat sebagai aplikasi web ringan yang menampilkan pesan edukasi SDGs, target pengurangan emisi, dan ajakan tindakan hemat energi.

Aplikasi ini cocok untuk demonstrasi komputasi awan karena:

- Dapat dijalankan sebagai container stateless.
- Dapat direplikasi menjadi beberapa pod.
- Dapat diekspos ke internet melalui Ingress.
- Dapat diskalakan otomatis menggunakan Horizontal Pod Autoscaler.

## Arsitektur Singkat

Pengguna mengakses Public IP EC2 melalui HTTP port 80. Traffic masuk ke Traefik Ingress, diteruskan ke Service `greencampus-service`, lalu diseimbangkan ke beberapa Pod aplikasi.

Lihat diagram di [architecture.md](architecture.md).

## Persiapan AWS

1. Buat EC2 instance Ubuntu Server.
2. Pilih instance Free Tier, misalnya `t2.micro` atau `t3.micro`.
3. Buat Security Group dengan inbound rule:
   - SSH: TCP 22 dari IP pribadi kamu.
   - HTTP: TCP 80 dari `0.0.0.0/0`.
   - HTTPS: TCP 443 dari `0.0.0.0/0` jika ingin dikembangkan.
4. Login ke EC2 menggunakan SSH.

## Instalasi K3s di EC2

```bash
sudo apt update
sudo apt install -y curl
curl -sfL https://get.k3s.io | sh -
sudo kubectl get nodes
sudo kubectl get pods -A
```

K3s sudah membawa beberapa komponen penting, termasuk CoreDNS, Traefik, local-storage, metrics-server, dan ServiceLB.

## Deploy Aplikasi

Jalankan dari root repository:

```bash
sudo kubectl apply -f k8s/
sudo kubectl get all -n sdg-greencampus
sudo kubectl get ingress -n sdg-greencampus
```

Ambil Public IPv4 address EC2 dari AWS Console. Buka:

```text
http://PUBLIC-IP-EC2/
```

## Pengujian Load Balancing

Jalankan request berulang:

```bash
for i in {1..20}; do curl -s http://PUBLIC-IP-EC2/ | grep HOSTNAME; done
```

Jika jumlah replica lebih dari satu, hasil hostname akan berganti antar pod. Ini membuktikan Service melakukan load balancing ke beberapa pod.

## Pengujian HPA

Cek status awal:

```bash
sudo kubectl get hpa -n sdg-greencampus
sudo kubectl top pods -n sdg-greencampus
```

Buat beban trafik:

```bash
sudo kubectl run load-generator \
  --rm -i --tty \
  --image=busybox:1.36 \
  --restart=Never \
  -n sdg-greencampus \
  -- /bin/sh
```

Di dalam shell BusyBox:

```sh
while true; do wget -q -O- http://greencampus-service.sdg-greencampus.svc.cluster.local > /dev/null; done
```

Buka terminal SSH kedua, lalu pantau:

```bash
sudo kubectl get hpa -n sdg-greencampus -w
sudo kubectl get pods -n sdg-greencampus -w
```

Jika CPU meningkat, HPA akan menaikkan replica aplikasi sampai batas maksimal yang ditentukan.

## Pembersihan Resource

Supaya biaya AWS aman:

```bash
sudo kubectl delete -f k8s/
```

Setelah demo selesai, stop atau terminate EC2 instance dari AWS Console.

## Referensi Resmi

- AWS EKS Pricing: https://aws.amazon.com/eks/pricing/
- AWS Free Tier: https://aws.amazon.com/free/
- K3s Networking Services: https://docs.k3s.io/networking/networking-services
- K3s Packaged Components: https://docs.k3s.io/installation/packaged-components
