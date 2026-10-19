# Exercise 7: Introduction to Continuous Integration (CI) and Jenkins Installation

## Objective
Understand Continuous Integration (CI) and install Jenkins using Docker, unlock it, install plugins, and reach the Jenkins dashboard.

## What is Continuous Integration?
CI is the practice of frequently merging code changes into a shared repository, where every change is automatically built and tested.
- **Workflow:** Code change → push to Git → CI server triggers a build → automated tests run → developers get feedback.
- **Benefits:** early bug detection, faster feedback, always-releasable code, less integration pain.
- **CI tools:** Jenkins, GitHub Actions, GitLab CI/CD, CircleCI, Travis CI, Azure Pipelines, Bamboo, TeamCity.

## What is Jenkins?
Jenkins is an open-source automation server used to build, test and deploy software (CI/CD).

| Concept | Meaning |
|---------|---------|
| Job / Project | A task Jenkins runs |
| Build | One execution of a job |
| Pipeline | A series of stages (build, test, deploy) defined as code |
| Plugins | Extensions that add features (Git, Pipeline, etc.) |
| Nodes | Machines where jobs run (controller / agents) |

## Steps and Commands

### 1. Run Jenkins in Docker
```bash
docker run -d --name jenkins -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts
docker ps -a
```
- `-p 8080:8080`: Jenkins web UI
- `-p 50000:50000`: agent communication port
- (Exercises 6, 8 and 9 reuse a Jenkins started with `-v jenkins_home:/var/jenkins_home` so jobs survive restarts)

### 2. Get the initial admin password
```bash
docker exec -it <container-id> bash
cat /var/jenkins_home/secrets/initialAdminPassword
```

### 3. Unlock Jenkins
Open `http://localhost:8080`, paste the password, then continue through setup: create the first admin user and keep the instance URL as `http://localhost:8080/`.

### 4. Install plugins
The "Install suggested plugins" step failed on this network (plain-http mirror downloads are blocked), so the same suggested plugins were installed with the Jenkins Plugin CLI from the https archive server:
```bash
docker exec -e JENKINS_UC_DOWNLOAD_URL=https://archives.jenkins.io/plugins jenkins \
  jenkins-plugin-cli --plugin-download-directory /var/jenkins_home/plugins \
  --plugins cloudbees-folder antisamy-markup-formatter build-timeout credentials-binding \
  timestamper ws-cleanup ant gradle workflow-aggregator github-branch-source \
  pipeline-github-lib pipeline-graph-view git ssh-slaves matrix-auth ldap email-ext mailer dark-theme

docker restart jenkins
```

## Output Screenshots

### Jenkins container running
![Jenkins Container](screenshots/1.png)

### Initial admin password
![Initial Password](screenshots/2.png)

### Unlock Jenkins page
![Unlock Jenkins](screenshots/3.png)

### Jenkins dashboard
![Jenkins Dashboard](screenshots/4.png)

### Installed plugins
![Installed Plugins](screenshots/5.png)
