# Borneo

Deployment platform for Kubernetes applications using reusable Helm charts.

## 📁 Structure

```
borneo/
├── containers/          # Base Docker images
│   ├── node/           # Node.js with OpenTelemetry
│   └── psql/           # Custom PostgreSQL
├── helm/
│   ├── app/            # Reusable base chart
│   └── helm-charts/    # App-specific charts
│       ├── go-socket-client/
│       ├── silicon-valley/
│       └── singapur/
└── docs/
```

## 🚀 Quick Start

### Deploy existing application:

```bash
cd helm/helm-charts/go-socket-client
helm dependency update
helm upgrade --install go-socket-client . -n go-socket-client -f dev.yaml --create-namespace
```

### Create new application:

1. **Create Chart.yaml:**
```yaml
apiVersion: v2
name: my-app
version: 0.1.0
dependencies:
  - name: app
    version: 0.0.1
    repository: "file://../../app"
```

2. **Create dev.yaml:**
```yaml
appName: my-app
image:
  repository: my-registry/my-app
  tag: latest
service:
  port: 80
environments:
  enabled: true
  data:
    APP_PORT: "3000"
```

3. **Deploy:**
```bash
helm upgrade --install my-app . -f dev.yaml --create-namespace
```
## 📝 Basic Configuration

### Base chart (values.yaml):
```yaml
appName: "my-app"
image:
  repository: "my-registry/my-app"
  tag: "latest"

service:
  port: 80

resources:
  limits:
    cpu: "500m"
    memory: "512Mi"
  requests:
    cpu: "250m"
    memory: "256Mi"

autoscaling:
  enabled: true
  minReplicas: 1
  maxReplicas: 5
```

## 🔧 Useful Commands

```bash
# Validate chart
helm lint .

# View generated templates
helm template my-app . -f dev.yaml

# Update application
helm upgrade my-app . -f dev.yaml

# Uninstall
helm uninstall my-app -n my-namespace
```

## 🐳 Node.js Container

The base image includes preconfigured OpenTelemetry. Use like this:

```dockerfile
FROM borneo/node:latest
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "index.js"]
```

## ⚠️ Troubleshooting

```bash
# View pods
kubectl get pods -n my-namespace

# View logs
kubectl logs deployment/my-app -n my-namespace

# Describe issues
kubectl describe pod my-app-xxx -n my-namespace
```