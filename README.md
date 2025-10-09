# Python Health App — CI/CD Plan

- Goal: Deploy a Python application with a fully automated CI/CD pipeline
- Flow: Push to GitHub → Jenkins → SonarQube scan → Trivy image scan → Docker deploy → Run on AWS EC2
- Repo: https://github.com/abirall/Python-App.git

---

## 1) Environment Setup

### Local machine

- Install Docker and Docker Compose
- Run SonarQube via Docker at [localhost:9000](http://localhost:9000)

![](attachment:33159165-ce84-4918-9755-c15935b915a5:image.png)

### Trivy installation

- Keep packages up to date
- Add Aqua Security signing key and repository
- Install Trivy and verify

```jsx
sudo apt-get update
sudo apt-get install wget apt-transport-https gnupg lsb-release
```

- `wget -qO -` [`](https://aquasecurity.github.io/trivy-repo/deb/public.key)https://aquasecurity.github.io/trivy-repo/deb/public.key` `| gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null`
- `echo "deb [signed-by=/usr/share/keyrings/trivy.gpg]` [`](https://aquasecurity.github.io/trivy-repo/deb)https://aquasecurity.github.io/trivy-repo/deb` `$(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list`

```jsx
sudo apt-get update
sudo apt-get install trivy
trivy --version
```

### AWS EC2 (t2.micro)

- Open ports 3000–10000
- Install Docker and Docker Compose
- Install Jenkins

![](attachment:4b6f483d-e3fa-46ba-a366-9f1af0c89ed6:image.png)

---

## 2) Jenkins and SonarQube Integration

### Required plugins

1. SonarQube Scanner
2. Docker
3. OWASP Dependency-Check
    - Name: dc
    - Install from [github.com](http://github.com)
    - Version: latest

### Configure SonarQube in Jenkins

- Manage Jenkins → System → SonarQube Servers
    - Name: Sonar
    - URL: [localhost:9000](http://localhost:9000)
    - Credentials: Secret text token from SonarQube UI

### Scanner installations

- Manage Jenkins → Tools → SonarQube Scanner installations
    - Provide a name
    - Enable “Install automatically”

### SonarQube UI configuration

- Webhook: Administration → Configuration → Webhooks
    - Name: Jenkins
    - URL: https://<EC2_IP:8080>/sonarqube-webhook/
- Token: Administration → Security → Users → Generate token
    - Name: Jenkins, set duration, copy token and save in Jenkins

Note: An initial attempt from the local PC failed due to network restrictions. After moving everything to the PC and stopping EC2, the setup worked as expected.

---

## 3) Pipeline

```groovy
 pipeline {
    agent any

    environment {
        SONAR_HOME = tool 'Sonar'
    }

    stages {
        stage('Git Code Clone') {
            steps {
                git url: 'https://github.com/abirall/Python-App.git', branch: 'main'
            }
        }

        stage('SonarQube Quality Analysis') {
            steps {
                withSonarQubeEnv('Sonar') {
                    sh "${SONAR_HOME}/bin/sonar-scanner -Dsonar.projectName=health_app -Dsonar.projectKey=health_app"
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Sonar Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Trivy File System Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }

        stage('Deploy With Docker') {
            steps {
                sh 'docker-compose up -d'
            }
        }

        stage('Final Check') {
            steps {
                echo 'check'
            }
        }
    }
}

```

![image.png](attachment:0556e50d-9551-47fe-bc17-00960a7f18fe:image.png)
