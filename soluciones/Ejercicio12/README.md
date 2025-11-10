# Solución del Ejercicio 12

## Enunciado
Realiza el ejercicio 11 pero en este caso, en lugar de usar la estrategia RollingUpdate, usa la estrategia Recreate.

1. Crea un namespace con el nombre que prefieras.
2. Tomaremos el deployment del ejercicio 5 y lo modificaremos para que use la estrategia Recreate.
3. Despliega el deployment en el namespace creado.
4. Modifica el deployment para usar una tag diferente de la imagen.
5. Actualiza el deployment para que use la nueva imagen.
6. Explica el proceso de actualización de la imagen.
7. Limpia todos los recursos creados.


## Resolución

### 1. Crear el namespace

Creamos un namespace llamado `lcm` para aislar los recursos del ejercicio.

```bash
kubectl create namespace lcm
```

![Captura sobre el código](../../datos/ejercicio%2012/crear%20el%20namespace.png)


### 2. Deployment con estrategia Recreate

El archivo `deployment_demogat.yaml` contiene la configuración del deployment con la estrategia `Recreate`. Esta estrategia elimina los pods antiguos antes de crear los nuevos, a diferencia de `RollingUpdate` que actualiza los pods de forma gradual.

Contenido del archivo `deployment_demogat.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demogat-yaml  # Nombre del deployment
  namespace: lcm  # Namespace al que pertenece el deployment
spec:
  replicas: 1  # Número de réplicas de pods que se mantendrán
  strategy:
    type: Recreate  # Estrategia de despliegue utilizada (recrea los pods en lugar de actualizarlos de forma continua)
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

### 3. Desplegar el deployment

Aplicamos el deployment en el namespace `lcm`:

```bash
kubectl apply -f deployment_demogat.yaml
```

![Captura sobre el código](../../datos/ejercicio%2012/desplegar%20yaml.png)


### 4. Modificar la imagen del deployment

Actualizamos la imagen del contenedor `tomcat` a una nueva versión:

```bash
kubectl set image deployment/demogat-yaml tomcat=tomcat:11.0.0-M21 -n lcm
```

![Captura sobre el código](../../datos/ejercicio%2012/cambio%20imagen.png)

### 5. Verificar el estado del rollout

Comprobamos que el deployment se haya actualizado correctamente:

```bash
kubectl rollout status deployment/demogat-yaml -n lcm
kubectl describe deployment demogat-yaml -n lcm
```

![Captura sobre el código](../../datos/ejercicio%2012/rollout.png)
![Captura sobre el código](../../datos/ejercicio%2012/describe.png)


### 6. Explicación del proceso de actualización

La estrategia `Recreate` elimina todos los pods existentes antes de crear los nuevos con la imagen actualizada. Esto puede causar un tiempo de inactividad mientras no hay pods disponibles. Es útil cuando no se puede tener múltiples versiones corriendo simultáneamente.

### 7. Limpieza de recursos

Eliminamos el deployment y el namespace para limpiar el entorno:

```bash
kubectl delete deployment demogat-yaml -n lcm
kubectl get deployment -n lcm
kubectl delete namespace lcm
kubectl get namespace
```

![Captura sobre el código](../../datos/ejercicio%2012/borrar%20recursos.png)
