# Solución del Ejercicio 07

## Enunciado

1. Realizaremos el ejercicio del vídeo de las clases 89/90.
2. Documentar todo el proceso.


## Solución

Antes de empezar con el ejercicio como tal debemos poseer una imagen de docker como nos muestra el propio ejercicio. Por lo tanto, debemos crear la imagen y subirla a docker hub. Para ello usaremos el archivo Dockerfile que se encuentra en la carpeta del ejercicio. Comenzamos creando la imagen usando `docker build -t lcarbajo/kubernetes .`. En mi caso he probado si la imagen funciona y la he usado para crear un contenedor local con `docker run --name=web1 -d -p 9090:80 lcarbajo/kubernetes` mostrando el funcionamiento de forma correcta como aparece en el vídeo. Para detenerlo usamos `docker stop web1`.

![Captura sobre el código](../../datos/ejercicio%207/imagen%20de%20docker.png)

![Captura sobre el código](../../datos/ejercicio%207/docker%20run.png)

![Captura sobre el código](../../datos/ejercicio%207/ejecucion%20de%20docker.png)

![Captura sobre el código](../../datos/ejercicio%207/subir%20la%20imagen%20de%20docker%20a%20doker%20hub.png)

![Captura sobre el código](../../datos/ejercicio%207/stop%20web%201.png)

Tras su dessarrollo de forma local, vamos a pasar a subir la imagen a Docker Hub. Para ello debemos registrarnos y subir la imagen. Para ello usaremos el comando `docker login` y luego `docker push lcarbajo/kubernetes:latest`.

![Captura sobre el código](../../datos/ejercicio%207/subir%20la%20imagen%20de%20docker%20a%20doker%20hub.png)

![Captura sobre el código](../../datos/ejercicio%207/imagen%20en%20docker%20hub.png)

Una vez subida la imagen a Docker Hub, podemos crear un deploy sobre esa imagen. Para ello usaremos el comando `kubectl apply -f deploy-service.yaml`, luego `kubectl get pods` y `kubectl get svc` para obtener la informacion de los pods y los servicios.

Nuestro archivo `deploy-service.yaml` es el siguiente:

```yaml
# Configuración del Deployment para la aplicación web
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-d  # Nombre del deployment
spec:
  selector:
    matchLabels:
      app: web  # Etiqueta que deben coincidir los pods gestionados por este deployment
  replicas: 2  # Número de réplicas de pods que se mantendrán
  template:
    metadata:
      labels:
        app: web  # Etiqueta asignada a los pods para su identificación
    spec:
      containers:
      - name: apache  # Nombre del contenedor
        image: lcarbajo/kubernetes:latest  # Imagen Docker utilizada para el contenedor
        ports:
        - containerPort: 80  # Puerto expuesto por el contenedor
---
# Configuración del Service para exponer el deployment
apiVersion: v1
kind: Service
metadata:
  name: web-svc  # Nombre del servicio
spec:
  type: NodePort  # Tipo de servicio expuesto a través de un NodePort
  ports:
  - port: 80  # Puerto del servicio
    nodePort: 30002  # Puerto del nodo mapeado al puerto del servicio
  selector:
    app: web  # Selector para asociar el servicio con los pods
```
En el archivo anterior, hemos creado un deployment con 2 réplicas de pods y un servicio que expone el puerto 80 del contenedor. Le damos un nombre a nuestro deployment con el nombre `web-d` y a nuestro servicio con el nombre `web-svc`. Además, como hemos ido planteando antes, hemos usado nuestra imagen de Docker `lcarbajo/kubernetes:latest` para el contenedor. Por último, hemos usado un servicio que utiliza un NodePort para exponer el puerto 80 del contenedor a través de un puerto mapeado en el nodo. En este caso, el puerto mapeado es el 30002. 


Muestra de los comandos:

![Captura sobre el código](../../datos/ejercicio%207/apply.png)

![Captura sobre el código](../../datos/ejercicio%207/get%20pod.png)

![Captura sobre el código](../../datos/ejercicio%207/get%20svc.png)


En este caso, una vez ya levantado el pod y el servicio, podemos ver que el servicio está expuesto a través de un NodePort en el puerto 30002. Aunque, se haya expuesto a través de un NodePort, no podemos acceder a él directamente desde fuera de la máquina debido a que la IP que obtienes con `minikube ip` pertenece a la red interna de Minikube y puede no ser accesible directamente desde tu máquina anfitriona incluso con el puerto expuesto. Por eso, he tenido que crear un tunel para poder acceder al servicio desde fuera de la máquina. Con el comando `minikube service web-svc` se crea un tunel que permite acceder al servicio porque maneja automáticamente la complejidad de las redes entre Minikube y tu máquina anfitriona


![Captura sobre el código](../../datos/ejercicio%207/get%20service.png)

![Captura sobre el código](../../datos/ejercicio%207/ejecucion%20con%20kubernete.png)
