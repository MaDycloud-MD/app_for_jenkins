# Jenkins Pipeline for Automating React App Deployment on AWS using Docker

## Objective
The goal of this setup was to automate the build and deployment process of a React application using Jenkins and Docker. All tasks were executed on AWS cloud infrastructure.

---

## Tools Used
- **Jenkins** (running on an EC2 instance)
- **Docker** (for containerizing the app)
- **GitHub** (code repository)
- **AWS EC2** (to host Jenkins and the deployed app)

---

## Step-by-Step Process

### 1. Setting Up Jenkins on AWS EC2
- Launched an Ubuntu EC2 instance.
- Updated system packages and installed Java (required by Jenkins).
- Installed Jenkins via apt repository.
- Enabled port `8080` in AWS security group to access Jenkins UI.
- Retrieved Jenkins initial admin password and completed setup.
- Installed recommended plugins and created a new admin user.

### 2. Installing Docker on EC2
- Installed Docker CE using official Docker instructions.
- Added the Jenkins user to the `docker` group to allow Docker access without sudo.
- Verified Docker was working by running `docker run hello-world`.

### 3. Configuring GitHub Repository
- Pushed the demo React app to a public GitHub repository.
- Ensured the branch name was `main` (Jenkins default matches this).
- In Jenkins, created a **New Item → Pipeline** and linked it to the GitHub repo using the HTTPS URL.

### 4. Writing the `Jenkinsfile`
In the root of the GitHub repo, I created the following `Jenkinsfile`:

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'madycloudmd/react-demo-app'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: '< repo-link >'
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                      echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                      docker push $DOCKER_IMAGE
                    '''
                }
            }
        }
    }
}

```

## Webhook Integration (Trigger on Git Push)

To automate the Jenkins pipeline to trigger on every Git push, I integrated a GitHub webhook into my Jenkins setup. This allowed Jenkins to be notified instantly whenever new code is pushed to the repository, initiating the pipeline automatically.

### Steps I Followed:

1. **Configured GitHub Webhook**  
   - Went to my GitHub repository → **Settings** → **Webhooks** → **Add webhook**.
   - Set the **Payload URL** to:  
     ```
     http://<jenkins-public-ip>:8080/github-webhook/
     ```
   - Selected **Content type** as `application/json`.
   - Chose to trigger on **Just the push event**.
   - Clicked **Add Webhook**.

2. **Jenkins Job Configuration**  
   - In the Jenkins dashboard, I opened my pipeline job.
   - Under **Build Triggers**, I checked:
     - ✅ *GitHub hook trigger for GITScm polling*
   - Saved the configuration.

3. **Firewall/Security Group Configuration**  
   - Ensured my AWS EC2 instance allowed **port 8080** to be publicly accessible.
   - Updated security group rules accordingly.

4. **Tested the Integration**  
   - Pushed a new commit to the `main` branch on GitHub.
   - Observed Jenkins automatically triggering the pipeline without manual input.

### Troubleshooting Faced:

- **Webhook Not Triggering Jenkins:**
  - Initially, I missed opening port 8080 in the AWS Security Group.
  - Fixed by editing the security group and allowing inbound traffic on port 8080.

- **Jenkins Didn't Receive the Payload:**
  - My Jenkins URL had `/` at the end but GitHub added it again: `.../github-webhook//`
  - Fixed by using the correct URL: `http://<jenkins-ip>:8080/github-webhook/`

- **Payload URL Error from GitHub:**
  - GitHub showed a **"We couldn't deliver this payload"** error.
  - Ensured Jenkins server was publicly accessible and GitHub could reach it.

> After these configurations and fixes, Jenkins now automatically builds the pipeline whenever I push code to the repository.
