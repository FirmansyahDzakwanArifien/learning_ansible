# Pengaturan Load Balancer  

---

## 1. Instalasi HAProxy di Control Node  
Jalankan perintah berikut di **control node**:  

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install haproxy -y
haproxy -v   # verifikasi instalasi
```

---

## 2. Instalasi Web Server di Managed Node  
Lakukan di setiap **managed node**:  

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install apache2 -y   # install apache/web server
systemctl status apache2      # verifikasi instalasi
```

Buat halaman web sederhana di setiap server (berbeda isi untuk identifikasi):  

```bash
echo "<h1>Server 1</h1>" | sudo tee /var/www/html/index.html
```

Untuk server kedua bisa diganti:  

```bash
echo "<h1>Server 2</h1>" | sudo tee /var/www/html/index.html
```

---

## 3. Konfigurasi HAProxy (1)  
Edit file konfigurasi HAProxy di **control node**:  

```bash
sudo nano /etc/haproxy/haproxy.cfg
```

Tambahkan konfigurasi berikut di bagian **frontend** dan **backend**:  

```haproxy
frontend http_front
   bind *:80
   default_backend http_back

backend http_back
   balance roundrobin
   server web1 <IP_MANAGED_NODE_1>:80 check
   server web2 <IP_MANAGED_NODE_2>:80 check
```

> ⚠️ Gantilah `<IP_MANAGED_NODE_1>` dan `<IP_MANAGED_NODE_2>` dengan alamat IP dari masing-masing managed node 

---

## 4. Konfigurasi HAProxy (2)  
1. Simpan dan tutup file dengan `CTRL + X`, kemudian `Y`, lalu tekan `Enter`.  
2. Restart layanan HAProxy:  

```bash
sudo systemctl restart haproxy
```

3. Verifikasi status HAProxy:  

```bash
sudo systemctl status haproxy
```

---

## 5. Uji Load Balancer  

### 1. Akses Load Balancer  
Buka browser dan masukkan **alamat IP dari control node** (yang menjalankan HAProxy).  
- Anda akan melihat halaman web yang ditampilkan oleh salah satu managed node.  
- Jika di-refresh, tampilan akan bergantian antara **Server 1** dan **Server 2** (round-robin).  

### 2. Periksa log HAProxy secara real-time  
```bash
sudo tail -f /var/log/haproxy.log
```

### 3. Cek koneksi dari control node ke managed node  
```bash
curl http://<IP_MANAGED_NODE_1>
curl http://<IP_MANAGED_NODE_2>
```

Jika berhasil, akan muncul isi halaman web dari masing-masing server.  

---

## 📌 Catatan  
- Load balancer memastikan distribusi trafik ke beberapa server untuk meningkatkan **ketersediaan** dan **redundansi**.  
- Pastikan firewall tidak memblokir port 80.  
- Gunakan algoritma **roundrobin** (default) untuk pembagian trafik secara merata.  
