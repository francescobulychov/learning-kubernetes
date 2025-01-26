# Learning Kubernetes

This repository is dedicated to learning and experimenting with Kubernetes. It includes resources, tutorials, commands, configurations, and a small project designed to provide hands-on experience.

## Overloaded App

The Overloaded App is a simple Flask application designed to help experiment with the capabilities of Kubernetes. The app features a load generator that intentionally overloads the Flask application, allowing you to observe how Kubernetes dynamically scales the number of Pods up or down based on resource usage, thanks to the HorizontalPodAutoscaler.

Both the Flask app and the load generator are public Docker images hosted on my Docker Hub, making it easy to pull and deploy them.

Additionally, a namespace has been configured with Grafana and Prometheus (`admin:prom-operator`) to monitor the CPU usage of each Pod associated with the Flask App.

To make the app and Grafana accessible externally, an Ingress Controller has been configured, exposing both the Flask app (`flask-app.testk8s.com`) and Grafana (`monitoring.admink8s.com`) to the outside world.

### Deployment commands

```
minikube start

k apply -f Namespace.flask-app.yaml

kubens overloaded-app

k apply -f Deployment.flask-app.yaml

k apply -f HorizontalAutoScaler.flask-app.yaml

k apply -f Service.flask-app.yaml

k apply -f ConfigMap.load-generator.yaml

k apply -f Deployment.load-generator.yaml

k apply -f metrics-server0.7.2.yaml

helm upgrade --install monitoring kube-prometheus-stack --repo https://prometheus-community.github.io/helm-charts --namespace monitoring --create-namespace

helm upgrade --install ingress-nginx ingress-nginx --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx --create-namespace

k apply -f Ingress.flask-app.yaml

k apply -f Ingress.monitoring.yaml

minikube tunnel &
```