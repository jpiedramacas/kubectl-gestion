## Escalado Automático con Kubernetes

En este documento, vamos a explorar cómo implementar el escalado automático en Kubernetes utilizando un Horizontal Pod Autoscaler (HPA). También veremos cómo desplegar una aplicación, exponerla a través de un servicio y ejecutar un generador de carga para simular tráfico.

### 1. Configuración del Horizontal Pod Autoscaler (HPA)

El HPA es una característica clave de Kubernetes que ajusta automáticamente el número de réplicas de un Deployment basado en la carga de la CPU o la memoria. Aquí está el código YAML para definir un HPA:

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
  targetCPUUtilizationPercentage: 10
```

En este HPA:

- `minReplicas`: Especifica el número mínimo de réplicas que el HPA mantendrá.
- `maxReplicas`: Especifica el número máximo de réplicas que el HPA permitirá.
- `targetCPUUtilizationPercentage`: Define el objetivo de utilización de CPU para el HPA.

### 2. Despliegue de la Aplicación y el Servicio

Vamos a desplegar una aplicación Nginx y exponerla a través de un servicio para que pueda ser accesible externamente. Aquí está el código YAML:

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
              cpu: "62.5m"
```

Este código despliega un pod de Nginx y un servicio `my-app` del tipo LoadBalancer, que expone la aplicación a través de un balanceador de carga externo en el puerto 80.

### 3. Observando el HPA

Una vez desplegado el HPA, puedes observar su comportamiento utilizando el siguiente comando:

```bash
kubectl get hpa my-app-hpa --watch
```

Este comando te proporcionará información en tiempo real sobre la escala de réplicas del Deployment en función de la carga de CPU.

### 4. Simulación de Tráfico

Para simular tráfico en nuestra aplicación y ver cómo responde el escalado automático, ejecutamos un generador de carga:

```bash
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://my-app; done"
```

Este comando generará tráfico continuo hacia la URL del servicio `my-app`, lo que debería activar el escalado automático si la carga de CPU aumenta por encima del 10%.

Con estos pasos, has implementado con éxito el escalado automático en Kubernetes y has observado cómo responde tu aplicación a las fluctuaciones de la carga de trabajo.
