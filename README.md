
---

## 🚀 Jenkins Pipeline Job – Automating Docker Build & Run

### 🧠 What is a Jenkins Pipeline Job?
A **Jenkins pipeline job** defines a sequence of automated tasks like **building**, **testing**, and **deploying** code. Pipelines are written in a **Groovy-based DSL** and support **CI/CD** as code.

> ✅ Pipelines can be:
- **Declarative** (recommended, simpler syntax)
- **Scripted** (more flexible)

---

### 🛠️ Creating a Pipeline Job

1. From the Jenkins dashboard, click **“New Item”**
2. Enter a name like `My pipeline job`
3. Select **Pipeline** and click **OK**

---

### ⚙️ Configure Build Trigger (Webhook)

To automate builds on GitHub commits:

1. Click **Configure** on your job
2. Under **Build Triggers**, check:
   - **GitHub hook trigger for GITScm polling**
3. Ensure a **GitHub Webhook** is set up:
   - URL: `http://<jenkins-ip>:<port>/github-webhook/`
   - Content type: `application/json`
   - Trigger on `push` events

---

### ✏️ Writing the Jenkins Pipeline Script

```groovy
pipeline {
    agent any

    stages {
        stage('Connect To Github') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/YourUsername/jenkins-scm.git']]
                )
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t dockerfile .'
                }
            }
        }
        stage('Run Docker Container') {
            steps {
                script {
                    sh 'docker run -itd --name nginx -p 8081:80 dockerfile'
                }
            }
        }
    }
}
```

> ✅ Replace the GitHub URL with your own repo.  
> ✅ Ensure the branch name is correct (e.g., `main` or `master`).

---

### 🐳 Installing Docker on Jenkins Server

Create and run a shell script to install Docker:

**docker.sh**
```bash
#!/bin/bash
sudo apt-get update -y
sudo apt-get install ca-certificates curl gnupg -y
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo systemctl status docker
```

Make it executable:
```bash
chmod u+x docker.sh
./docker.sh
```

---

### 📦 Create Dockerfile & index.html in GitHub Repo

**dockerfile**
```Dockerfile
FROM nginx:latest
WORKDIR /usr/share/nginx/html/
COPY index.html /usr/share/nginx/html/
EXPOSE 80
```

**index.html**
```html
<!DOCTYPE html>
<html>
  <head><title>Success</title></head>
  <body>
    <h1>Congratulations, You have successfully run your first pipeline code.</h1>
  </body>
</html>
```

Push both files to the `main` branch. Jenkins should auto-trigger a build via the webhook.

---

### 🌐 Access the Running App

1. Update **inbound rules** in your cloud provider to allow TCP traffic on port **8081**
2. Visit:  
   `http://<jenkins-ip>:8081`

You’ll see your `index.html` content served via Nginx running in Docker!

---
