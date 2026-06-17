# Script Demo Video

Durasi ideal: 5 sampai 8 menit.

## 1. Pembukaan

Perkenalkan proyek:

> Pada demo ini saya membuat cluster Kubernetes di AWS Free Tier menggunakan EC2 dan K3s. Aplikasi yang dihosting bernama GreenCampus, yaitu aplikasi edukasi hemat energi yang berkaitan dengan SDG 7, SDG 11, dan SDG 13.

## 2. Jelaskan Arsitektur

Tampilkan file `architecture.md`.

Poin yang dijelaskan:

- User mengakses Public IP EC2.
- Traffic masuk melalui Traefik Ingress.
- Service membagi request ke beberapa pod.
- ConfigMap menyimpan teks tema SDGs.
- HPA menambah pod ketika CPU naik.

## 3. Tampilkan Manifest Kubernetes

Tampilkan folder `k8s`.

Jelaskan singkat:

- `00-namespace.yaml`: namespace khusus aplikasi.
- `01-configmap.yaml`: konfigurasi tema SDGs.
- `02-deployment.yaml`: menjalankan container aplikasi web Python dan resource limit.
- `03-service.yaml`: load balancing internal.
- `04-ingress.yaml`: akses HTTP dari luar.
- `05-hpa.yaml`: auto-scaling berdasarkan CPU.

## 4. Deploy ke Cluster

Jalankan:

```bash
sudo kubectl apply -f k8s/
sudo kubectl get all -n sdg-greencampus
sudo kubectl get ingress -n sdg-greencampus
```

Jelaskan bahwa semua resource sudah berjalan.

## 5. Uji Aplikasi

Buka browser:

```text
http://PUBLIC-IP-EC2/
```

Tunjukkan halaman GreenCampus.

## 6. Uji Load Balancing

Scale manual sebentar agar hostname lebih mudah terlihat:

```bash
sudo kubectl scale deployment greencampus-web --replicas=3 -n sdg-greencampus
sudo kubectl get pods -n sdg-greencampus -o wide
```

Jalankan:

```bash
for i in {1..20}; do curl -s http://PUBLIC-IP-EC2/ | grep HOSTNAME; done
```

Jelaskan bahwa hostname yang berbeda menunjukkan request dibagi ke beberapa pod.

## 7. Uji HPA

Tampilkan HPA:

```bash
sudo kubectl get hpa -n sdg-greencampus
```

Buat traffic:

```bash
sudo kubectl run load-generator \
  --rm -i --tty \
  --image=busybox:1.36 \
  --restart=Never \
  -n sdg-greencampus \
  -- /bin/sh
```

Di dalam BusyBox:

```sh
while true; do wget -q -O- http://greencampus-service.sdg-greencampus.svc.cluster.local > /dev/null; done
```

Terminal kedua:

```bash
sudo kubectl get hpa -n sdg-greencampus -w
sudo kubectl get pods -n sdg-greencampus -w
```

Jelaskan bahwa jumlah pod bertambah otomatis saat beban naik.

## 8. Penutup

Sampaikan:

> Kesimpulannya, aplikasi GreenCampus berhasil dihosting di Kubernetes pada AWS Free Tier. Cluster sudah memiliki Deployment, Service, Ingress, ConfigMap, dan HPA. Pengujian menunjukkan aplikasi dapat diakses dari internet, trafik dibagi ke beberapa pod, dan replica dapat bertambah otomatis saat beban meningkat.
