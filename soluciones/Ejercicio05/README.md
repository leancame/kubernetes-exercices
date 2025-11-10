# Solución del Ejercicio 05


## Enunciado
En esta ocasión no nos preocuparemos por los namespaces (ya llegaremos a eso) y trabajaremos en el namespace por defecto. Si no se piden otros, usaremos solo los elementos estrictamente necesarios.

1. Crea un manifiesto de un DEPLOYMENT con las siguientes características:
- Nombre: demogat-yaml
- Imagen: tomcat:11.0.0-M20
- Nombre de contendor: tomcat
- Usaremos la etiqueta app:gato, para el match
2. Crea este de manera DECLARATIVA.
3. Lista los POD.
4. Lista los DEPLOYMENT.
5. En este caso NO BORRAR EL DEPLOY, YA QUE SE USARÁ PARA EL PRÓXIMO EJERCICIO.


## Resolución

En este supuesto se nos solicita la creación de un Deployment de forma declarativa, es decir, a partir de un manifiesto YAML. El manifiesto `deployment_demogat.yaml` crea un Deployment de Kubernetes con las siguientes características:

```yaml
# Especifica la versión de la API que se usará para manejar este recurso
apiVersion: apps/v1 
# Define el tipo de recurso, en este caso, un Deployment
kind: Deployment 
# Metadatos del Deployment, como su nombre único dentro del namespace
metadata:
  name: demogat-yaml  # Nombre del Deployment
# Especificación del Deployment
spec:
  replicas: 1  # Número de réplicas de pods que se deben mantener activas
  selector:
    matchLabels:
      app: gato  # Etiqueta que selecciona los pods gestionados por este Deployment
  template:
    metadata:
      labels:
        app: gato  # Etiqueta asignada a los pods creados por este template
    spec:
      containers:
      - name: tomcat  # Nombre del contenedor dentro del pod
        image: tomcat:11.0.0-M20  # Imagen Docker para este contenedor (Tomcat versión específica)
```
Este manifiesto crea un Deployment llamado `demogat-yaml` con una sola réplica, que selecciona los pods con la etiqueta `app: gato`. El contenedor `tomcat` ejecuta la imagen oficial de Tomcat con la etiqueta `tomcat:11.0.0-M20`. 

Para aplicar el manifiesto y verificar el estado de los recursos, se ejecutaron los siguientes comandos:

```bash
kubectl apply -f deployment_demogat.yaml
kubectl get pods
kubectl get deployments
```

El primero de ellos aplica el manifiesto, creando el Deployment y los pods correspondientes. El segundo comando muestra la lista de pods y el tercer comando muestra la lista de Deployments.

A continuación se incluyen capturas de pantalla que documentan el proceso:

- Creación del manifiesto YAML y aplicación del Deployment:

  ![Captura sobre el código](../../datos/ejercicio%205/crear%20el%20yaml.png)

- Listado de pods y deployments después de aplicar el manifiesto:

  ![Captura sobre el código](../../datos/ejercicio%205/listar%20en%20pods%20y%20deve.png)


