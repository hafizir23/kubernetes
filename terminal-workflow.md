# Alur Kerja di Terminal

## 1. Login ke EC2

```bash
ssh -i key.pem ubuntu@PUBLIC-IP-EC2
```

## 2. Install K3s

```bash
sudo apt update
sudo apt install -y curl
curl -sfL https://get.k3s.io | sh -
sudo kubectl get nodes
```

## 3. Clone Repository

```bash
git clone https://github.com/USERNAME/sdgs-k8s-aws-free-tier.git
cd sdgs-k8s-aws-free-tier
```

## 4. Apply Manifest

```bash
sudo kubectl apply -f k8s/
```

## 5. Cek Resource

```bash
sudo kubectl get namespace
sudo kubectl get all -n sdg-greencampus
sudo kubectl get ingress -n sdg-greencampus
sudo kubectl get hpa -n sdg-greencampus
```

## 6. Uji Akses dari Server

```bash
curl http://localhost/
```

## 7. Uji Akses dari Browser

Buka:

```text
http://PUBLIC-IP-EC2/
```

## 8. Uji Load Balancing

```bash
sudo kubectl scale deployment greencampus-web --replicas=3 -n sdg-greencampus
for i in {1..20}; do curl -s http://PUBLIC-IP-EC2/ | grep HOSTNAME; done
```

## 9. Uji HPA

```bash
sudo kubectl get hpa -n sdg-greencampus
sudo kubectl top pods -n sdg-greencampus
```

Load generator:

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

Pantau:

```bash
sudo kubectl get hpa -n sdg-greencampus -w
sudo kubectl get pods -n sdg-greencampus -w
```

## 10. Hapus Resource Setelah Demo

```bash
sudo kubectl delete -f k8s/
```
