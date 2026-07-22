# Java Maven App Jenkins Setup Guide

This repository contains a sample Java Maven application that can be built with Maven, containerized with Docker, and automated with Jenkins.

This README organizes the shell commands used during setup into a practical workflow so you can repeat the process without depending on raw terminal history.

## What This Project Uses

- Java 17 Maven application
- Jenkins running in Docker
- Docker for image builds
- GitHub remote for your fork

## 1. Prepare the Machine

Update package metadata:

```bash
sudo apt update
```

Install Docker:

```bash
sudo apt install -y docker.io
```

Check that Docker is available:

```bash
docker
docker ps
docker ps -a
docker images
docker volume ls
```

If Docker has issues, inspect the service:

```bash
systemctl status docker
journalctl -u docker --no-pager -n 50
sudo systemctl restart docker
```

## 2. Start Jenkins in Docker

Run Jenkins with a persistent volume:

```bash
docker run -p 8080:8080 -p 50000:50000 -d \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts
```

If Jenkins must also build Docker images from inside the container, mount the Docker socket:

```bash
docker run -p 8080:8080 -p 50000:50000 -d \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  jenkins/jenkins:lts
```

Useful container commands:

```bash
docker ps
docker start <container_id>
docker stop <container_id>
docker exec -it <container_id> bash
docker exec -u 0 -it <container_id> bash
```

Notes:

- Use `jenkins/jenkins:lts`, not `jenkin/jenkins:lts`.
- Use `jenkins_home`, not `jenkin_home`.
- The root shell inside the container requires `docker exec -u 0 -it <container_id> bash`.

## 3. Get the Jenkins Initial Admin Password

Inspect the Jenkins volume:

```bash
ls /var/lib/docker/volumes/jenkins_home/
ls /var/lib/docker/volumes/jenkins_home/_data/secrets/initialAdminPassword
cat /var/lib/docker/volumes/jenkins_home/_data/secrets/initialAdminPassword
```

You can also inspect the volume directly:

```bash
docker volume inspect jenkins_home
```

Open Jenkins in the browser at `http://<server-ip>:8080` and paste the password when prompted.

## 4. Create a Workspace and Clone the Project

Create a local projects directory:

```bash
mkdir -p ~/projects
cd ~/projects
ls -la ~
ls
```

Clone the starter code branch:

```bash
git clone -b starting-code https://gitlab.com/twn-devops-bootcamp/latest/08-jenkins/java-maven-app.git
cd java-maven-app
```

## 5. Repoint the Git Remotes

Rename the original remote and add your GitHub repository:

```bash
git remote rename origin upstream
git remote add origin git@github.com:Godswill012/jenkins-java-app.git
git remote -v
```

Create and push your working branch:

```bash
git checkout -b freestyle-demo
git push -u origin freestyle-demo
```

## 6. Configure GitHub SSH Access

Generate an SSH key:

```bash
ssh-keygen -t ed25519 -C "josephnsudegodswill@gmail.com"
cat ~/.ssh/id_ed25519.pub
```

Add the public key to GitHub, then test access:

```bash
ssh -T git@github.com
git push -u origin freestyle-demo
```

## 7. Sync Additional Branches from Upstream

Fetch remote branches:

```bash
git fetch upstream
git branch -r
```

Create local branches from upstream and push them to your fork:

```bash
git checkout -b starting-code upstream/starting-code
git push -u origin starting-code

git checkout -b jenkins-jobs upstream/jenkins-jobs
git push -u origin jenkins-jobs

git checkout -b jenkins-shared-lib upstream/jenkins-shared-lib
git push -u origin jenkins-shared-lib

git checkout -b master upstream/master
git push -u origin master

git checkout -b feature/payment upstream/feature/payment
git push -u origin feature/payment
```

Return to your main working branch when needed:

```bash
git checkout freestyle-demo
git branch -a
git status
```

## 8. Merge Jenkins Work into Your Branch

Example merge flow:

```bash
git checkout freestyle-demo
git merge jenkins-jobs
git status
```

If Git identity is missing, configure it:

```bash
git config --global user.email "josephnsudegodswill@gmail.com"
```

If a merge must be canceled:

```bash
git merge --abort
git status
```

## 9. Install Tools Inside the Jenkins Container

Open a root shell inside Jenkins:

```bash
docker exec -u 0 -it <container_id> bash
```

Install Node.js and npm:

```bash
apt-get update
apt-get install -y nodejs npm
node -v
npm -v
```

Exit the container when done:

```bash
exit
```

## 10. Docker Socket Access for Jenkins

If Jenkins needs to run Docker commands, verify the socket:

```bash
ls -l /var/run/docker.sock
```

You used this quick fix during setup:

```bash
chmod 666 /var/run/docker.sock
ls -l /var/run/docker.sock
```

This works for a local lab environment, but it is too permissive for production. A safer long-term option is to manage Docker group permissions explicitly.

## 11. Local Development Shortcuts

Useful navigation commands from your history:

```bash
cd ~/projects
cd ~/projects/java-maven-app
ls
history
code .
```

## 12. Current Docker Images Seen During Setup

Example images present on the machine:

```text
142.93.13.165:8083/java-maven-app:1.1
godswill012/demo-app:jma-1.0
godswill012/demo-app:jma-1.1
java-maven-app:1.0
jenkins/jenkins:lts
redis:latest
```

## Common Command Corrections

- `apt update` and `apt-get update` both appeared in your history. Either works, but stay consistent within one guide.
- `docker exec -u 0 it 4c96c1c08196` is missing `-it`. The correct form is `docker exec -u 0 -it 4c96c1c08196 bash`.
- `docker start` without a container ID is incomplete.
- `docker volume inspect jenkin_home` should be `docker volume inspect jenkins_home`.
- `curl https://get.docker.com/ > dockerinstall && chmod 777 dockerinstall && ./dockerinstall` is not needed if `docker.io` is already installed from `apt`.

## Suggested Workflow Summary

1. Install and verify Docker.
2. Start Jenkins with a persistent volume.
3. Retrieve the initial admin password and complete Jenkins setup.
4. Clone the project and connect your GitHub fork.
5. Push required branches to your fork.
6. Merge the Jenkins job branch into your working branch.
7. Install any extra tooling Jenkins needs inside the container.
8. Build and publish Docker images from Jenkins.

# Auto-matic version generator (WINDOWS)
mvn build-helper:parse-version versions:set -DnewVersion=${parsedVersion.majorVersion}.${parsedVersion.nextMinorVersion}.${parsedVersion.incrementalVersion} versions:commit