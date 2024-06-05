# Gestión de Recursos en Kubernetes

Este documento detalla los conceptos y pasos para la gestión de recursos en Kubernetes, con un enfoque específico en los Deployments y el escalado de aplicaciones. Se explicarán los comandos y archivos YAML necesarios para la correcta configuración y manejo de los recursos dentro de un clúster de Kubernetes.

## Tabla de Contenidos
1. [Introducción](#introducción)
2. [Deployments y Scaling](#deployments-y-scaling)
   - [Estrategias de despliegue](#estrategias-de-despliegue)
   - [Autoescalado de aplicaciones (Horizontal Pod Autoscaler)](#autoescalado-de-aplicaciones-horizontal-pod-autoscaler)

## Introducción

La gestión de recursos en Kubernetes es crucial para el despliegue, mantenimiento y escalado eficiente de aplicaciones en un clúster. En esta sección se abordan los Deployments y el escalado de aplicaciones, fundamentales para la gestión del ciclo de vida de las aplicaciones.

## Deployments y Scaling

### Estrategias de despliegue

Los Deployments permiten gestionar el ciclo de vida de las aplicaciones, incluyendo el escalado y las actualizaciones.

#### Estrategias de Despliegue

- **Recreate:** Elimina todos los Pods existentes antes de crear nuevos Pods con la versión actualizada.
- **RollingUpdate:** Actualiza los Pods gradualmente para minimizar el tiempo de inactividad.

#### Ejemplo de RollingUpdate en un Deployment

Para implementar una estrategia de RollingUpdate en un Deployment, se define en el archivo YAML de la siguiente manera:

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

En este ejemplo:
- `type: RollingUpdate`: Define que se usará la estrategia RollingUpdate.
- `maxUnavailable: 1`: Define el número máximo de Pods que pueden estar no disponibles durante la actualización.
- `maxSurge: 1`: Define el número máximo de Pods adicionales que pueden crearse más allá del número deseado de Pods durante la actualización.

#### Comando para actualizar un Deployment

Para actualizar la imagen de un contenedor en un Deployment existente, se utiliza el siguiente comando:

```sh
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
```

Este comando actualiza el Deployment `nginx-deployment` para usar la imagen `nginx:1.16.1`.

#### Ejemplo de definición de un Deployment en YAML

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

En este ejemplo:
- `apiVersion: apps/v1`: Especifica la versión de la API de Kubernetes para aplicaciones.
- `kind: Deployment`: Define que se está creando un Deployment.
- `metadata`: Contiene información sobre el Deployment, como su nombre (`my-deployment`) y etiquetas.
- `spec`: Describe las características del Deployment.
  - `replicas: 3`: Número de réplicas del Pod que se desean.
  - `selector`: Selector de etiquetas para identificar los Pods gestionados por este Deployment.
    - `matchLabels`: Etiquetas que deben coincidir.
      - `app: my-app`: Etiqueta que deben tener los Pods.
  - `template`: Plantilla para los Pods creados por el Deployment.
    - `metadata`: Etiquetas aplicadas a los Pods.
      - `labels`: Etiquetas de los Pods.
        - `app: my-app`: Etiqueta `app` con valor `my-app`.
    - `spec`: Descripción de los contenedores en los Pods.
      - `containers`: Lista de contenedores.
        - `name: my-container`: Nombre del contenedor.
        - `image: nginx:latest`: Imagen del contenedor.
        - `ports`: Lista de puertos expuestos por el contenedor.
          - `containerPort: 80`: Puerto expuesto por el contenedor.

#### Comando para aplicar el Deployment

```sh
kubectl apply -f deployment.yaml
```
Este comando crea o actualiza un Deployment basado en las especificaciones del archivo `deployment.yaml`.

### Autoescalado de aplicaciones (Horizontal Pod Autoscaler)

El autoescalado horizontal (Horizontal Pod Autoscaler, HPA) permite escalar automáticamente el número de Pods en función de la carga.

#### Crear un HPA

Para crear un Horizontal Pod Autoscaler, se usa el siguiente comando:

```sh
kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=1 --max=10
```

Este comando configura un HPA para el Deployment `nginx-deployment`, ajustando automáticamente el número de réplicas entre 1 y 10 en función del uso de CPU, manteniendo un promedio de uso de CPU del 50%.

#### Verificar el estado del HPA

Para verificar el estado del HPA, se utiliza el siguiente comando:

```sh
kubectl get hpa
```

Este comando muestra el estado actual de todos los HPA en el clúster.

#### Ejemplo Práctico

Para escalar horizontalmente una aplicación en respuesta a la carga, configuramos un HPA definiendo un objeto HPA y asociándolo con un Deployment existente.

**Ejemplo de archivo YAML para un HPA:**

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

En este ejemplo:
- `apiVersion: autoscaling/v1`: Especifica la versión de la API de Kubernetes para autoescalado.
- `kind: HorizontalPodAutoscaler`: Define que se está creando un HPA.
- `metadata`: Contiene información sobre el HPA, como su nombre (`webapp-hpa`).
- `spec`: Describe las características del HPA.
  - `scaleTargetRef`: Referencia al Deployment que será escalado.
    - `apiVersion: apps/v1`: Versión de la API del Deployment.
    - `kind: Deployment`: Tipo de recurso que será escalado.
    - `name: webapp`: Nombre del Deployment que será escalado.
  - `minReplicas: 2`: Número mínimo de réplicas.
  - `maxReplicas: 10`: Número máximo de réplicas.
  - `targetCPUUtilizationPercentage: 50`: Uso de CPU objetivo para el autoescalado.

#### Comando para aplicar el HPA

```sh
kubectl apply -f hpa.yaml
```
Este comando crea o actualiza un HPA basado en las especificaciones del archivo `hpa.yaml`.

---

## Servicios y Networking

Los servicios en Kubernetes permiten exponer aplicaciones en el clúster y gestionar el tráfico de red.

### Tipos de servicios (ClusterIP, NodePort, LoadBalancer)

- **ClusterIP:** Exponiendo el servicio dentro del clúster.
- **NodePort:** Exponiendo el servicio en cada nodo del clúster en un puerto estático.
- **LoadBalancer:** Exponiendo el servicio utilizando un balanceador de carga externo.

#### Ejemplo de servicio ClusterIP:

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

Este ejemplo define un servicio llamado `my-service` con un tipo `ClusterIP`, exponiendo el servicio dentro del clúster en el puerto 80, y redirigiendo el tráfico al puerto 9376 de los Pods que coincidan con la etiqueta `app: my-app`.

#### Crear el servicio:

```sh
kubectl apply -f service.yaml
```

Este comando crea el servicio `my-service` basado en las especificaciones del archivo `service.yaml`.

---

Este documento cubre los fundamentos para gestionar recursos en Kubernetes, incluyendo Deployments, el escalado de aplicaciones y servicios de red. Los ejemplos y comandos proporcionados ofrecen una guía práctica para implementar y mantener aplicaciones en un clúster de Kubernetes.
