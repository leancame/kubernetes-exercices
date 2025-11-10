# Solución del Ejercicio 11

## Enunciado
1. Crea un namespace con el nombre que prefieras.
2. Tomaremos el deployment del ejercicio 5 y lo modificaremos para que use la estrategia RollingUpdate.
3. Despliega el deployment en el namespace creado.
4. Modifica el deployment para usar una tag diferente de la imagen.
5. Actualiza el deployment para que use la nueva imagen.
6. Explica el proceso de actualización de la imagen.
7. Limpia todos los recursos creados.

## Resolución

### 1. Crear el namespace
Creamos un namespace llamado `lcm` :

```bash
kubectl create namespace lcm
```
![Captura sobre el código](../../datos/ejercicio%2011/crear%20el%20namespace.png)

### 2. Descripción del Deployment

El archivo `deployment_demogat.yaml` define un Deployment llamado `demogat-yaml` en el namespace `lcm`.  
Este deployment tiene las siguientes características:

- 2 réplicas de pods.
- Estrategia de despliegue: `RollingUpdate`, que permite actualizar los pods sin downtime.
- Selector y etiquetas para los pods con `app: gato`.
- Contenedor `tomcat` usando la imagen `tomcat:11.0.0-M20`.

Contenido relevante del YAML:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demogat-yaml  # Nombre del deployment
  namespace: lcm  # Namespace al que pertenece el deployment
spec:
  replicas: 2  # Número de réplicas de pods que se mantendrán
  strategy:
    type: RollingUpdate  # Estrategia de despliegue utilizada (actualización continua)
  selector:
    matchLabels:
      app: gato  # Selector para identificar los pods gestionados por este deployment
  template:
    metadata:
      labels:
        app: gato  # Etiqueta asignada a los pods para su identificación
    spec:
      containers:
      - name: tomcat  # Nombre del contenedor
        image: tomcat:11.0.0-M20  # Imagen Docker utilizada para Tomcat
```

### 3. Desplegar el Deployment

Aplicamos el deployment en el namespace `lcm`:

```bash
kubectl apply -f deployment_demogat.yaml
```
![Captura sobre el código](../../datos/ejercicio%2011/apply%20demogat.png)

### 4. Modificar la imagen del Deployment

Actualizamos la imagen del contenedor `tomcat` a una nueva versión, por ejemplo `tomcat:11.0.0-M21` donde el `set image` nos permite actualizar la imagen de un contenedor específico en un deployment existente en un namespace usando `-n namespace`:

```bash
kubectl set image deployment/demogat-yaml tomcat=tomcat:11.0.0-M21 -n lcm
```
![Captura sobre el código](../../datos/ejercicio%2011/modificar%20imagen.png)

Podemos verificar el cambio con:

```bash
kubectl describe deployment demogat-yaml -n lcm
```
![Captura sobre el código](../../datos/ejercicio%2011/describir%20el%20yaml%20para%20que%20veas%20la%20imagen%20modificada.png)

### 5. Verificar el estado del rollout

Para asegurarnos que la actualización se realizó correctamente y sin downtime, usamos:

```bash
kubectl rollout status deployment/demogat-yaml -n lcm
```
![Captura sobre el código](../../datos/ejercicio%2011/rolluot.png)

### 6. Explicación del proceso de actualización (Rolling Update)

La estrategia `RollingUpdate` permite actualizar los pods de forma gradual, reemplazando las réplicas antiguas por nuevas sin interrumpir el servicio. Kubernetes crea nuevos pods con la nueva imagen y elimina los antiguos una vez que los nuevos están listos y saludables. Esto garantiza alta disponibilidad durante la actualización.

### 7. Limpieza de recursos

Para eliminar el deployment y el namespace creado:

```bash
kubectl delete deployment demogat-yaml -n lcm
kubectl delete namespace lcm
```

Podemos verificar que se eliminaron con:

```bash
kubectl get deployments -n lcm
kubectl get namespaces
```
![Captura sobre el código](../../datos/ejercicio%2011/borrar%20recursos%201.png)

![Captura sobre el código](../../datos/ejercicio%2011/borrar%20recursos%202.png)