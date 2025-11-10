# Solución del Ejercicio 09

## Enunciado

1. Realizaremos el ejercicio propuesto en la sección 10.
2. Documentar todo el proceso.

## Resolución

Antes de ponernos a trabajar con el ejercicio, debemos comentar un aspecto de los vídeos. Siguiendo la ruta que se hace en este ejercicio cabe deducir un error en la conexión entre la maestra y las esclavas debido a las etiquetas que se usan en el vídeo ya que estas han sido paulatinamente cambiando a lo largo de los años. Actualmente, se usan etiquetas como `leader` y `follower` en lugar de `master` y `slave`. Esto lo podemos ver en la propia página de Redis (https://redis.io/docs/latest/operate/oss_and_stack/management/replication/) o incluso en discusiones de Gitbub y Reddit que solicitaban ya un cambio de estos términos por su ámbiente sociopolitico (https://github.com/redis/redis/issues/5335). Por ende, cuando intentamos seguir el ejercicio, nos encontramos con que no podemos conectar la maestra con las esclavas dando un error. Esto se soluciona con la modicación de la etiqueta que se usa en el video sustituyendo `leader` por `master` y `follower` por `slave` y con esto debería funcionar. Si es verdad que encontré la posibilidad de crear las etiquetas `master` y `slave` como variables de entorno e intentar usarlas desde ese punto, pero por falta de tiempo no pude probarlo. Además, gracias al compañero Javier García que comentó todo esto y paso incluso una guía, no obstante, también hablando con otros compañeros como Iván me pudo aportar otra del propio Kubernetes (https://kubernetes.io/docs/tutorials/stateless-application/guestbook/).

Esta actividad se resumen en conectar el maestro con las esclavas y posteriromente estos a un fronted lo que otorga la creación en la práctica de un gestor de notas donde iremos añadiendo las notas deseadas. Vamos a comenzar con el `leader`, luego con los `followers` y finalmente con el `fronted` tanto en el deployment como en el service.


Deploy del `leader`:

````yaml
# Configuración del Deployment para Redis en el rol de líder
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-leader  # Nombre del deployment
  labels:
    app: redis  # Nombre de la aplicación
    role: leader  # Rol del pod dentro del sistema (líder)
    tier: backend  # Nivel de la aplicación (backend)
spec:
  replicas: 1  # Número de réplicas del líder (una única instancia)
  selector:
    matchLabels:
      app: redis  # Selector para identificar los pods gestionados por este deployment
  template:
    metadata:
      labels:
        app: redis  # Etiqueta para los pods
        role: leader  # Rol específico de los pods (líder)
        tier: backend  # Nivel asignado a los pods
    spec:
      containers:
      - name: leader  # Nombre del contenedor
        image: "docker.io/redis:6.0.5"  # Imagen de Docker para Redis
        resources:
          requests:
            cpu: 100m  # Recursos mínimos de CPU requeridos
            memory: 100Mi  # Recursos mínimos de memoria requeridos
        ports:
        - containerPort: 6379  # Puerto expuesto por el contenedor
````


Service del `leader`:

````yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-leader  # Nombre del servicio
  labels:
    app: redis  # Nombre de la aplicación
    role: leader  # Rol del servicio dentro del sistema (líder)
    tier: backend  # Nivel de la aplicación (backend)
spec:
  ports:
  - port: 6379  # Puerto expuesto por el servicio
    targetPort: 6379  # Puerto objetivo dentro del contenedor
  selector:
    app: redis  # Selector para asociar el servicio con los pods correspondientes
    role: leader  # Rol del pod asociado
    tier: backend  # Nivel asignado a los pods
````

Como puedes ver, creamos un deployment para el `leader` con una única réplica y un contenedor que utiliza el imagen de Docker `docker.io/redis:6.0.5`. El servicio asociado al `leader` expone el puerto 6379 del contenedor. En resumen, hemos creado un deployment para el `leader` y un servicio que expone el puerto 6379 del contenedor. A todo esto le hemos añadido una serie de etiquetas que nos ayudarán a identificarlos. En este caso, solo se posee una réplica del `leader`.

Para aplicar el deployment y el servicio, utilizaremos los comandos `kubectl apply -f redis-leader-deployment.yaml` y `kubectl apply -f redis-leader-service.yaml` respectivamente. Una vez aplicados, podemos verificar el estado de los recursos con `kubectl get pods` y `kubectl get service`.

![Captura sobre el código](../../datos/ejercicio%209/creacion%20del%20master%20tanto%20deployment%20y%20services.png)

En el caso de los esclavos, se realizará de manera similar, pero con la diferencia de que se creará un deployment con múltiples réplicas y poseerá sus etiquetas e imagen correspondientes.

Deploy de los esclavos: 

````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-follower  # Nombre del deployment
  labels:
    app: redis  # Nombre de la aplicación
    role: follower  # Rol del pod dentro del sistema (seguidor)
    tier: backend  # Nivel de la aplicación (backend)
spec:
  replicas: 2  # Número de réplicas para los seguidores
  selector:
    matchLabels:
      app: redis  # Selector para identificar los pods gestionados por este deployment
  template:
    metadata:
      labels:
        app: redis  # Etiqueta para los pods
        role: follower  # Rol específico de los pods (seguidor)
        tier: backend  # Nivel asignado a los pods
    spec:
      containers:
      - name: follower  # Nombre del contenedor
        image: us-docker.pkg.dev/google-samples/containers/gke/gb-redis-follower:v2  # Imagen de Docker para Redis follower
        resources:
          requests:
            cpu: 100m  # Recursos mínimos de CPU requeridos
            memory: 100Mi  # Recursos mínimos de memoria requeridos
        ports:
        - containerPort: 6379  # Puerto expuesto por el contenedor
````

Service de los esclavos:

````yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-follower  # Nombre del servicio
  labels:
    app: redis  # Nombre de la aplicación
    role: follower  # Rol del servicio dentro del sistema (seguidor)
    tier: backend  # Nivel de la aplicación (backend)
spec:
  ports:
  - port: 6379  # Puerto expuesto por el servicio
  selector:
    app: redis  # Selector para asociar el servicio con los pods correspondientes
    role: follower  # Rol del pod asociado (seguidor)
    tier: backend  # Nivel asignado a los pods
````
Para aplicar tanto el deployment como el service, se utilizarán los comandos `kubectl apply -f redis-follower-deployment.yaml` y `kubectl apply -f redis-follower-service.yaml` respectivamente. Esto creará los recursos en el cluster de Kubernetes. Para verificar que los recursos se hayan creado correctamente, se puede utilizar el comando `kubectl get pods` y `kubectl get services`.

![Captura sobre el código](../../datos/ejercicio%209/creacion%20del%20exclavo%20tanto%20deployment%20y%20services.png)

Por último, nos faltaría crear el fronted que se vinculará con el servicio de los esclavos. Para ello, debemos crear un deployment y un servicio para el frontend. El deployment del frontend posee tres réplicas, un contendor con variabeles de entorno para la configuración de hosts y recursos de CPU y memoria, además de todas las etiquetas pertinentes para poder encontrarse en el cluster. Se puede crear de la siguiente manera:

````yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend  # Nombre del deployment
spec:
  replicas: 3  # Número de réplicas para la capa frontend
  selector:
    matchLabels:
        app: guestbook  # Selector para identificar los pods de la aplicación Guestbook
        tier: frontend  # Selector para la capa frontend
  template:
    metadata:
      labels:
        app: guestbook  # Etiqueta de la aplicación
        tier: frontend  # Etiqueta para la capa frontend
    spec:
      containers:
      - name: php-redis  # Nombre del contenedor
        image: us-docker.pkg.dev/google-samples/containers/gke/gb-frontend:v5  # Imagen de Docker para el frontend
        env:
        - name: GET_HOSTS_FROM  # Variable de entorno para la configuración de hosts
          value: "dns"  # Valor de la variable
        resources:
          requests:
            cpu: 100m  # Recursos mínimos de CPU requeridos
            memory: 100Mi  # Recursos mínimos de memoria requeridos
        ports:
        - containerPort: 80  # Puerto expuesto por el contenedor
````

Y el servicio del frontend que usará un NodePort se puede crear de la siguiente manera:

````yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend  # Nombre del servicio
  labels:
    app: guestbook  # Etiqueta de la aplicación
    tier: frontend  # Etiqueta para la capa frontend
spec:  
  type: NodePort  # Tipo de servicio expuesto a través de un NodePort
  ports:
  - port: 80  # Puerto expuesto por el servicio
  selector:
    app: guestbook  # Selector para asociar el servicio con los pods correspondientes
    tier: frontend  # Selector para la capa frontend
````
Para crear y verificar el deployment y el servicio, he utilizado el comando `kubectl apply` con el archivo YAML correspondiente:

````bash
kubectl apply -f frontend-deployment.yaml
kubectl get pods -l app=guestbook -l tier=frontend

kubectl apply -f frontend-service.yaml
kubectl get service
````

![Captura sobre el código](../../datos/ejercicio%209/creacion%20del%20front%20tanto%20deployment%20y%20services.png)

Una vez realizado todo los archivos y pasos correspondientes, decidí crear un tunel para poder observar el resultado de la aplicación, para ello utilicé el comando `minikube service frontend` y me apareció el siguiente resultado porbando si guardaba las notas no como en el caso de usar etiquetas como master y slave:

![Captura sobre el código](../../datos/ejercicio%209/tunel.png)

![Captura sobre el código](../../datos/ejercicio%209/ejecucion.png)