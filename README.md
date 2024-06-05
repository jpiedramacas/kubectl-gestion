# Gestión de Recursos en Kubernetes

La gestión de recursos en Kubernetes es fundamental para el despliegue, mantenimiento y escalado eficiente de aplicaciones en un clúster. Este apartado aborda la creación y gestión de diferentes tipos de recursos, desde pods hasta servicios y almacenamiento persistente.

## Tabla de Contenidos
1. [Creación y Gestión de Pods](#creación-y-gestión-de-pods)
    - [YAML para definir Pods](#yaml-para-definir-pods)
    - [Estrategias de despliegue de Pods](#estrategias-de-despliegue-de-pods)
2. [Deployments y Scaling](#deployments-y-scaling)
    - [Conceptos clave de los Deployments](#conceptos-clave-de-los-deployments)
    - [Estrategias de despliegue](#estrategias-de-despliegue)
    - [Autoescalado de aplicaciones (Horizontal Pod Autoscaler)](#autoescalado-de-aplicaciones-horizontal-pod-autoscaler)

## Creación y Gestión de Pods
### YAML para definir Pods
Los pods son la unidad básica de despliegue en Kubernetes. Un pod puede contener uno o más contenedores que comparten la misma red y almacenamiento. Para definir un pod en Kubernetes, se utiliza un archivo YAML. Este archivo describe las características del pod, incluyendo los contenedores que contiene, las imágenes de esos contenedores y las variables de entorno.

#### Ejemplo de YAML para un pod:
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

#### Comando para crear un pod a partir de un archivo YAML:
```sh
kubectl apply -f pod-definition.yaml
```

#### Verificación del estado del pod:
```sh
kubectl get pods
```

### Estrategias de despliegue de Pods
Existen varias estrategias para desplegar pods en Kubernetes:

- **Despliegue manual:** Creación de pods de manera manual usando archivos YAML.
- **Despliegue automatizado:** Uso de controladores como Deployments, ReplicaSets y DaemonSets para gestionar pods de manera automática y escalable.

#### Ejemplo de despliegue manual:
```sh
kubectl run nginx --image=nginx --port=80
```

#### Uso de Deployments para automatizar el despliegue:
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

#### Aplicación del Deployment:
```sh
kubectl apply -f deployment.yaml
```

## Deployments y Scaling

Los Deployments son uno de los recursos más importantes en Kubernetes para gestionar aplicaciones de manera declarativa. Un Deployment permite definir el estado deseado de las aplicaciones y Kubernetes se encargará de mantener ese estado a lo largo del tiempo.

### Conceptos clave de los Deployments

- **Declarativo:** Los Deployments permiten definir el estado deseado de las aplicaciones de forma declarativa. Se escribe un archivo YAML que describe el estado deseado, y Kubernetes se encarga de hacerlo realidad.
- **Gestión del ciclo de vida:** Un Deployment gestiona la creación y eliminación de Pods, asegurando que el número correcto de Pods esté siempre en ejecución.
- **Actualizaciones y Retrocesos:** Permite actualizar las aplicaciones sin tiempo de inactividad y volver a versiones anteriores si algo sale mal.
- **Escalabilidad:** Facilita el escalado de aplicaciones, tanto horizontal como automáticamente utilizando el Horizontal Pod Autoscaler.

### Estrategias de despliegue

Kubernetes ofrece diferentes estrategias de despliegue para gestionar cómo se actualizan los Pods:

- **Recreate:** Esta estrategia elimina todos los Pods existentes antes de crear nuevos Pods con la versión actualizada. Es simple pero puede causar tiempo de inactividad.
  
  ```yaml
  spec:
    strategy:
      type: Recreate
  ```

- **RollingUpdate:** Esta estrategia actualiza los Pods gradualmente, minimizando el tiempo de inactividad y asegurando que siempre haya un conjunto de Pods disponibles. Kubernetes controla el número máximo de Pods que pueden estar fuera de servicio (maxUnavailable) y el número máximo de nuevos Pods que se pueden crear (maxSurge) durante la actualización.

  ```yaml
  spec:
    strategy:
      type: RollingUpdate
      rollingUpdate:
        maxUnavailable: 1
        maxSurge: 1
  ```

#### Ejemplo de definición de un Deployment en YAML:
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

### Actualizar un Deployment
Para actualizar un Deployment, simplemente se modifica el archivo YAML con la nueva configuración y se aplica nuevamente. También se puede usar el comando `kubectl set image` para actualizar la imagen del contenedor.

#### Ejemplo de actualización de imagen:
```sh
kubectl set image deployment/my-deployment my-container=nginx:1.16.1
```

### Autoescalado de aplicaciones (Horizontal Pod Autoscaler)

El autoescalado horizontal (Horizontal Pod Autoscaler, HPA) permite ajustar automáticamente el número de Pods en un Deployment en función de la carga del sistema, como el uso de CPU o memoria. Esto es crucial para manejar la variabilidad en la carga de trabajo y asegurar que la aplicación siempre tenga los recursos necesarios para funcionar de manera óptima.

#### Crear un HPA:
```sh
kubectl autoscale deployment my-deployment --cpu-percent=50 --min=1 --max=10
```

Este comando configura un HPA que ajustará el número de réplicas del Deployment `my-deployment` para mantener el uso de CPU en aproximadamente el 50%. El número de réplicas variará entre 1 y 10, según la carga.

#### Verificar el estado del HPA:
```sh
kubectl get hpa
```

#### Ejemplo práctico:
Para escalar horizontalmente una aplicación en respuesta a la carga, configuramos un Horizontal Pod Autoscaler (HPA). Esto se hace definiendo un objeto HPA y asociándolo con un Deployment existente.

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

#### Comando para aplicar el HPA:
```sh
kubectl apply -f hpa.yaml
```

Esta guía proporciona una base sólida para gestionar recursos en Kubernetes, incluyendo la creación y gestión de pods, despliegues y el escalado automático de aplicaciones. Para un uso más avanzado y opciones adicionales.

Documentacion oficial de Kubernetes:
[Deployment](https://kubernetes.io/es/docs/concepts/workloads/controllers/deployment/)
[Horizontal Pod Autoscaling](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
