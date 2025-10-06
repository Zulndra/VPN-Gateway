# 🚀 Jenkins + WireGuard Deployment

Proyek ini menambahkan **Jenkins** ke dalam server VPN Gateway (WireGuard) untuk menjalankan **CI/CD pipeline** yang terhubung dengan **GitHub Webhook**.

---

## 📦 Persiapan Server

1. Pastikan sudah ada **Docker** & **Docker Compose** di server.

```bash
sudo apt update && sudo apt install -y docker.io docker-compose
```

2. Clone repository ini:

```bash
git clone https://github.com/<username>/<repo-name>.git
cd <repo-name>
```

## ⚙️ Deployment

Jalankan perintah berikut untuk build dan start container:

```bash
docker compose up --build -d
```

Jenkins UI akan berjalan di:
👉 http://<PUBLIC_IP>:8080

WireGuard berjalan sesuai konfigurasi di folder `config/`.

## 🔑 Setup Jenkins Pertama Kali

Akses Jenkins melalui browser:
👉 http://<PUBLIC_IP>:8080

Ambil password admin awal:

```bash
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Login dengan password tersebut.

### Install Plugin Utama

* GitHub Integration
* GitHub Branch Source
* Pipeline
* Docker Pipeline
* Blue Ocean (opsional, untuk UI modern)

## ⚙️ Konfigurasi di Jenkins Web

### Tambahkan GitHub Credential

Menu: Manage Jenkins → Credentials → (global) ➝ Add Credentials.

* Type: Username with password (Username = akun GitHub, Password = Personal Access Token)
* ID: github-credentials

### Tambahkan GitHub Server

Menu: Manage Jenkins → Configure System → GitHub Servers

* Server: [https://github.com/](https://github.com/)
* Credentials: github-credentials
* Klik Test Connection → harus berhasil

### Buat Pipeline Baru

* Klik New Item ➝ beri nama ➝ pilih Pipeline ➝ OK
* Pipeline script from SCM:

  * SCM: Git
  * Repository URL: [https://github.com/](https://github.com/)<username>/<repo-name>.git
  * Credentials: github-credentials
  * Branch: main
  * Script Path: Jenkinsfile
* Save

### Aktifkan GitHub Webhook Trigger

* Masuk ke job ➝ Configure → Build Triggers
* Centang: GitHub hook trigger for GITScm polling
* Save

## 🔗 Konfigurasi di GitHub

* Masuk ke repo GitHub ➝ Settings → Webhooks → Add webhook

  * Payload URL: http://<PUBLIC_IP>:8080/github-webhook/
  * Content type: application/json
  * Trigger: Just the push event
* Save, lalu klik Redeliver untuk tes
* Pastikan status delivery = 200 OK

## 🔄 Update Repository

* Masuk ke Jenkins UI → Pilih job → Configure → Ubah Repository URL → Save

## 🛑 Stop Jenkins & WireGuard

```bash
docker compose down
```

## 📖 Catatan

* Pastikan port 8080 dan 50000 terbuka di Security Group AWS.
* Gunakan IP Public AWS sebagai webhook URL.
* Repository wajib punya file Jenkinsfile.
* Dokumentasi Jenkins plugin: [Jenkins Plugins Index](https://plugins.jenkins.io/)
