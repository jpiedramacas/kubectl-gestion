# Gestión de Recursos en Kubernetes

La gestión de recursos en Kubernetes es esencial para el despliegue, mantenimiento y escalado eficiente de aplicaciones en un clúster. Este documento proporciona una guía paso a paso para la creación y gestión de diferentes tipos de recursos, desde pods hasta servicios y escalado automático.

---

## 1. Creación y Gestión de Pods

Los pods son la unidad básica de despliegue en Kubernetes, y pueden contener uno o más contenedores que comparten la misma red y almacenamiento.

### 1.1. Definición de Pods mediante YAML

Para definir un pod en Kubernetes, se utiliza un archivo YAML. Este archivo describe las características del pod, incluyendo los contenedores que contiene, las imágenes de esos contenedores y las variables de entorno.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

Comando para crear un pod a partir de un archivo YAML:

```bash
kubectl apply -f pod-definition.yaml
```

Verificación del estado del pod:

```bash
kubectl get pods
```

### 1.2. Estrategias de Despliegue de Pods

Existen varias estrategias para desplegar pods en Kubernetes, desde despliegue manual hasta despliegue automatizado utilizando controladores como Deployments, ReplicaSets y DaemonSets.

Ejemplo de despliegue manual:

```bash
kubectl run nginx --image=nginx --port=80
```

Uso de Deployments para automatizar el despliegue:

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
        image: nginx:latest
        ports:
        - containerPort: 80
```

Aplicación del Deployment:

```bash
kubectl apply -f deployment.yaml
```

---

## 2. Deployments y Scaling

Los Deployments permiten gestionar el ciclo de vida de las aplicaciones, incluyendo el escalado y las actualizaciones.

### 2.1. Estrategias de Despliegue

- **Recreate**: Elimina todos los pods existentes antes de crear nuevos pods con la versión actualizada.
- **RollingUpdate**: Actualiza los pods gradualmente para minimizar el tiempo de inactividad.

Ejemplo de estrategia RollingUpdate en un Deployment:

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

Actualizar un Deployment:

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```

Ejemplo de definición de un Deployment en YAML:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: nginx:latest
        ports:
        - containerPort: 80
```

### 2.2. Autoescalado de Aplicaciones (Horizontal Pod Autoscaler)

El autoescalado horizontal (HPA) permite escalar automáticamente el número de pods en función de la carga.

Crear un HPA:

```bash
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=1 --max=10
```

Verificar el estado del HPA:

```bash
kubectl get hpa
```

Ejemplo de definición de un HPA en YAML:

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: webapp
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

Comando para aplicar el template:

```bash
kubectl apply -f hpa.yaml
```

---

## 3. Servicios y Networking

Los servicios en Kubernetes permiten exponer aplicaciones en el clúster y gestionar el tráfico de red.

### 3.1. Tipos de Servicios (ClusterIP, NodePort, LoadBalancer)

- **ClusterIP**: Exponiendo el servicio dentro del clúster.
- **NodePort**: Exponiendo el servicio en cada nodo del clúster en un puerto estático.
- **LoadBalancer**: Exponiendo el servicio utilizando un balanceador de carga externo.

Ejemplo de servicio ClusterIP:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
  type: ClusterIP
```

Crear el servicio:

```bash
kubectl apply -f service.yaml
```

--- 

Con esta guía, podrás gestionar de manera eficiente los recursos en tu clúster de Kubernetes, desde la creación y despliegue de pods hasta la configuración de servicios y el escalado automático de aplicaciones.
