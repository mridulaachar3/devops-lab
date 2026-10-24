# Exercise 8: Creating a "Hello World" Jenkins Job

## Objective
Create a shell script, push it to a new GitHub repository, and create a Jenkins Freestyle job that pulls the script from GitHub and runs it.

## Environment
Windows 11 + WSL2 (Ubuntu) + Docker Desktop; Jenkins runs in a Docker container (`jenkins/jenkins:lts`, UI on port 8080).

## Repository used by Jenkins
**https://github.com/mridulaachar3/devops-sample-code** (created as per Step 1a of the manual)

A copy of `hello-world.sh` is also kept in this folder for reference.

## Steps

### 1a. Create a new GitHub repository
- Name: `devops-sample-code`
- Description: `A demo repository for Jenkins scripting.`
- Visibility: Public

### 1b. Authentication
GitHub authentication is done with the GitHub CLI (`gh auth login`) instead of a personal access token.

### 2. Create the script locally
~~~bash
cat > hello-world.sh << 'SCRIPT'
#!/bin/bash
echo "Hello, Jenkins!"
SCRIPT
chmod +x hello-world.sh
~~~

### 3-6. Initialise Git, commit and push
`git init -b main` is used instead of the manual's plain `git init`, so the local branch is called `main` and the manual's `git push -u origin main` works (plain `git init` creates `master`, and pushing `main` then fails with `src refspec main does not match any`).
~~~bash
git init -b main
git add hello-world.sh
git status
git commit -m "Add hello-world.sh"
git remote add origin https://github.com/mridulaachar3/devops-sample-code.git
git push -u origin main
~~~

### 7. Verify the script on GitHub
`hello-world.sh` is visible in the `devops-sample-code` repository.

## Jenkins Job
- **New Item** → `HelloWorld` → **Freestyle project**
- **Description:** `Hello World! Jenkins job.`
- **Source Code Management:** Git → `https://github.com/mridulaachar3/devops-sample-code.git`, branch `*/main`
- **Build Steps:** Execute shell → `sh hello-world.sh`
- **Save → Build Now → Console Output**

## Output
~~~
Started by user Mridula
Fetching upstream changes from https://github.com/mridulaachar3/devops-sample-code.git
Commit message: "Add hello-world.sh"
+ sh hello-world.sh
Hello, Jenkins!
Finished: SUCCESS
~~~

## Output Screenshots

### New repository devops-sample-code
![New Repo](screenshots/1.png)

### Script created, committed and pushed
![Git Push](screenshots/6.png)

### Script verified on GitHub
![Script on GitHub](screenshots/7.png)

### Source Code Management configuration
![SCM Config](screenshots/2.png)

### Execute shell build step
![Build Step](screenshots/3.png)

### Console output: Hello, Jenkins!
![Console Output](screenshots/4.png)

### Successful build on the job page
![Job Page](screenshots/5.png)

## Changes from the Manual
- `git init -b main` instead of plain `git init`, so the local branch is `main` and the manual's `git push -u origin main` works.
- GitHub login with the GitHub CLI (`gh auth login`) instead of a personal access token.

## Learnings
- A Freestyle project can pull a script from a Git repository (Source Code Management) and run it with an Execute shell build step.
- The Console Output of a build shows the Git fetch, the commands run, and the final status.

## Result
The `HelloWorld` job fetched `hello-world.sh` from `devops-sample-code`, ran it, printed `Hello, Jenkins!` and finished with `Finished: SUCCESS`.
