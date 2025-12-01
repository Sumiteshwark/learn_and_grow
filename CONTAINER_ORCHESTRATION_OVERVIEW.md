**Last Updated:** November 2025  
**Author:** SUMITESHWAR KUMAR

---

# **Container Orchestration Concepts - Complete Guide**

## Table of Contents
1. [Introduction to Container Orchestration](#introduction-to-container-orchestration)
2. [Container Orchestration Challenges](#container-orchestration-challenges)
3. [Kubernetes Basics](#kubernetes-basics)
4. [Alternative Orchestration Platforms](#alternative-orchestration-platforms)
5. [Service Discovery](#service-discovery)
6. [Load Balancing](#load-balancing)
7. [Scaling and Auto-scaling](#scaling-and-auto-scaling)
8. [Health Checks and Monitoring](#health-checks-and-monitoring)
9. [Configuration Management](#configuration-management)
10. [Security in Container Orchestration](#security-in-container-orchestration)
11. [Networking in Orchestrated Environments](#networking-in-orchestrated-environments)
12. [Storage and Persistence](#storage-and-persistence)

---

## Introduction to Container Orchestration

### What is Container Orchestration?
Container orchestration is the automated management of containerized applications, including deployment, scaling, networking, and availability. It handles the complexity of running containers at scale across multiple hosts.

### Why Container Orchestration?
```
Without Orchestration:
┌─────────────────────────────────────────────────────────────┐
│ Host 1: Container A, Container B                           │
│ Host 2: Container C                                        │
│ Host 3: Container D, Container E                           │
│                                                             │
│ Problems:                                                   │
│ - Manual deployment                                         │
│ - No automatic scaling                                      │
│ - Difficult service discovery                               │
│ - No health monitoring                                      │
│ - Manual failover                                           │
└─────────────────────────────────────────────────────────────┘

With Orchestration:
┌─────────────────────────────────────────────────────────────┐
│ Orchestration Platform                                      │
│ ├─ Automatic deployment                                     │
│ ├─ Service discovery                                        │
│ ├─ Load balancing                                           │
│ ├─ Auto-scaling                                            │
│ ├─ Health monitoring                                        │
│ ├─ Rolling updates                                          │
│ └─ Self-healing                                            │
│                                                             │
│ Host 1: Container A, Container B                           │
│ Host 2: Container C, Container D                           │
│ Host 3: Container E, Container F                           │
└─────────────────────────────────────────────────────────────┘
```

### Key Benefits
- **Automation**: Automated deployment and scaling
- **High Availability**: Automatic failover and recovery
- **Scalability**: Easy horizontal and vertical scaling
- **Resource Optimization**: Efficient resource utilization
- **Service Discovery**: Automatic service registration and discovery
- **Rolling Updates**: Zero-downtime deployments
- **Self-healing**: Automatic recovery from failures

---

## Container Orchestration Challenges

### Complexity Management
```
Challenges:
├─ Service Discovery
│  ├─ How do services find each other?
│  ├─ Dynamic IP addresses
│  └─ Load balancing
├─ Configuration Management
│  ├─ Environment-specific configs
│  ├─ Secrets management
│  └─ Configuration updates
├─ Networking
│  ├─ Inter-service communication
│  ├─ External access
│  └─ Network policies
├─ Storage
│  ├─ Persistent data
│  ├─ Shared storage
│  └─ Backup and recovery
├─ Security
│  ├─ Access control
│  ├─ Network security
│  └─ Container security
└─ Monitoring
   ├─ Health checks
   ├─ Metrics collection
   └─ Logging
```

### Resource Management
```
Resource Allocation:
├─ CPU allocation and limits
├─ Memory allocation and limits
├─ Storage allocation
├─ Network bandwidth
└─ Resource quotas and limits

Scheduling:
├─ Node selection
├─ Resource availability
├─ Affinity and anti-affinity
├─ Taints and tolerations
└─ Priority and preemption
```

---

## Kubernetes Basics

### Kubernetes Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    Control Plane                            │
├─────────────────────────────────────────────────────────────┤
│ API Server │ Scheduler │ Controller Manager │ etcd         │
├─────────────────────────────────────────────────────────────┤
│                    Worker Nodes                             │
├─────────────────────────────────────────────────────────────┤
│ Kubelet │ Kube-proxy │ Container Runtime │ Pods           │
└─────────────────────────────────────────────────────────────┘
```

### Core Concepts

#### Pods
Pods are the smallest deployable units in Kubernetes. They contain one or more containers that share the same network namespace and storage.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

#### ReplicaSets
ReplicaSets ensure that a specified number of pod replicas are running at any given time.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
```

#### Deployments
Deployments provide declarative updates for Pods and ReplicaSets, enabling rolling updates and rollbacks.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

#### Services
Services provide stable network endpoints for Pods, enabling service discovery and load balancing.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

#### ConfigMaps and Secrets
ConfigMaps and Secrets store configuration data and sensitive information.

```yaml
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://localhost:5432/mydb"
  log_level: "info"

# Secret
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: dXNlcm5hbWU=  # base64 encoded
  password: cGFzc3dvcmQ=  # base64 encoded
```

### Kubernetes Commands
```bash
# Get resources
kubectl get pods
kubectl get services
kubectl get deployments
kubectl get nodes

# Describe resources
kubectl describe pod pod-name
kubectl describe service service-name

# Create resources
kubectl apply -f deployment.yaml
kubectl create -f service.yaml

# Delete resources
kubectl delete pod pod-name
kubectl delete -f deployment.yaml

# Execute commands in pods
kubectl exec -it pod-name -- /bin/bash

# View logs
kubectl logs pod-name
kubectl logs -f pod-name

# Port forwarding
kubectl port-forward pod-name 8080:80

# Scale deployments
kubectl scale deployment nginx-deployment --replicas=5
```

---

## Alternative Orchestration Platforms

### Docker Swarm
Docker Swarm is Docker's native clustering and orchestration solution.

```bash
# Initialize swarm
docker swarm init

# Create service
docker service create --name nginx --replicas 3 nginx

# Scale service
docker service scale nginx=5

# Update service
docker service update --image nginx:1.19 nginx

# Remove service
docker service rm nginx
```

### Apache Mesos
Apache Mesos is a distributed systems kernel that provides resource isolation and sharing across distributed applications.

```yaml
# Marathon application definition
{
  "id": "/nginx",
  "cmd": null,
  "cpus": 0.1,
  "mem": 128,
  "disk": 0,
  "instances": 3,
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "nginx:latest",
      "portMappings": [
        {
          "containerPort": 80,
          "hostPort": 0,
          "protocol": "tcp"
        }
      ]
    }
  },
  "healthChecks": [
    {
      "protocol": "HTTP",
      "path": "/",
      "portIndex": 0,
      "gracePeriodSeconds": 30,
      "intervalSeconds": 10,
      "timeoutSeconds": 5,
      "maxConsecutiveFailures": 3
    }
  ]
}
```

### HashiCorp Nomad
Nomad is a flexible workload orchestrator that can manage containers, virtual machines, and standalone applications.

```hcl
job "nginx" {
  datacenters = ["dc1"]
  type = "service"

  group "web" {
    count = 3

    network {
      port "http" {
        static = 8080
      }
    }

    task "nginx" {
      driver = "docker"

      config {
        image = "nginx:latest"
        ports = ["http"]
      }

      resources {
        cpu    = 500
        memory = 512
      }

      service {
        name = "nginx"
        port = "http"

        check {
          type     = "http"
          path     = "/"
          interval = "10s"
          timeout  = "2s"
        }
      }
    }
  }
}
```

---

## Service Discovery

### What is Service Discovery?
Service discovery is the automatic detection of services and their network locations in a distributed system.

### Service Discovery Patterns

#### Client-Side Discovery
```
Client ──▶ Service Registry ──▶ Service Instance
   │              │
   └─▶ Load Balancer
```

#### Server-Side Discovery
```
Client ──▶ Load Balancer ──▶ Service Registry
   │              │
   └─▶ Service Instance
```

### Service Discovery Implementations

#### Consul
```bash
# Register service
consul services register -name=web -port=8080

# Discover services
consul catalog services
consul catalog nodes -service=web

# Health checks
curl http://localhost:8500/v1/health/service/web
```

#### etcd
```bash
# Store service information
etcdctl put /services/web/instance1 '{"host":"192.168.1.10","port":8080}'

# Retrieve service information
etcdctl get /services/web/instance1

# Watch for changes
etcdctl watch /services/web/ --prefix
```

#### Kubernetes Service Discovery
```yaml
# Service definition
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
```

```bash
# DNS resolution
nslookup web-service.default.svc.cluster.local

# Environment variables
echo $WEB_SERVICE_SERVICE_HOST
echo $WEB_SERVICE_SERVICE_PORT
```

---

## Load Balancing

### Load Balancing Strategies

#### Round Robin
```
Request 1 ──▶ Server 1
Request 2 ──▶ Server 2
Request 3 ──▶ Server 3
Request 4 ──▶ Server 1
```

#### Least Connections
```
Server 1: 5 connections
Server 2: 3 connections
Server 3: 7 connections
New Request ──▶ Server 2 (least connections)
```

#### IP Hash
```
Client IP: 192.168.1.100
Hash(192.168.1.100) % 3 = 1
Request ──▶ Server 1
```

#### Weighted Round Robin
```
Server 1: weight 3
Server 2: weight 2
Server 3: weight 1

Request 1,2,3 ──▶ Server 1
Request 4,5 ──▶ Server 2
Request 6 ──▶ Server 3
```

### Load Balancer Types

#### Application Load Balancer (Layer 7)
```yaml
# Kubernetes Ingress
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

#### Network Load Balancer (Layer 4)
```yaml
# Kubernetes Service
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

### Health Checks
```yaml
# Kubernetes health checks
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
  timeoutSeconds: 3
  failureThreshold: 3
```

---

## Scaling and Auto-scaling

### Manual Scaling
```bash
# Kubernetes
kubectl scale deployment nginx-deployment --replicas=5

# Docker Swarm
docker service scale nginx=5

# Nomad
nomad job scale nginx 5
```

### Horizontal Pod Autoscaler (HPA)
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### Vertical Pod Autoscaler (VPA)
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: nginx-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: '*'
      minAllowed:
        cpu: 100m
        memory: 50Mi
      maxAllowed:
        cpu: 1
        memory: 500Mi
```

### Cluster Autoscaler
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
    spec:
      containers:
      - name: cluster-autoscaler
        image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.20.0
        command:
        - ./cluster-autoscaler
        - --v=4
        - --stderrthreshold=info
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/my-cluster
        - --balance-similar-node-groups
        - --skip-nodes-with-system-pods=false
```

---

## Health Checks and Monitoring

### Health Check Types

#### HTTP Health Checks
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
    httpHeaders:
    - name: Custom-Header
      value: health-check
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
  successThreshold: 1
```

#### TCP Health Checks
```yaml
livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
```

#### Command Health Checks
```yaml
livenessProbe:
  exec:
    command:
    - /bin/sh
    - -c
    - "ps aux | grep nginx | grep -v grep"
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
```

### Monitoring Stack

#### Prometheus
```yaml
# Prometheus configuration
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
    - role: pod
    relabel_configs:
    - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
      action: keep
      regex: true
    - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
      action: replace
      target_label: __metrics_path__
      regex: (.+)
    - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
      action: replace
      regex: ([^:]+)(?::\d+)?;(\d+)
      replacement: $1:$2
      target_label: __address__
```

#### Grafana
```yaml
# Grafana dashboard configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-dashboards
data:
  dashboard.json: |
    {
      "dashboard": {
        "title": "Kubernetes Cluster",
        "panels": [
          {
            "title": "CPU Usage",
            "type": "graph",
            "targets": [
              {
                "expr": "rate(container_cpu_usage_seconds_total[5m])",
                "legendFormat": "{{pod}}"
              }
            ]
          }
        ]
      }
    }
```

### Logging
```yaml
# Fluentd configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluent.conf: |
    <source>
      @type tail
      path /var/log/containers/*.log
      pos_file /var/log/fluentd-containers.log.pos
      tag kubernetes.*
      read_from_head true
      <parse>
        @type json
        time_format %Y-%m-%dT%H:%M:%S.%NZ
      </parse>
    </source>

    <match kubernetes.**>
      @type elasticsearch
      host elasticsearch-logging
      port 9200
      logstash_format true
      logstash_prefix k8s
      <buffer>
        @type file
        path /var/log/fluentd-buffers/kubernetes.system.buffer
        flush_mode interval
        retry_type exponential_backoff
        flush_interval 5s
        retry_forever false
        retry_max_interval 30
        chunk_limit_size 2M
        queue_limit_length 8
        overflow_action block
      </buffer>
    </match>
```

---

## Configuration Management

### Environment Variables
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: DATABASE_URL
      value: "postgresql://localhost:5432/mydb"
    - name: LOG_LEVEL
      value: "info"
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: api-key
```

### ConfigMaps
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "postgresql://localhost:5432/mydb"
  log_level: "info"
  app_config.yaml: |
    server:
      port: 8080
      host: 0.0.0.0
    database:
      host: localhost
      port: 5432
      name: mydb
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    envFrom:
    - configMapRef:
        name: app-config
    volumeMounts:
    - name: config-volume
      mountPath: /etc/app
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

### Secrets
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  username: dXNlcm5hbWU=  # base64 encoded
  password: cGFzc3dvcmQ=  # base64 encoded
  api-key: YXBpLWtleQ==   # base64 encoded
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app
    image: myapp:latest
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: password
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
```

---

## Security in Container Orchestration

### Pod Security Policies
```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  hostNetwork: false
  hostIPC: false
  hostPID: false
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  supplementalGroups:
    rule: 'MustRunAs'
    ranges:
      - min: 1
        max: 65535
  fsGroup:
    rule: 'MustRunAs'
    ranges:
      - min: 1
        max: 65535
  readOnlyRootFilesystem: true
```

### Network Policies
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-web-traffic
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
```

### RBAC (Role-Based Access Control)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

### Image Security
```yaml
# Admission controller for image security
apiVersion: admission.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: image-policy-webhook
webhooks:
- name: image-policy.example.com
  rules:
  - apiGroups: [""]
    apiVersions: ["v1"]
    operations: ["CREATE"]
    resources: ["pods"]
    scope: "Namespaced"
  clientConfig:
    service:
      namespace: default
      name: image-policy-webhook
      path: "/validate"
  admissionReviewVersions: ["v1"]
  sideEffects: None
```

---

## Networking in Orchestrated Environments

### Kubernetes Networking

#### Pod Network
```yaml
# CNI configuration (Calico example)
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: default-ipv4-ippool
spec:
  cidr: 192.168.0.0/16
  ipipMode: Always
  natOutgoing: true
```

#### Service Network
```yaml
# Service with ClusterIP
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
```

#### Ingress Network
```yaml
# Ingress controller
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

### Service Mesh (Istio)
```yaml
# Virtual Service
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: web-vs
spec:
  hosts:
  - web.example.com
  gateways:
  - web-gateway
  http:
  - route:
    - destination:
        host: web-service
        port:
          number: 80
      weight: 100
```

```yaml
# Destination Rule
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: web-dr
spec:
  host: web-service
  trafficPolicy:
    loadBalancer:
      simple: ROUND_ROBIN
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 1024
        maxRequestsPerConnection: 10
```

---

## Storage and Persistence

### Persistent Volumes
```yaml
# Persistent Volume
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: fast
  hostPath:
    path: /mnt/data
```

```yaml
# Persistent Volume Claim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: fast
```

```yaml
# Pod using PVC
apiVersion: v1
kind: Pod
metadata:
  name: pv-pod
spec:
  containers:
  - name: app
    image: nginx:latest
    volumeMounts:
    - name: storage-volume
      mountPath: /data
  volumes:
  - name: storage-volume
    persistentVolumeClaim:
      claimName: pvc-demo
```

### Storage Classes
```yaml
# Storage Class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  iops: "3000"
  encrypted: "true"
reclaimPolicy: Delete
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: Immediate
```

### StatefulSets
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "fast"
      resources:
        requests:
          storage: 1Gi
```

---

## Best Practices

### Application Design
1. **Stateless applications**: Design applications to be stateless
2. **Health checks**: Implement proper health check endpoints
3. **Graceful shutdown**: Handle shutdown signals properly
4. **Configuration**: Use environment variables and ConfigMaps
5. **Logging**: Log to stdout/stderr
6. **Security**: Follow security best practices

### Deployment Strategies
1. **Rolling updates**: Use rolling update strategy
2. **Blue-green deployment**: Deploy new version alongside old
3. **Canary deployment**: Gradually roll out to subset of users
4. **A/B testing**: Test different versions with different users

### Monitoring and Observability
1. **Metrics**: Collect application and infrastructure metrics
2. **Logging**: Centralized logging with proper structure
3. **Tracing**: Distributed tracing for microservices
4. **Alerting**: Set up proper alerting rules
5. **Dashboards**: Create meaningful dashboards

### Security
1. **Image scanning**: Scan container images for vulnerabilities
2. **Network policies**: Implement network segmentation
3. **RBAC**: Use role-based access control
4. **Secrets management**: Secure handling of sensitive data
5. **Pod security**: Use pod security policies

---

## Next Steps

After mastering container orchestration concepts, you'll be ready to:

1. **Learn Nomad**: Understand how Nomad differs from Kubernetes
2. **Explore Consul**: See how service discovery works in practice
3. **Integrate Vault**: Understand secure secret management
4. **Build Production Systems**: Deploy scalable, resilient applications

### Practice Projects

1. **Multi-tier Application**: Deploy a web application with database
2. **Microservices**: Build and deploy microservices architecture
3. **CI/CD Pipeline**: Set up automated deployment pipeline
4. **Monitoring Stack**: Deploy monitoring and logging solutions

### Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Docker Swarm Documentation](https://docs.docker.com/engine/swarm/)
- [Nomad Documentation](https://www.nomadproject.io/docs/)
- [Consul Documentation](https://www.consul.io/docs/)
- [Vault Documentation](https://www.vaultproject.io/docs/) 