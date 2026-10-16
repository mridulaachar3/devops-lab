# Exercise 9: Jenkins Multi-Stage Pipeline (Deploying a Python Application)

## Objective
Create and run a Jenkins Pipeline with multiple stages (Build, Test, Deploy, Run, Test Application) for a Python Flask app, with the pipeline defined as code in a `Jenkinsfile` stored in GitHub.

## Files
| File | Purpose |
|------|---------|
| `app.py` | Flask app returning "Hello, Jenkins Multi-Stage Pipeline!" |
| `requirements.txt` | Python dependencies |
| `test_app.py` | Unit test that checks the home page response |
| `Jenkinsfile` | Declarative pipeline with all the stages |

## Pipeline Stages
| Stage | What it does |
|-------|-------------|
| Checkout SCM | Jenkins clones this GitHub repo |
| Build | Checks Python and Flask are available |
| Test | Runs unit tests with `python3 -m unittest discover -s .` |
| Deploy | Copies `app.py` to a mock deployment folder `python-app-deploy` |
| Run Application | Starts the app in the background with `nohup` |
| Test Application | Runs `test_app.py` against the app |
| Post Actions | Prints success or failure message |

## Steps

### 1. Install Python and Flask inside the Jenkins container
(The manual's Step 8 "Handling build errors": the Jenkins image has no Python.)
~~~bash
docker exec -u root jenkins bash -c "sed -i 's|http://|https://|g' /etc/apt/sources.list.d/debian.sources && apt-get update && apt-get install -y python3 python3-flask"
~~~

### 2. Push app, tests and Jenkinsfile to GitHub
~~~bash
git add .
git commit -m "Add Exercise 9 app, tests and Jenkinsfile"
git push
~~~

### 3. Create the Pipeline job
- New Item → `Python-MultiStage-Pipeline` → **Pipeline** → OK
- Definition: **Pipeline script from SCM**
- SCM: **Git**, URL `https://github.com/mridulaachar3/devops-lab.git`, no credentials
- Branch Specifier: `*/main`
- Script Path: `09-Jenkins-Multi-Stage-Pipeline/Jenkinsfile`

### 4. Run the pipeline
- **Build Now** → all stages pass → Console Output shows `Ran 1 test ... OK` and `Finished: SUCCESS`

## Changes from the Lab Manual
- `dir('09-Jenkins-Multi-Stage-Pipeline')` is used in the Jenkinsfile because the files live in this folder of the repo.
- Extra lines (`python3 --version`, `ls`, `cat app.log`) give more proof of each stage in the console output.

## Output Screenshots

### Pipeline job configuration (SCM)
![Pipeline Config](screenshots/1.png)

### Script Path
![Script Path](screenshots/2.png)

### All stages passed
![Pipeline Stages](screenshots/3.png)

### Console output: tests OK and Finished SUCCESS
![Console Output](screenshots/4.png)
