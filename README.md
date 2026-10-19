# DevOps Lab Exercises

**Name:** Mridula
**Branch:** Information Science and Engineering
**College:** BMS College of Engineering, Bengaluru

This repository contains my solutions for all 10 DevOps lab exercises, with code, commands and output screenshots.

| No. | Exercise | Folder |
|-----|----------|--------|
| 1 | Kubernetes Getting Started (Hello Pod) | [01-Kubernetes-Getting-Started](01-Kubernetes-Getting-Started) |
| 2 | Deploy Flask App on Minikube using kubectl and YAML | [02-Minikube-Kubectl-Flask](02-Minikube-Kubectl-Flask) |
| 3 | Scaling Flask App with ReplicaSets | [03-Minikube-Scaling-ReplicaSets](03-Minikube-Scaling-ReplicaSets) |
| 4 | Docker Networking with Multiple Containers | [04-Docker-Networking](04-Docker-Networking) |
| 5 | Docker Security with AppArmor | [05-Docker-Security-AppArmor](05-Docker-Security-AppArmor) |
| 6 | Real-Time Monitoring with Prometheus and Grafana | [06-Grafana-Prometheus-Monitoring](06-Grafana-Prometheus-Monitoring) |
| 7 | Jenkins CI Automation | [07-Jenkins-CI-Automation](07-Jenkins-CI-Automation) |
| 8 | Jenkins Hello World Job | [08-Jenkins-Hello-World-Job](08-Jenkins-Hello-World-Job) |
| 9 | Jenkins Multi-Stage Pipeline | [09-Jenkins-Multi-Stage-Pipeline](09-Jenkins-Multi-Stage-Pipeline) |
| 10 | Multi-Node Minikube with Multiple Apps | [10-Minikube-Multi-Node-Multi-App](10-Minikube-Multi-Node-Multi-App) |

**Second repository (used by Jenkins in Exercises 8 and 9, as the manual asks):** https://github.com/mridulaachar3/devops-sample-code
(Copies of its files are also kept in the Exercise 8 and 9 folders.)

## Environment and Notes for the Reviewer
All exercises were done on **Windows 11 + WSL2 (Ubuntu) + Docker Desktop**, following the lab manual step by step. Each exercise README lists its exact commands, output screenshots, and any change from the manual with the reason. Summary:

| Ex | Change from the manual | Reason |
|----|------------------------|--------|
| 2, 3, 10 | `minikube start` uses `--container-runtime=docker` | Needed for `minikube docker-env` and `minikube ssh -- docker images` |
| 3 | Dockerfile copies `app.py`; names use `flashsale` | The manual's Dockerfile/YAML/commands use mismatched names |
| 4 | MySQL root password, Werkzeug pinned, `ping` installed inside the container | MySQL exits without a password; Flask 2.0.1 crashes with newer Werkzeug; `python:3.9-slim` has no `ping` |
| 5 | AppArmor profile could not be loaded | The WSL2 kernel has AppArmor disabled; all steps were run and the errors are documented |
| 6 | `host.docker.internal` instead of `172.17.0.1`; `-p 9090:9090` instead of `--network=host` | The manual's own note says to use `host.docker.internal` on Windows; host networking doesn't publish ports on Docker Desktop |
| 6 | Jenkinsfile: `localSourcePath` set to the Git checkout, `Dockerfile` also copied, Prometheus configs added with `docker cp` | The manual's path is a placeholder; `-v $WORKSPACE/...` mounts don't work when Jenkins itself runs in Docker |
| 7 | Suggested plugins installed with `jenkins-plugin-cli` | The setup wizard's plugin download was blocked on this network |
| 9 | `python3-venv` instead of `python3.11-venv` | The Jenkins image ships Python 3.13 |
| 10 | Namespace `default` instead of `devops-exercise` | The manual never creates the `devops-exercise` namespace |

All exercises completed successfully except the AppArmor enforcement in Exercise 5 (an environment limitation).

## Tools Used
Docker, Kubernetes (Minikube), kubectl, Jenkins, Prometheus, Grafana, Git, GitHub
