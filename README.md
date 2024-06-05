# Escalado Automático con Kubernetes

En este documento, vamos a explorar cómo implementar el escalado automático en Kubernetes utilizando un Horizontal Pod Autoscaler (HPA). También veremos cómo desplegar una aplicación, exponerla a través de un servicio y ejecutar un generador de carga para simular tráfico. Finalmente, aprenderemos cómo habilitar el Metrics Server en Minikube, esencial para el funcionamiento del HPA.

## Tabla de Contenidos
1. [Configuración del Horizontal Pod Autoscaler (HPA)](#1-configuración-del-horizontal-pod-autoscaler-hpa)
2. [Despliegue de la Aplicación y el Servicio](#2-despliegue-de-la-aplicación-y-el-servicio)
3. [Observando el HPA](#3-observando-el-hpa)
4. [Simulación de Tráfico](#4-simulación-de-tráfico)
5. [Habilitación de Metrics Server en Minikube](#5-habilitación-de-metrics-server-en-minikube)

## 1. Configuración del Horizontal Pod Autoscaler (HPA)

El HPA es una característica clave de Kubernetes que ajusta automáticamente el número de réplicas de un Deployment basado en la carga de la CPU o la memoria. Permite mantener un rendimiento óptimo de la aplicación al escalar automáticamente el número de pods en función de la demanda.

### Código YAML para definir un HPA:
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

En este HPA:
- `minReplicas`: Especifica el número mínimo de réplicas que el HPA mantendrá.
- `maxReplicas`: Especifica el número máximo de réplicas que el HPA permitirá.
- `targetCPUUtilizationPercentage`: Define el objetivo de utilización de CPU para el HPA. Por ejemplo, un valor de 50 significa que el HPA intentará mantener el uso de CPU en aproximadamente el 50% en todos los pods.

### Aplicar el HPA:
Para aplicar el HPA, guardamos el YAML anterior en un archivo `hpa.yaml` y ejecutamos:
```sh
kubectl apply -f hpa.yaml
```

## 2. Despliegue de la Aplicación y el Servicio

Vamos a desplegar una aplicación Nginx y exponerla a través de un servicio para que pueda ser accesible externamente. Esto se hace utilizando dos recursos de Kubernetes: un Deployment y un Service.

### Código YAML para el Deployment y el Service:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 1
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
          resources:
            requests:
              memory: "16Mi"
              cpu: "62.5m"
            limits:
              memory: "32Mi"
              cpu: "125m"
```

Este código define:
- Un **Service** llamado `my-app` que utiliza un selector para encontrar los pods con la etiqueta `app: my-app` y los expone en el puerto 80 a través de un balanceador de carga.
- Un **Deployment** que despliega una réplica de un contenedor Nginx con recursos solicitados y límites definidos.

### Aplicar el Deployment y el Service:
Guardamos el YAML anterior en un archivo `deployment-service.yaml` y ejecutamos:
```sh
kubectl apply -f deployment-service.yaml
```

## 3. Observando el HPA

Una vez desplegado el HPA, puedes observar su comportamiento utilizando el siguiente comando:

```sh
kubectl get hpa my-app-hpa --watch
```

Este comando te proporcionará información en tiempo real sobre la escala de réplicas del Deployment en función de la carga de CPU.

## 4. Simulación de Tráfico

Para simular tráfico en nuestra aplicación y ver cómo responde el escalado automático, ejecutamos un generador de carga. Esto ayudará a verificar que el HPA está funcionando correctamente al aumentar la carga en la aplicación.

### Comando para generar tráfico:
```sh
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://my-app; done"
```

Este comando crea un pod temporal que realiza solicitudes continuas a la URL del servicio `my-app`, generando carga continua.

## 5. Habilitación de Metrics Server en Minikube

Antes de utilizar el Horizontal Pod Autoscaler (HPA), necesitamos asegurarnos de que Metrics Server esté habilitado en Minikube. Metrics Server proporciona recursos métricos de los nodos y los pods del clúster, lo que es fundamental para que el HPA funcione correctamente.

### Comando para habilitar Metrics Server:
```sh
minikube addons enable metrics-server
```

Este comando activará Metrics Server en tu clúster Minikube, permitiendo que el HPA acceda a las métricas necesarias para ajustar automáticamente el número de réplicas de tus pods. Asegúrate de ejecutar este comando antes de aplicar el HPA.

---

Con estos pasos, has implementado con éxito el escalado automático en Kubernetes y has observado cómo responde tu aplicación a las fluctuaciones de la carga de trabajo. El uso de HPA junto con Metrics Server asegura que tus aplicaciones puedan manejar la variabilidad en la carga de manera eficiente, garantizando un rendimiento óptimo y una utilización eficiente de los recursos.
