# Gestión de Recursos en Kubernetes

Este documento detalla los conceptos y pasos para la gestión de recursos en Kubernetes, desde la creación de Pods hasta la implementación y escalado de aplicaciones. Se explicarán los comandos y códigos utilizados para la correcta configuración y manejo de los recursos dentro de un clúster de Kubernetes.

## Tabla de Contenidos
1. [Introducción](#introducción)
2. [Creación y Gestión de Pods](#creación-y-gestión-de-pods)
3. [Deployments y Scaling](#deployments-y-scaling)
4. [Servicios y Networking](#servicios-y-networking)

## Introducción

La gestión de recursos en Kubernetes es crucial para el despliegue, mantenimiento y escalado eficiente de aplicaciones en un clúster. En esta sección se abordan la creación y gestión de diferentes tipos de recursos, desde Pods hasta servicios y almacenamiento persistente.

---

## 1. Creación y Gestión de Pods

**Descripción:**
Los Pods son la unidad básica de despliegue en Kubernetes. Un Pod puede contener uno o más contenedores que comparten la misma red y almacenamiento.

### 1.1. YAML para definir Pods

**Archivo YAML para definir un Pod:**
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

**Comandos:**
1. **Crear un Pod a partir de un archivo YAML:**
   ```sh
   kubectl apply -f pod-definition.yaml
   ```
2. **Verificación del estado del Pod:**
   ```sh
   kubectl get pods
   ```

### 1.2. Estrategias de Despliegue de Pods

**Estrategias:**
- **Despliegue manual:** Creación de Pods de manera manual usando archivos YAML.
  ```sh
  kubectl run nginx --image=nginx --port=80
  ```
- **Despliegue automatizado:** Uso de controladores como Deployments, ReplicaSets y DaemonSets para gestionar Pods de manera automática y escalable.

**Ejemplo de archivo YAML para un Deployment:**
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

**Comando para aplicar el Deployment:**
```sh
kubectl apply -f deployment.yaml
```

---

## 2. Deployments y Scaling

**Descripción:**
Los Deployments permiten gestionar el ciclo de vida de las aplicaciones, incluyendo el escalado y las actualizaciones.

### 2.1. Estrategias de Despliegue

**Estrategias de Despliegue:**
- **Recreate:** Elimina todos los Pods existentes antes de crear nuevos Pods con la versión actualizada.
- **RollingUpdate:** Actualiza los Pods gradualmente para minimizar el tiempo de inactividad.

**Ejemplo de RollingUpdate en un Deployment:**
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

**Comando para actualizar un Deployment:**
```sh
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```

---

## 3. Servicios y Networking

**Descripción:**
Los Servicios en Kubernetes exponen una aplicación corriendo en un conjunto de Pods y facilitan el descubrimiento y el balanceo de carga.

**Tipos de Servicios:**
- **ClusterIP:** Exposición interna dentro del clúster.
- **NodePort:** Exposición externa en un puerto específico de cada nodo.
- **LoadBalancer:** Exposición externa utilizando un balanceador de carga provisto por el proveedor de la nube.

**Ejemplo de archivo YAML para un Servicio de tipo NodePort:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30007
  type: NodePort
```

**Comando para aplicar el Servicio:**
```sh
kubectl apply -f service.yaml
```

---

Este documento cubre los fundamentos para gestionar recursos en Kubernetes, desde la creación y gestión de Pods, despliegues y escalado, hasta la configuración de servicios y networking. Estos ejemplos y comandos proporcionan una guía práctica para implementar y mantener aplicaciones en un clúster de Kubernetes.
