Absolutely. Since you're practicing **Git + Docker + Jenkins Pipeline**, I recommend turning the `mypipeline` repository into a small but realistic **CI/CD hands-on project**.

The goal will be:

**Developer → GitHub → Jenkins → Checkout → Test → Docker Build → Docker Run → Verify**

Jenkins recommends keeping the pipeline definition in a `Jenkinsfile` inside source control, which is exactly what we'll practice here. ([Jenkins][1])

## 🚀 Hands-on Project: Git → Jenkins → Docker

Repository:

[https://github.com/atulkamble/mypipeline](https://github.com/atulkamble/mypipeline?utm_source=chatgpt.com)

### What you will learn

By completing this project, you'll practice:

1. Git repository integration
2. Jenkins Pipeline
3. Declarative Jenkinsfile
4. Jenkins stages
5. Environment variables
6. Shell commands from Jenkins
7. Docker image creation
8. Docker container execution
9. Docker image tagging
10. Automated application testing
11. Jenkins build parameters
12. Build failure handling
13. Docker cleanup
14. GitHub → Jenkins webhook
15. Docker Hub push
16. Credentials in Jenkins
17. CI/CD troubleshooting

---

# Project Architecture

```text
                    Developer
                        |
                        | git push
                        v
                +----------------+
                |    GitHub      |
                |   Repository   |
                +-------+--------+
                        |
                        | Webhook
                        v
                +----------------+
                |    Jenkins     |
                |                |
                |   Jenkinsfile  |
                +-------+--------+
                        |
              +---------+---------+
              |                   |
              v                   v
        Checkout Code        Run Tests
              |                   |
              +---------+---------+
                        |
                        v
                 Docker Build
                        |
                        v
                 Docker Image
                        |
                        v
                 Docker Run
                        |
                        v
                Application
```

Jenkins Pipeline can interact with Docker from a `Jenkinsfile` when the appropriate Pipeline/Docker Pipeline functionality is installed. ([Jenkins][2])

---

# Phase 1 — Prepare Jenkins Server

If you're using your AWS EC2 Jenkins server, first verify:

```bash
java -version
```

```bash
jenkins --version
```

```bash
git --version
```

```bash
docker --version
```

Then:

```bash
sudo systemctl status jenkins
sudo systemctl status docker
```

Both should be running.

### Give Jenkins permission to use Docker

```bash
sudo usermod -aG docker jenkins
```

Then:

```bash
sudo systemctl restart jenkins
```

Check:

```bash
id jenkins
```

You should see `docker` in the groups.

---

# Phase 2 — Create Your Own Project

I recommend **not modifying the original repository directly**.

Fork or create your own repository, for example:

```text
jenkins-docker-pipeline
```

Use this structure:

```text
jenkins-docker-pipeline/
│
├── app/
│   ├── app.py
│   └── requirements.txt
│
├── tests/
│   └── test_app.py
│
├── Dockerfile
├── Jenkinsfile
├── .dockerignore
├── .gitignore
└── README.md
```

This gives you a proper DevOps project rather than only a Jenkins test job.

---

# Phase 3 — Create Python Application

### `app/app.py`

```python
from flask import Flask

app = Flask(__name__)


@app.route("/")
def home():
    return "Jenkins Docker Pipeline is working!"


@app.route("/health")
def health():
    return "OK"


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

### `app/requirements.txt`

```text
Flask==3.1.2
```

---

# Phase 4 — Create Test

Create:

```text
tests/test_app.py
```

Put:

```python
from app.app import app


def test_home():
    client = app.test_client()
    response = client.get("/")
    assert response.status_code == 200


def test_health():
    client = app.test_client()
    response = client.get("/health")
    assert response.status_code == 200
```

Now Jenkins will actually have something to test.

---

# Phase 5 — Dockerfile

Create:

```text
Dockerfile
```

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY app/requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY app/ .

EXPOSE 5000

CMD ["python", "app.py"]
```

---

# Phase 6 — Docker Ignore

Create:

```text
.dockerignore
```

```text
.git
.gitignore
Jenkinsfile
tests
__pycache__
*.pyc
```

---

# Phase 7 — Test Docker Manually

Before involving Jenkins, test everything manually.

Clone your repository:

```bash
git clone https://github.com/YOUR_USERNAME/jenkins-docker-pipeline.git
```

Go inside:

```bash
cd jenkins-docker-pipeline
```

Build:

```bash
docker build -t jenkins-docker-app:1.0 .
```

Check:

```bash
docker images
```

Run:

```bash
docker run -d \
  --name jenkins-docker-app \
  -p 5000:5000 \
  jenkins-docker-app:1.0
```

Check:

```bash
docker ps
```

Test:

```bash
curl http://localhost:5000
```

Expected:

```text
Jenkins Docker Pipeline is working!
```

Health check:

```bash
curl http://localhost:5000/health
```

Expected:

```text
OK
```

---

# Phase 8 — Your First Jenkinsfile

Now comes the important part.

Create:

```text
Jenkinsfile
```

Start with this:

```groovy
pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Building application...'
                sh 'docker build -t jenkins-docker-app:${BUILD_NUMBER} .'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'docker run --rm jenkins-docker-app:${BUILD_NUMBER} python -m pytest'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker rm -f jenkins-docker-app || true

                    docker run -d \
                      --name jenkins-docker-app \
                      -p 5000:5000 \
                      jenkins-docker-app:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    sleep 5
                    curl -f http://localhost:5000/health
                '''
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }

        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }
    }
}
```

This gives you a real multi-stage pipeline.

Jenkins' documentation describes `Jenkinsfile` as the pipeline definition stored with the project in source control, providing a single source of truth and an audit trail. ([Jenkins][3])

---

# Phase 9 — Jenkins Job

Go to:

```text
Jenkins Dashboard
        ↓
New Item
```

Choose:

```text
Pipeline
```

Name:

```text
jenkins-docker-pipeline
```

Then:

```text
Pipeline
→ Definition
→ Pipeline script from SCM
```

Select:

```text
SCM: Git
```

Repository:

```text
https://github.com/YOUR_USERNAME/jenkins-docker-pipeline.git
```

Branch:

```text
*/main
```

Script Path:

```text
Jenkinsfile
```

Save.

Jenkins supports defining a Pipeline directly in the UI or loading a `Jenkinsfile` from SCM; the latter is generally recommended. ([Jenkins][4])

---

# Phase 10 — Run Pipeline

Click:

```text
Build Now
```

You should see:

```text
Checkout
   ↓
Build
   ↓
Test
   ↓
Run Container
   ↓
Verify
   ↓
SUCCESS
```

Your Jenkins console should contain something similar to:

```text
[Pipeline] stage
[Pipeline] { (Checkout)

[Pipeline] stage
[Pipeline] { (Build)

docker build ...

[Pipeline] stage
[Pipeline] { (Test)

Running tests...

[Pipeline] stage
[Pipeline] { (Run Container)

docker run ...

[Pipeline] stage
[Pipeline] { (Verify)

OK

Finished: SUCCESS
```

---

# Phase 11 — Important Jenkins Variables

Now modify your Jenkinsfile and learn these:

```groovy
echo "Build Number: ${BUILD_NUMBER}"
echo "Job Name: ${JOB_NAME}"
echo "Workspace: ${WORKSPACE}"
echo "Build URL: ${BUILD_URL}"
```

You can also define your own:

```groovy
environment {
    APP_NAME = "jenkins-docker-app"
    APP_PORT = "5000"
}
```

Then:

```groovy
echo "Application: ${APP_NAME}"
```

---

# Phase 12 — Dynamic Docker Tags

Instead of:

```bash
jenkins-docker-app:1.0
```

use:

```text
jenkins-docker-app:${BUILD_NUMBER}
```

For example:

```text
jenkins-docker-app:1
jenkins-docker-app:2
jenkins-docker-app:3
```

This is an important CI/CD concept because every Jenkins build gets a unique image version.

---

# Phase 13 — Add Docker Hub

Once your local pipeline works, move to:

```text
GitHub
   ↓
Jenkins
   ↓
Test
   ↓
Docker Build
   ↓
Docker Hub
```

Create a Docker Hub repository such as:

```text
YOUR_USERNAME/jenkins-docker-app
```

Then configure Jenkins credentials.

Go to:

```text
Manage Jenkins
    ↓
Credentials
    ↓
Global
    ↓
Add Credentials
```

Use:

```text
Kind: Username with password
```

ID:

```text
dockerhub-creds
```

Then your pipeline can authenticate and push the image.

For example:

```groovy
stage('Docker Push') {
    steps {
        withCredentials([
            usernamePassword(
                credentialsId: 'dockerhub-creds',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS'
            )
        ]) {

            sh '''
                echo "$DOCKER_PASS" | docker login \
                    -u "$DOCKER_USER" \
                    --password-stdin

                docker tag jenkins-docker-app:${BUILD_NUMBER} \
                    $DOCKER_USER/jenkins-docker-app:${BUILD_NUMBER}

                docker push \
                    $DOCKER_USER/jenkins-docker-app:${BUILD_NUMBER}
            '''
        }
    }
}
```

**Don't put Docker Hub passwords directly inside your Jenkinsfile.**

Jenkins has dedicated credential handling for Jenkinsfiles, and its documentation specifically covers credential usage and safe handling. ([Jenkins][3])

---

# Phase 14 — GitHub Webhook

After manual builds work, automate the trigger:

```text
Developer
    |
    | git push
    v
GitHub
    |
    | webhook
    v
Jenkins
    |
    v
Pipeline
```

Then:

```bash
git add .
git commit -m "Update application"
git push origin main
```

Jenkins should automatically start a build.

---

# 🔥 Hands-on Exercises

Don't just copy the pipeline. Do these yourself.

### Lab 1 — Basic Pipeline

Create:

```text
Checkout
Build
Test
```

---

### Lab 2 — Docker Pipeline

Add:

```text
Docker Build
Docker Run
```

---

### Lab 3 — Health Check

Add:

```bash
curl -f http://localhost:5000/health
```

Make Jenkins fail if the application isn't healthy.

---

### Lab 4 — Build Number

Tag images:

```text
app:1
app:2
app:3
```

using:

```groovy
${BUILD_NUMBER}
```

---

### Lab 5 — Failure Handling

Intentionally break:

```python
@app.route("/health")
```

Then push the change.

Observe:

```text
Test → FAILED
```

Then fix it.

---

### Lab 6 — Docker Hub

Implement:

```text
Jenkins
   ↓
docker build
   ↓
docker tag
   ↓
docker push
   ↓
Docker Hub
```

---

### Lab 7 — GitHub Webhook

Implement:

```text
git push
    ↓
GitHub webhook
    ↓
Jenkins
    ↓
Pipeline
```

---

### Lab 8 — Parameters

Add:

```groovy
parameters {
    choice(
        name: 'ENVIRONMENT',
        choices: ['dev', 'test', 'prod'],
        description: 'Select deployment environment'
    )
}
```

Then:

```groovy
echo "Deploying to ${params.ENVIRONMENT}"
```

---

### Lab 9 — Manual Approval

Add:

```groovy
stage('Approval') {
    steps {
        input message: 'Deploy to production?'
    }
}
```

Your pipeline becomes:

```text
Checkout
   ↓
Build
   ↓
Test
   ↓
Docker Build
   ↓
Docker Push
   ↓
Approval
   ↓
Deploy
```

---

# 🏆 Final Project

Once you've completed all labs, your final architecture should be:

```text
                  GitHub
                    |
                    | git push
                    v
              GitHub Webhook
                    |
                    v
              +-------------+
              |   Jenkins   |
              +------+------+
                     |
              +------+------+
              |             |
              v             v
          Checkout       Parameters
              |
              v
          Unit Tests
              |
              v
         Docker Build
              |
              v
         Docker Image
              |
              v
         Security/Test
              |
              v
         Docker Hub
              |
              v
        Manual Approval
              |
              v
          Deployment
              |
              v
       Running Container
              |
              v
         Health Check
```

## Skills you'll have covered

| Area                  | Hands-on |
| --------------------- | -------- |
| Git                   | ✅        |
| GitHub                | ✅        |
| Jenkins               | ✅        |
| Jenkinsfile           | ✅        |
| Declarative Pipeline  | ✅        |
| Stages                | ✅        |
| Environment variables | ✅        |
| Parameters            | ✅        |
| Unit testing          | ✅        |
| Docker build          | ✅        |
| Docker run            | ✅        |
| Docker tagging        | ✅        |
| Docker Hub            | ✅        |
| Jenkins credentials   | ✅        |
| Webhooks              | ✅        |
| Failure handling      | ✅        |
| Approval              | ✅        |
| Health checks         | ✅        |
| CI/CD                 | ✅        |

**Best approach:** do this in order rather than jumping straight to Docker Hub or webhooks. First get **Checkout → Build → Test → Docker Build → Docker Run → Verify** working locally in Jenkins. Then add Docker Hub and GitHub webhook automation.

If you're using the **Amazon Linux EC2 Jenkins machine you've been practicing with**, this project is especially suitable because it will also give you practice troubleshooting Jenkins's access to the Docker daemon—the same kind of issue that commonly causes `permission denied while trying to connect to the Docker API` errors. The Jenkins Docker documentation notes that Docker integration requires access to an actual Docker daemon; the Docker plugin itself does not provide that daemon. ([Jenkins Plugins][5])

[1]: https://www.jenkins.io/doc/book/pipeline/?utm_source=chatgpt.com "Pipeline"
[2]: https://www.jenkins.io/doc/book/pipeline/docker/?utm_source=chatgpt.com "Using Docker with Pipeline"
[3]: https://www.jenkins.io/doc/book/pipeline/jenkinsfile/?utm_source=chatgpt.com "Using a Jenkinsfile"
[4]: https://www.jenkins.io/doc/book/pipeline/getting-started/?utm_source=chatgpt.com "Getting started with Pipeline"
[5]: https://plugins.jenkins.io/docker-plugin?utm_source=chatgpt.com "Docker | Jenkins plugin"
