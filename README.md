# Sample Backend Golang dengan Echo dan Load Balancer Nginx

## 1. Deskripsi

Proyek ini adalah backend API yang dibuat menggunakan **Golang** dan **Echo Framework**. API ini memiliki dua endpoint utama:

- **GET /user** → Mengembalikan data user dalam format JSON.
- **POST /user/create** → Menerima JSON payload dari Postman atau klien lain dan mengembalikan respons.

Untuk meningkatkan ketersediaan dan performa, kita menggunakan **Nginx sebagai Load Balancer** untuk mendistribusikan request ke beberapa instance backend.

---

## 2. Instalasi

### **A. Instalasi Nginx**

#### **1. Instalasi di Ubuntu**

```sh
sudo apt update && sudo apt install nginx -y
```

#### **2. Instalasi di macOS**

```sh
brew install nginx
```

### **B. Konfigurasi Load Balancer dengan Nginx**

Edit file konfigurasi Nginx:

```sh
sudo vi /usr/local/etc/nginx/nginx.conf  # macOS
sudo vi /etc/nginx/nginx.conf   # Ubuntu
```

Tambahkan konfigurasi berikut di dalam blok `http`:

```nginx
http {
    upstream golang_backend {
        server 127.0.0.1:8081;
        server 127.0.0.1:8082;
        server 127.0.0.1:8083;
    }

    server {
        listen 8080;
        server_name localhost;

        location / {
            proxy_pass http://golang_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

### **Restart Nginx untuk Menerapkan Perubahan**

```sh
sudo nginx -s reload  # Jika sudah berjalan
sudo systemctl restart nginx  # Untuk Ubuntu
brew services restart nginx  # Untuk macOS
```

---

## 3. Instalasi Golang dan Echo Framework

### **A. Instalasi Golang**

#### **1. Di Ubuntu**

```sh
sudo apt update && sudo apt install golang -y
```

Tambahkan konfigurasi Golang di `~/.profile` atau `~/.bashrc`:

```sh
echo 'export GOPATH=$HOME/go' >> ~/.profile
echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.profile
source ~/.profile
```

#### **2. Di macOS**

```sh
brew install go
```

Tambahkan konfigurasi Golang di `~/.bash_profile` atau `~/.zshrc`:

```sh
echo 'export GOPATH=$HOME/go' >> ~/.bash_profile
echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.bash_profile
source ~/.bash_profile
```

Untuk memastikan Golang sudah terinstal:

```sh
go version
```

### **B. Clone Repository Proyek Echo**

Untuk memulai, clone repository proyek ini menggunakan Git:

```sh
git clone https://github.com/kusnadi8605/nginx_configuration
cd nginx_configuration
```

### **C. Instalasi Echo Framework**

```sh
go get -u github.com/labstack/echo/v4
```

### **D. Inisialisasi Modul Golang**

```sh
go mod init nginx_configuration
go mod tidy
go mod vendor
```

---

## 4. Menjalankan Backend Golang

### **A. Menjalankan Backend Secara Lokal**

Buka terminal dan jalankan perintah berikut satu per satu di terminal yang berbeda untuk menjalankan beberapa instance server:

```sh
PORT=8081 go run main.go &
PORT=8082 go run main.go &
PORT=8083 go run main.go &
```

Server akan berjalan di `http://localhost:8081/user` dan `http://localhost:8081/user/create`.

---

## 5. Testing API dengan Curl

### **A. GET Request**

```sh
curl -X GET http://localhost:8080/user
```

### **B. POST Request**

```sh
curl -X POST http://localhost:8080/user/create \
     -H "Content-Type: application/json" \
     -d '{"id": 2, "name": "Alice", "email": "alice@example.com", "age": 25}'
```

### **C. Mengirim Request 10x dengan Curl**

#### **1. GET Request 10x**

```sh
for i in {1..10}; do curl -X GET http://localhost:8080/user; done
```

#### **2. POST Request 10x**

```sh
for i in {1..10}; do
  curl -X POST http://localhost:8080/user/create \
       -H "Content-Type: application/json" \
       -d "{\"id\": $i, \"name\": \"Alice$i\", \"email\": \"alice$i@example.com\", \"age\": 25}"
done
```

---

## 6. Troubleshooting

### **A. Error: invalid PID number "" in nginx.pid**

#### **Solusi untuk Ubuntu dan macOS:**

1. Hapus file **nginx.pid** yang corrupt:
   ```sh
   sudo rm -f /usr/local/var/run/nginx.pid  # macOS
   sudo rm -f /run/nginx.pid  # Ubuntu
   ```
2. Restart Nginx:
   ```sh
   sudo nginx
   ```

### **B. Error: Permission denied pada error.log atau access.log**

#### **Solusi untuk macOS:**

```sh
sudo chown -R $(whoami) /usr/local/var/log/nginx/
sudo chmod -R 755 /usr/local/var/log/nginx/
```

#### **Solusi untuk Ubuntu:**

```sh
sudo chown -R www-data:www-data /var/log/nginx/
sudo chmod -R 755 /var/log/nginx/
```

### **C. Cek Status Backend**

```sh
lsof -i :8081
lsof -i :8082
lsof -i :8083
```

Jika tidak ada proses yang berjalan, jalankan ulang backend.

### **D. Cek Log Error Nginx**

```sh
cat /usr/local/var/log/nginx/error.log  # macOS
cat /var/log/nginx/error.log  # Ubuntu
```

