# 🚀 Jenkins + WireGuard Deployment

This project integrates **Jenkins** into a VPN Gateway (WireGuard) server to enable a **CI/CD pipeline** connected with a **GitHub Webhook**.

---

## 📦 Server Preparation

1. Ensure that **Docker** & **Docker Compose** are installed on the server.

```bash
sudo apt update && sudo apt install -y docker.io docker-compose
```

2. Clone this repository:

```bash
git clone https://github.com/<username>/<repo-name>.git
cd <repo-name>
```

## ⚙️ Deployment

Run the following command to build and start the containers:

```bash
docker compose up --build -d
```

Jenkins UI will be available at:  
👉 http://<PUBLIC_IP>:8080

WireGuard will run according to the configuration in the `config/` folder.

## 🔑 Initial Jenkins Setup

Access Jenkins via your browser:  
👉 http://<PUBLIC_IP>:8080

Get the initial admin password:

```bash
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Login using that password.

### Install Main Plugins

* GitHub Integration  
* GitHub Branch Source  
* Pipeline  
* Docker Pipeline  
* Blue Ocean (optional, for a modern UI)

## ⚙️ Jenkins Web Configuration

### Add GitHub Credentials

Navigate to: Manage Jenkins → Credentials → (global) ➝ Add Credentials.

* Type: Username with password (Username = your GitHub account, Password = Personal Access Token)
* ID: github-credentials

### Add GitHub Server

Navigate to: Manage Jenkins → Configure System → GitHub Servers

* Server: [https://github.com/](https://github.com/)
* Credentials: github-credentials  
* Click **Test Connection** → should be successful

### Create a New Pipeline

* Click **New Item** ➝ give it a name ➝ choose **Pipeline** ➝ OK  
* Under **Pipeline script from SCM**:
  * SCM: Git  
  * Repository URL: [https://github.com/](https://github.com/)<username>/<repo-name>.git  
  * Credentials: github-credentials  
  * Branch: main  
  * Script Path: Jenkinsfile  
* Save

### Enable GitHub Webhook Trigger

* Go to the job ➝ Configure → Build Triggers  
* Check: **GitHub hook trigger for GITScm polling**  
* Save

## 🔗 GitHub Configuration

* Go to your GitHub repository ➝ **Settings → Webhooks → Add webhook**
  * Payload URL: http://<PUBLIC_IP>:8080/github-webhook/  
  * Content type: application/json  
  * Trigger: Just the push event  
* Save, then click **Redeliver** to test  
* Ensure delivery status = **200 OK**

## 🔄 Update Repository

* Go to Jenkins UI → Select your job → Configure → Update Repository URL → Save

## 🛑 Stop Jenkins & WireGuard

```bash
docker compose down
```

## 📖 Notes

* Make sure ports **8080** and **50000** are open in your AWS Security Group.  
* Use your AWS Public IP as the webhook URL.  
* The repository must include a `Jenkinsfile`.  
* Jenkins plugin documentation: [Jenkins Plugins Index](https://plugins.jenkins.io/)
