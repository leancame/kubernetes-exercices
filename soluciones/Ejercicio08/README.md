### Ejercicio 08

#### Enunciado
En esta ocasión, no nos preocuparemos por los namespaces (ya llegaremos a eso) y trabajaremos en el namespace por defecto. Si no se piden otros, usaremos solo los elementos estrictamente necesarios.

Crea un manifiesto de un **DEPLOYMENT** con las siguientes características:

- **Nombre**: `colors`.
- **Imagen**: `noloknolo/colors`.
- **Nombre del contenedor**: `coloread`.
- Usa la etiqueta que quieras para el match.
- **3 réplicas**.
- Investigar cómo conseguir el puerto que usa la imagen y exponerlo.
- Despliega este de manera **DECLARATIVA**.

Crea un manifiesto **SERVICE** para el **DEPLOYMENT** anterior con las siguientes características:

- **Nombre**: `colors-service`.
- **Tipo**: `ClusterIP`.
- **Puerto**: el puerto que has averiguado antes.
- Muestra los comandos necesarios para documentar el proceso y que todo funcione correctamente.

Conecta el servicio a un puerto local utilizando **`minikube service`** en lugar de `port-forward` para acceder al servicio desde un navegador de manera similar a un entorno real.

`bash`
minikube service colors-service

Esto abrirá una URL en el navegador que te permitirá acceder al servicio expuesto. Recarga la página varias veces para observar el comportamiento del servicio y registra lo que muestra el navegador (capturas de pantalla).



## Resolución

Como en este supuesto debemos usar una imagen de docker personalizada debemos saber dos aspectos importantes:

- Conocer la etiqueta correcta de la imagen para poderla usar en el manifiesto.
- Averiguar el puerto que se expone en la imagen.

Para conocer la etiqueta de la imagen podemos buscar en el propio Docker Hub dentro del repositorio de la imagen. En este caso, la imagen es `noloknolo/colors:v1` y a raíz de esta podemos usarla para nuestro manifiesto. Respecto al puerto, la forma sencilla es viendo el tag de la imagen que nos muestra el puerto expuesto. No obstante, he decidido descargar la propia imagen para poder averiguar el puerto expuesto. Para ello he usado el comando `docker pull --quiet noloknolo/colors:v1` descargando la imagen de forma silenciosa o con una salida minimizada. Una vez descargada la imagen, he usado el comando `docker inspect noloknolo /colors:v1` para obtener información sobre el puerto, otra forma más simple es usar `docker inspect noloknolo /colors:v1 | findstr ExposedPorts` donde `findstr` es un comando de Windows que busca concretamente la etiqueta que deseamos buscar y saber si existe la exposición del puerto. 


![Captura sobre el código](../../datos/ejercicio%208/docker%20hub%20puerto.png)

![Captura sobre el código](../../datos/ejercicio%208/docker%20push%20--quiet.png)

![Captura sobre el código](../../datos/ejercicio%208/findstr%20ExposedPorts.png)

![Captura sobre el código](../../datos/ejercicio%208/docker%20inspects.png)

Al conocer el puerto, podemos ya realizar el deployment y servicio con los datos concretos. Pero, antes de realizarlo, he decidido realizar la imagen en local para observar su funcionamiento. Usé el comando `docker run -p 5000:8080 noloknolo/colors:v1` para crear un contenedor y exponer el puerto 8080 en el puerto 5000 de la máquina local. Una vez creado el contenedor, he podido acceder a él mediante el navegador en `http://localhost:5000` y he podido observar el funcionamiento de este. 

![Captura sobre el código](../../datos/ejercicio%208/docker%20run.png)

Ya hecho la prueba, paso al desarrollo del yaml con estos datos:

```yaml
# Configuración del Deployment para la aplicación "colors"
apiVersion: apps/v1
kind: Deployment
metadata:
  name: colors  # Nombre del deployment
spec:
  replicas: 3  # Número de réplicas de pods que se mantendrán
  selector:
    matchLabels:
      app: colors  # Etiqueta que deben coincidir los pods gestionados por este deployment
  template:
    metadata:
      labels:
        app: colors  # Etiqueta asignada a los pods para su identificación
    spec:
      containers:
      - name: coloread  # Nombre del contenedor
        image: noloknolo/colors:v1  # Imagen Docker utilizada para el contenedor
        ports:
        - containerPort: 8080  # Puerto expuesto por el contenedor

---

# Configuración del Service para la aplicación "colors"
apiVersion: v1
kind: Service
metadata:
  name: colors-service  # Nombre del servicio
spec:
  type: ClusterIP  # Tipo de servicio (solo accesible dentro del clúster)
  ports:
  - port: 8080  # Puerto del servicio expuesto
    protocol: TCP  # Protocolo utilizado (TCP en este caso)
    targetPort: 8080  # Puerto objetivo dentro del contenedor
  selector:
    app: colors  # Selector para asociar el servicio con los pods
```
Cumpliendo con los requisitos descritos, le damos el puerto investigado anterioremente, los nombres deseados a contenedor, al deploy, al servicio, etc. Además, añadimos el tipo de servicio (`ClusterIP`) para que solo se pueda acceder desde dentro del clúster. Continuamos con la ejecución del deployment con el comando `kubectl apply -f deploy_color.yaml`. Me dio error en el deployment, pero me creo correctamente el servicio, por eso al aplicarlo de nuevo no se muestra cambio en el servicio.

![Captura sobre el código](../../datos/ejercicio%208/apply.png)

Mostramos que todo se ha creado correctamente con el comando `kubectl get deployments, Kubectl get services y kubectl get pods`.

![Captura sobre el código](../../datos/ejercicio%208/pod,service,deploy.png)

Para poder acceder a la aplicación, utilizamos el comando `minikube service colors-service`

![Captura sobre el código](../../datos/ejercicio%208/minikube%20service.png)

Y obtenemos la URL que nos muestra el navegador y mostramos la ejecución de las diversas peticiones y pods mostrando cada uno un color cuando recargamos la página.

![Captura sobre el código](../../datos/ejercicio%208/ejecucion%201.png)

![Captura sobre el código](../../datos/ejercicio%208/ejecucion%202.png)

![Captura sobre el código](../../datos/ejercicio%208/ejecucion%203.png)

