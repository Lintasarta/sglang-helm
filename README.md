# Helm Chart for LLM deployment using Sglang on DekaGPU

This is a reusable Helm chart designed to deploy AI models (such as Qwen2.5-72B-Instruct) on DekaGPU using Sglang. It supports customization via `values.yaml` for flexibility across different models and environments.

---

## Features

- Deploy AI models like Qwen, LLaMA, etc.
- Configurable `replicaCount`, `affinity`, `tolerations`, `resources`, `args`, and `model path`
- Optional PVC and Service creation (can reuse existing resources)
- Customizable Service annotations and ports

---

## Prerequisites

- Helm 3.x
- DekaGPU's Kubernetes cluster with GPU-enabled nodes

---

## Folder Structure
```
sglang-helm/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── pvc.yaml
    └── service.yaml
```
---

## Installation

### 1️⃣ Clone the repository
```bash
git clone https://github.com/Lintasarta/sglang-helm.git
cd sglang-helm
```
### 2️⃣ Customize your values.yaml

Set your desired configurations in values.yaml:
```yaml
app: qwen25-72b-instruct

modelName: Qwen/Qwen2.5-72B-Instruct

replicaCount: 1

extraArgs:
  - "--tp"
  - "8"
  - "--dp"
  - "1"
  - "--schedule-conservativeness"
  - "0.5"
  - "--tool-call-parser"
  - "qwen25"

resources:
  limits:
    cpu: "32"
    memory: "64Gi"
    nvidia.com/gpu: "8"
  requests:
    cpu: "4"
    memory: "16Gi"
    nvidia.com/gpu: "8"

pvc:
  enabled: true
  name: ""  # Leave empty to default to app name
  storageClassName: storage-nvme-c1
  size: 250Gi

service:
  enabled: true
  name: ""
  type: LoadBalancer
  annotations:
    metallb.universe.tf/address-pool: address-pool
  ports:
    - name: http-sglang
      port: 80
      targetPort: 30000
      protocol: TCP
```
💡 Tip: If you want to reuse existing PVC/Service, set pvc.enabled: false and service.enabled: false.



### 3️⃣ Install the chart
```bash
helm install <release-name> . -f values.yaml -n <namespace> --create-namespace
```
Example:
```bash
helm install qwen . -f values.yaml -n sglang --create-namespace
```


### Updating values

To upgrade with new configurations:
```bash
helm upgrade <release-name> . -f values.yaml -n <namespace>
```

### Uninstalling the chart
```bash
helm uninstall <release-name> -n <namespace>
```

### Notes
- The chart automatically mounts PVC and configures GPUs for AI workloads.
- Supports MetalLB or cloud-based LoadBalancer.
- Fully parameterized for multi-model deployments.



### TODO / Suggestions
- Add configurable runtimeClassName
- Support Horizontal Pod Autoscaler (HPA)
- Add ConfigMap / Secrets injection (optional)