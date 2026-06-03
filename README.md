# Laboratorio 3 - Mi despliegue CI/CD en Kubernetes

**Alumno:** Roberto Rivera

## Estructura del proyecto

```
laboratorio3/
├── proyecto/                         # Aplicación NestJS
│   ├── Dockerfile                    # Dockerfile multi-stage
│   ├── .dockerignore
│   ├── package.json
│   ├── pnpm-lock.yaml
│   ├── src/                          # Código fuente
│   └── test/                         # Tests unitarios y e2e
├── entrega.yaml                      # Manifiestos Kubernetes
├── agent.yaml                        # Pod template para agente Jenkins
├── Jenkinsfile.Roberto-Rivera        # Pipeline Jenkins CI/CD
├── README.md
└── evidencias/                       # Salidas de comandos
```

## Naming

| Recurso         | Nombre                |
|-----------------|-----------------------|
| Namespace       | ns-roberto-rivera     |
| Deployment      | app-roberto-rivera    |
| Service         | svc-roberto-rivera    |
| ConfigMap       | config-roberto-rivera |
| Secret          | secret-roberto-rivera |
| Imagen Docker   | rarcardenas/tarea-final:roberto |
| APP_VERSION     | 3.0.0                 |
| Jenkinsfile     | Jenkinsfile.Roberto-Rivera |

## Prerrequisitos

- Docker
- Kubernetes (k3s, kind, minikube, etc.)
- kubectl
- Acceso a Docker Hub

## Pasos de ejecución manual

### 1. Construir y publicar la imagen Docker

```bash
cd proyecto
docker build -t rarcardenas/tarea-final:roberto .
docker push rarcardenas/tarea-final:roberto
```

### 2. Crear secrets de registro

```bash
kubectl create namespace ns-roberto-rivera

kubectl create secret docker-registry regcred-dh \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=rarcardenas \
  --docker-password=<token> \
  --docker-email=rarcardenas1990@gmail.com \
  -n ns-roberto-rivera

kubectl create secret docker-registry regcred-gh \
  --docker-server=https://ghcr.io \
  --docker-username=rrcardenas \
  --docker-password=<token> \
  --docker-email=rarcardenas1990@gmail.com \
  -n ns-roberto-rivera
```

### 3. Desplegar en Kubernetes

```bash
kubectl apply -f entrega.yaml
```

### 4. Verificar el despliegue

```bash
kubectl get pods -n ns-roberto-rivera
kubectl get deployment -n ns-roberto-rivera
kubectl get svc -n ns-roberto-rivera
```

### 5. Probar la aplicación

```bash
kubectl port-forward svc/svc-roberto-rivera 8080:80 -n ns-roberto-rivera
curl http://localhost:8080/
curl http://localhost:8080/lab
```

## Pipeline Jenkins

El archivo `Jenkinsfile.Roberto-Rivera` automatiza el flujo CI/CD:

| Stage      | Descripción                                    |
|------------|------------------------------------------------|
| install    | Configura pnpm e instala dependencias          |
| test       | Ejecuta tests unitarios (jest)                 |
| build      | Compila la aplicación NestJS                   |
| push       | Construye y publica imagen en DockerHub y GHCR |
| deploy     | Aplica manifiestos y hace rollout en K8s       |

El pipeline usa un agente Kubernetes definido en `agent.yaml` con contenedores para pnpm, buildkit y kubectl.

## Evidencias

En la carpeta `evidencias/` se incluyen las salidas de:

- `kubectl cluster-info`
- `kubectl get nodes`
- `kubectl get pods -n ns-roberto-rivera`
- `kubectl get deployment -n ns-roberto-rivera`
- `kubectl get svc -n ns-roberto-rivera`
- `kubectl logs deployment/app-roberto-rivera -n ns-roberto-rivera`
- `kubectl exec deployment/app-roberto-rivera -n ns-roberto-rivera -- printenv`
- `kubectl get configmap config-roberto-rivera`
- `kubectl get secret secret-roberto-rivera`
- `curl http://localhost:8080/`
- `curl http://localhost:8080/lab`
