# DevOps End-to-End CI/CD Pipeline

A hands-on DevOps project that demonstrates how to build, containerize,
continuously integrate, deploy, and monitor a Python Flask application
using modern DevOps tools.

## Project Overview

This project takes a Flask application from source code to a
containerized application running on Kubernetes. GitHub Actions
automates the build and image-publishing workflow, while Prometheus and
Grafana provide application and cluster monitoring.

## Architecture

``` text
Developer
   |
   v
GitHub Repository
   |
   v
GitHub Actions CI/CD
   |
   +--> Build Docker image
   |
   +--> Push image to Docker Hub
              |
              v
       Kubernetes (Kind)
              |
              v
        Flask Application
              |
              +--> /health
              +--> /ready
              +--> /metrics
                         |
                         v
                    Prometheus
                         |
                         v
                      Grafana
```

## Technology Stack

  Area                       Tools
  -------------------------- -------------------
  Application                Python, Flask
  Containerization           Docker
  CI/CD                      GitHub Actions
  Container Registry         Docker Hub
  Orchestration              Kubernetes
  Local Kubernetes Cluster   Kind
  Infrastructure as Code     Terraform
  Monitoring                 Prometheus
  Visualization              Grafana
  Metrics Instrumentation    prometheus-client
  Source Control             Git, GitHub

## Application Endpoints

The Flask application exposes the following endpoints:

  Endpoint     Purpose
  ------------ --------------------------------------------
  `/`          Returns application information and status
  `/health`    Health check endpoint
  `/ready`     Readiness check endpoint
  `/metrics`   Exposes Prometheus application metrics

Application metrics include HTTP request counts and request-duration
measurements.

## CI/CD Workflow

The GitHub Actions workflow automates the container image delivery
process:

1.  A code change is pushed to the GitHub repository.
2.  GitHub Actions runs the configured workflow.
3.  The workflow builds the Docker image.
4.  The image is tagged using the commit SHA and the `latest` tag.
5.  The image is pushed to Docker Hub.

Docker Hub image:

`jayeshdevre/demo-app`

## Kubernetes Deployment

The application is deployed to a local Kubernetes cluster created with
Kind.

The Kubernetes deployment runs two application replicas in the
`demo-app` namespace. Kubernetes manages the application pods and
exposes the application through a Service.

Useful commands:

``` bash
# Check cluster
kind get clusters

# Check application resources
kubectl get all -n demo-app

# Check application pods
kubectl get pods -n demo-app

# Check deployment rollout
kubectl rollout status deployment/demo-app -n demo-app
```

## Monitoring with Prometheus and Grafana

The monitoring stack is installed using the `kube-prometheus-stack` Helm
chart in the `monitoring` namespace.

The Flask application exposes metrics at `/metrics`. A Kubernetes
ServiceMonitor in `k8s/servicemonitor.yaml` configures Prometheus to
scrape the application service.

The Grafana dashboard is exported and version-controlled at:

`monitoring/grafana-dashboard.json`

### Grafana Dashboard Panels

The custom dashboard, **DevOps Application Monitoring**, contains:

1.  **Total HTTP Requests** --- displays the total number of recorded
    HTTP requests.
2.  **Application Requests Per Second** --- displays the request rate
    for the application endpoint.
3.  **Average HTTP Response Duration** --- visualizes average request
    processing duration.
4.  **HTTP Requests by Status Code** --- displays request rates grouped
    by HTTP status code.

## Repository Contents

``` text
devops-e2e-pipeline/
├── app/
│   ├── main.py
│   └── requirements.txt
├── k8s/
│   └── servicemonitor.yaml
├── monitoring/
│   └── grafana-dashboard.json
└── .github/
    └── workflows/
        └── [CI/CD workflow]
```

The repository may contain additional Docker, Kubernetes, Terraform, and
workflow configuration files.

## Running Locally

Create and activate a Python virtual environment, install the
application dependencies, and start the Flask application:

``` bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r app/requirements.txt
python app/main.py
```

Use the application endpoints to verify that it is responding:

``` bash
curl http://localhost:8080/
curl http://localhost:8080/health
curl http://localhost:8080/ready
curl http://localhost:8080/metrics
```

The application port and runtime configuration should match the values
defined in the project.

## Monitoring Setup

The Prometheus and Grafana stack can be installed in a Kind cluster with
Helm:

``` bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

Apply the ServiceMonitor after the application Service and monitoring
stack are available:

``` bash
kubectl apply -f k8s/servicemonitor.yaml
```

To access Grafana locally:

``` bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

Then open `http://localhost:3000` in your browser. Configure the
Prometheus data source if it is not already configured, and import
`monitoring/grafana-dashboard.json` to restore the custom dashboard.

## Infrastructure as Code

Terraform configuration is included as part of the project to
demonstrate infrastructure-as-code workflows. The Kubernetes application
and monitoring resources used in the running demo are also managed
through Kubernetes manifests and Helm commands.

## What This Project Demonstrates

-   Building and running a Python web application.
-   Creating and publishing Docker images.
-   Automating image builds and publishing with GitHub Actions.
-   Deploying and managing application replicas on Kubernetes.
-   Using Terraform for infrastructure-as-code practice.
-   Instrumenting an application with Prometheus metrics.
-   Configuring Prometheus scraping with a Kubernetes ServiceMonitor.
-   Visualizing application metrics in Grafana.
-   Version-controlling a Grafana dashboard as JSON.

## Future Improvements

-   Automate the monitoring stack and data-source provisioning.
-   Add alerting rules and Alertmanager notifications.
-   Add automated application tests to the CI/CD workflow.
-   Deploy to a managed cloud Kubernetes environment.

## Author

**Jayesh Devre**

GitHub: [jayesh-devre](https://github.com/jayesh-devre)
