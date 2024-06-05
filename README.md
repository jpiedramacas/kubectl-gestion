# Gestión de Recursos en Kubernetes

Este documento detalla los conceptos y pasos para la gestión de recursos en Kubernetes, desde la creación de pods hasta la implementación y escalado de aplicaciones. Se explicarán los comandos y códigos utilizados para la correcta configuración y manejo de los recursos dentro de un clúster de Kubernetes.

## Tabla de Contenidos
1. [Introducción](#introducción)
2. [Creación y Gestión de Pods](#creación-y-gestión-de-pods)
   - [Definición de Pods con YAML](#yaml-para-definir-pods)
   - [Estrategias de Despliegue de Pods](#estrategias-de-despliegue-de-pods)
3. [Deployments y Scaling](#deployments-y-scaling)
   - [Estrategias de Despliegue](#estrategias-de-despliegue)
   - [Actualización de un Deployment](#actualización-de-un-deployment)

## Introducción
La gestión de recursos en Kubernetes es crucial para el despliegue, mantenimiento y escalado eficiente de aplicaciones en un clúster. En esta sección se abordan la creación y gestión de diferentes tipos de recursos, desde pods hasta servicios y almacenamiento persistente.

## Creación y Gestión de Pods
### YAML para definir Pods
Un pod es la unidad básica de despliegue en Kubernetes. Para definir un pod, se utiliza un archivo YAML que describe sus características, incluyendo los contenedores que contiene, las imágenes de esos contenedores y las variables de entorno.

**Ejemplo de YAML para un pod:**
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

**Comando para crear un pod a partir de un archivo YAML:**
```sh
kubectl apply -f pod-definition.yaml
```
Este comando crea el pod definido en el archivo `pod-definition.yaml`.

**Verificación del estado del pod:**
```sh
kubectl get pods
```
Este comando muestra el estado de todos los pods en el clúster, permitiendo verificar que el pod se haya creado correctamente.

### Estrategias de Despliegue de Pods
Existen varias estrategias para desplegar pods en Kubernetes:

- **Despliegue manual:** Creación de pods de manera manual usando archivos YAML.
- **Despliegue automatizado:** Uso de controladores como Deployments, ReplicaSets y DaemonSets para gestionar pods de manera automática y escalable.

**Ejemplo de despliegue manual:**
```sh
kubectl run nginx --image=nginx --port=80
```
Este comando crea un pod con el contenedor de nginx expuesto en el puerto 80.

**Uso de Deployments para automatizar el despliegue:**
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

**Aplicación del Deployment:**
```sh
kubectl apply -f deployment.yaml
```
Este comando crea el Deployment definido en el archivo `deployment.yaml`, gestionando el número de réplicas especificado y facilitando actualizaciones.

## Deployments y Scaling
Los Deployments permiten gestionar el ciclo de vida de las aplicaciones, incluyendo el escalado y las actualizaciones.

### Estrategias de Despliegue
- **Recreate:** Elimina todos los pods existentes antes de crear nuevos pods con la versión actualizada.
- **RollingUpdate:** Actualiza los pods gradualmente para minimizar el tiempo de inactividad.

**Ejemplo de RollingUpdate en un Deployment:**
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

### Actualización de un Deployment
Para actualizar un Deployment a una nueva versión de la imagen de contenedor, se utiliza el siguiente comando:
```sh
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```
Este comando actualiza la imagen del contenedor nginx en el Deployment `nginx-deployment` a la versión 1.16.1, aplicando la estrategia de actualización definida (por defecto es RollingUpdate).

---

Este documento proporciona una guía esencial para gestionar recursos en Kubernetes, facilitando la creación, despliegue y escalado de aplicaciones en un entorno de clúster. Cada paso incluye ejemplos prácticos y comandos específicos para garantizar una implementación exitosa y eficiente.

---

 
