# Solución del Ejercicio 17 - Liveness y Readiness Probes en Kubernetes

## Enunciado

En este ejercicio, se utiliza la imagen `dboffelli/testing_probes:2.0`, una aplicación web en GO diseñada para probar el comportamiento de las sondas `readinessProbe` y `livenessProbe` en Kubernetes.

### Requisitos del despliegue:

1. Desplegar el contenedor utilizando la imagen proporcionada.
2. Configurar las sondas:
   - **Liveness Probe:** Verifica que el servidor está corriendo, consultando el endpoint `/`.
   - **Readiness Probe:** Verifica que el servidor está listo para recibir tráfico, consultando el endpoint `/healthy`.
3. Observar el comportamiento del contenedor con las sondas configuradas durante un minuto.



## Resolución

En primer lugar, debemos desplegar el contenedor utilizando la imagen proporcionada. Para ello, utilizaremos el archivo `deploy-probe.yaml`:
```yaml
# Configuración del Deployment con liveness y readiness probes
apiVersion: apps/v1           
kind: Deployment                
metadata:
  name: probes  # Nombre del deployment          
spec:
  replicas: 1  # Número de réplicas de pods                
  selector:
    matchLabels:
      app: probes  # Selector para identificar los pods gestionados por este deployment       
  template:                    
    metadata:
      labels:
        app: probes  # Etiqueta asignada a los pods para su identificación     
    spec:
      containers:
      - name: probes  # Nombre del contenedor     
        image: dboffelli/testing_probes:2.0  # Imagen Docker utilizada
        ports:
        - containerPort: 8080  # Puerto expuesto por el contenedor  
        livenessProbe:  # Configuración de la liveness probe        
          httpGet:  # Revisión basada en una solicitud HTTP             
            path: /  # Ruta a consultar para verificar la salud            
            port: 8080  # Puerto para la consulta          
          initialDelaySeconds: 15  # Tiempo de espera antes de realizar la primera consulta  
          periodSeconds: 10  # Intervalo entre consultas        
        readinessProbe:  # Configuración de la readiness probe       
          httpGet:
            path: /healthy  # Ruta a consultar para verificar si el pod está listo    
            port: 8080  # Puerto para la consulta
          initialDelaySeconds: 5  # Tiempo de espera antes de realizar la primera consulta  
          periodSeconds: 5  # Intervalo entre consultas         
```
En este archivo, se define un `Deployment` llamado `probes` que utiliza la imagen `dboffelli/testing_probes:2.0`. Se configuran las sondas `livenessProbe` y `readinessProbe` con diferentes intervalos de espera y periodos de consulta. Además, se establece un puerto expuesto por el contenedor en el puerto 8080. La `livenessProbe` está configurada con un `initialDelaySeconds` de 15 segundos para dar tiempo al servidor a iniciar. La `readinessProbe` verifica el endpoint `/healthy` para determinar si el pod está listo para recibir tráfico.

```bash
kubectl apply -f deploy-probe.yaml
kubectl get deployments
kubectl get pods
kubectl logs -f deployment/probes
kubectl describe pod -l app=probes
kubectl port-forward deployment/probes 8080:8080
```

Con esto comandos realizamos las siguientes tareas: 

- Desplegar el Deployment utilizando el archivo `deploy-probe.yaml`.
- Obtener información sobre los Deployments y Pods.
- Verificar los logs del Deployment y del Pod.
- Obtener información detallada del Pod utilizando la etiqueta `app=probes`.
- Realizar un port-forward para acceder al servicio expuesto por el contenedor en el puerto 8080.

![Captura sobre el código](../../datos/ejercicio%2017/comandos.png)

Tras realizar el port-forward, podemos acceder a la URL `http://localhost:8080` para verificar el comportamiento de las sondas.

![Captura sobre el código](../../datos/ejercicio%2017/ejecucion%20inicial.png)

![Captura sobre el código](../../datos/ejercicio%2017/no%20saludable.png)

![Captura sobre el código](../../datos/ejercicio%2017/saludable.png)


