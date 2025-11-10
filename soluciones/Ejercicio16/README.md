# Solución del Ejercicio 16 - Configuración de NFS con NGINX y Despliegue de WordPress y MySQL con almacenamiento persistente en Kubernetes

## Enunciado

1. Realizaremos el ejercicio del vídeo de las clases 191/195.
2. Documentar todo el proceso.


## Resolución

Este ejercicio se divide en dos partes principales:

1. **Configuración de un servidor NFS en un nodo de Kubernetes y conexión a un servidor NGINX:**
   - Se configura el servidor NFS en un nodo de Kubernetes mediante ssh.
   - Se crea un PersistentVolume (PV) y un PersistentVolumeClaim (PVC).
   - Se despliega un Pod con NGINX que monta el volumen NFS para servir contenido.

2. **Configuración del servidor NFS para ser utilizado por WordPress y MySQL:**

   - Se despliegan los recursos de WordPress y MySQL utilizando PersistentVolumeClaims que apuntan al almacenamiento NFS.
   - Se documenta todo el proceso, incluyendo la configuración del clúster, nodos, y los comandos necesarios.

Comentar que para esta actividad cree un nuevo clúster con tres nodos, un maestro y dos trabajadores para poder realizar el ejercicio de forma separada. 

```bash
# Configuración y verificación de nodos
minikube start --driver=docker --nodes=3 --profile=lcm
kubectl get nodes
# Cambiar el perfil
minikube profile lcm
minikube profile
```
![Captura sobre el código](../../datos/ejercicio%2016/configuracion%20de%20nodos.png)


### Parte 1 del ejercicio: Configuración de NFS y NGINX

Esta parte del ejercicio se divide primero en la configuración del servidor NFS y luego en la configuración de NGINX para servir contenido desde el servidor NFS. Para la configuración del servidor NFS, se utiliza el comando `minikube ssh` para conectarse a un nodo de Kubernetes y configurar el servidor NFS desde dentro. Antes de comenzar con el desarrollo de esta tarea debo comentar que algunos compañeros lo realizaron con samba o incluso hostpath. Sin embargo, gracias a la ayuda de Manuel pudimos iniciar este ejercicio mediante NFS ya que citó el comando necesario. 

Una vez que estemos conectados a un nodo de Kubernetes (en mi caso será el maestro denominado lcm), podemos comenzar la configuración del servidor NFS. En mi caso, los  comandos que se utilizaron fueron:

```bash
# Actualización de paquetes y instalación de NFS
sudo apt-get update
sudo apt-get install -y nfs-kernel-server
# Creación de la carpeta de datos y configuración de exports
sudo mkdir /var/datos
sudo nano /etc/exports
# Reinicio de NFS y verificación
sudo systemctl restart nfs-kernel-server
sudo showmount -e 192.168.67.2
```
En la carpeta `/etc/exports` se debe agregar la siguiente línea para permitir el acceso a la carpeta de datos:

```bash
/var/datos *(rw,sync,no_root_squash,no_all_squash)
```
Con todos estos comandos configuramos el servidor NFS en el nodo maestro del clúster. Mostraremos una serie de capturas para ilustrar el proceso:

![Captura sobre el código](../../datos/ejercicio%2016/instalar%20nfs.png)

![Captura sobre el código](../../datos/ejercicio%2016/creacion%20de%20carpeta.png)

![Captura sobre el código](../../datos/ejercicio%2016/exports.png)

![Captura sobre el código](../../datos/ejercicio%2016/exposicion%20de%20carpeta.png)


Además, en este supuesto y siguiendo la dinámica del vídeo, he querido probar en el nodo esclavo o trabajador la conexión. 

1.- En primer lugar, tenemos que conocer la IP del nodo maestro, que en mi caso es `192.168.67.2` sacado del comando `kubectl get nodes -o wide`:

![Captura sobre el código](../../datos/ejercicio%2016/ip%20de%20nodos.png)

2.- Continuamos con la instalación del common package de NFS en el nodo esclavo interactuando dentro de este con el comando `minikube ssh --node=lcm-m03`, aunque suelen poseerlo por defecto. Usamos el comando `apt-get update` para actualizar los paquetes y luego `apt-get install -y nfs-common` para instalar el paquete NFS.

![Captura sobre el código](../../datos/ejercicio%2016/exclavo%20instalar%20comon.png)

3.- Una vez instalado el paquete, podemos probar la conexión con el comando `showmount -e 192.168.67.2`. Si nos muestra la carpeta, podemos crear la nuestra propia con `sudo mkdir /var/datos` y montar la carpeta de datos del servidor NFS con el comando `sudo mount -t nfs 192.168.67.2:/var/datos /var/datos` vinculando la carpeta local con la remota:

![Captura sobre el código](../../datos/ejercicio%2016/conexion%20esclavo.png)

4.- Hacemos alguna prueba de crear archivos en esta carpeta:

![Captura sobre el código](../../datos/ejercicio%2016/archivos%20en%20esclavo.png)


La segunda parte de este ejercicio es el despliegue de NGINX con PersistentVolumes y PersistentVolumeClaims. Para ello, primero debemos crear un archivo YAML que describa el PV y el PVC. En mi caso, los archivos que se utilizaron fueron:

El archivo que actua como volúmen persistente llamado `pv-nfs.yaml`:

```yaml
# Configuración del PersistentVolume utilizando NFS
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv  # Nombre del PersistentVolume
spec:
  capacity:
    storage: 5Gi  # Capacidad de almacenamiento del volumen
  accessModes:
    - ReadWriteMany  # Modo de acceso que permite lecturas y escrituras desde múltiples nodos
  storageClassName: volumen-nfs  # Clase de almacenamiento asociada al volumen
  nfs:  # Configuración para el almacenamiento NFS
    path: /var/datos  # Ruta en el servidor NFS donde se almacena el volumen
    server: 192.168.67.2  # Dirección IP del servidor NFS
```
Este archivo se utiliza para crear el PersistentVolume (PV) que se utilizará para almacenar los datos en el servidor NFS. El PV tiene una capacidad de almacenamiento de 5 GiB y se puede acceder de manera read-write-many desde múltiples nodos. La clase de almacenamiento asociada es `volumen-nfs` y se utiliza el servidor NFS con la dirección IP `192.168.67.2` y la ruta `/var/datos`.

El archivo que actúa como llamada al volúmen persistente llamado `pvc-nfs.yaml`:

```yaml
# Configuración del PersistentVolumeClaim asociado a NFS
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc  # Nombre del PersistentVolumeClaim
spec:
  storageClassName: volumen-nfs  # Clase de almacenamiento asociada al volumen
  accessModes:
    - ReadWriteMany  # Modo de acceso que permite lecturas y escrituras desde múltiples nodos
  resources:
    requests:
      storage: 1Gi  # Capacidad de almacenamiento solicitada para el volumen
```
Este archivo se utiliza para crear el PersistentVolumeClaim (PVC) que solicita el acceso al PV. La clase de almacenamiento asociada es `volumen-nfs` y se utiliza un acceso de lectura y escritura desde múltiples nodos. La capacidad de almacenamiento solicitada es de 1 GiB. 

Por último, creamos un archivo YAML llamado `pod-nfs.yaml` que describe el pod que se utilizará para desplegar NGINX con el acceso al volúmen NFS:

```yaml
# Configuración del Pod que utiliza un PersistentVolumeClaim con NFS
apiVersion: v1
kind: Pod
metadata:
  name: pod-nfs  # Nombre del pod
spec:
  containers:
  - name: nginx  # Nombre del contenedor
    image: nginx  # Imagen Docker utilizada
    volumeMounts:  # Montaje de volúmenes en el contenedor
      - mountPath: /usr/share/nginx/html  # Ruta dentro del contenedor donde se monta el volumen
        name: nfs-vol  # Nombre del volumen
  volumes:
    - name: nfs-vol  # Nombre del volumen
      persistentVolumeClaim:
        claimName: nfs-pvc  # PersistentVolumeClaim utilizado por el volumen
```
Este utiliza el pod llamado `pod-nfs` y utiliza el contenedor llamado `nginx` con la imagen Docker `nginx`. Los volúmenes se montan en la ruta `/usr/share/nginx/html` dentro del contenedor y se utiliza el PersistentVolumeClaim llamado `nfs-pvc` para acceder al volúmen NFS. En este punto me saltó un fallo a la hora de crear el ngnix porque la ruta escogida no existía como tal, entonces me mostraba una falta de configuración.

Ahora es aplicar los archivos YAML para crear los recursos en el clúster de Kubernetes. Para ello, podemos utilizar el comando `kubectl apply` con los archivos YAML correspondientes y podemos observarlos con el comando `kubectl get`:

```bash
kubectl apply -f pv-nfs.yaml
kubectl apply -f pvc-nfs.yaml
kubectl apply -f pod-nfs.yaml
```

![Captura sobre el código](../../datos/ejercicio%2016/pv.png)

![Captura sobre el código](../../datos/ejercicio%2016/pvc.png)


![Captura sobre el código](../../datos/ejercicio%2016/)

Esto hará que se creen los recursos de Kubernetes basados en los archivos YAML proporcionados. Para mostrar el servicio de NGINX usaremos `Kubectl port-forward`: 

```bash
kubectl port-forward pod/pod-nfs 8080:80
```
También se puede utilizar `Kubectl proxy` para acceder a los servicios de NGINX con una rutsa especificada `http://localhost:8001/api/v1/namespaces/default/pods/pod-nfs`.

Voy a mostrar la ejecución de estos comandos, aunque el segundo muestra el error que dije anteriormente por el fallo de la ruta.

![Captura sobre el código](../../datos/ejercicio%2016/fostward.png)

En esta captura se muestra el resultado de la ejecución con un index que cree en el esclavo para mostrar archivos.

![Captura sobre el código](../../datos/ejercicio%2016/ejecucion.png)



## Parte 2: Despliegue de WordPress y MySQL con almacenamiento persistente NFS

En este punto, partimos de dos problemáticas: la primera es la imcopatibilidad de NFS con archivos de tipo overlay cosa que en docker (que es donde estoy trabajando) suele crear mayoritariamente este tipo en sus contenedores, por lo que las rutas que dí inicialmente para compartir y servir como servidor web no funcionan teniendo que crearlas de nuevo en la carpeta `/var` ya que esta poseía un tipo diferente (ex04) compatible con NFS. La segunda es que a lo largo de la ejecución de los comandos, se nos agotó el número de descargas anónimas de imagenes por parte de Kubernetes (al estar todos en la misma red) y por tanto no se creaban los contenedores oportunos. Para solucionar este problema, hay varias opciones: cambiar la IP usando los datos para ir a otra red  por ejemplo  que suele ser lo más sencillo o la que decidimos algunos descargar las imagenes con docker y usarlas en el clúster. 

Como elegimos la segunda opción descargamos la imagen de WordPress y MySQL con el comando `docker pull`: 

```bash
docker pull wordpress:4.8-apache
docker pull mysql:5.6
```
Ahora solo tendríamos que traerla a nuestro clúster de Kubernetes con el comando `minikube image load`:

```bash
minikube image load wordpress:4.8-apache
minikube image load mysql:5.6
```

Otro problema en mi caso, en este punto, era la necesidad de un complemento denominado wmic, que es un complemento de Windows que permite la administración de equipos, tanto locales como remotos y es posible ejecutar cualquier tipo de tareas como obtener información, iniciar, detener, pausar procesos y servicios así como cambiar cualquier tipo de configuración en el equipo al que se tenga acceso como administrador. Tuve que adrentrarme en mi configuración y agregarlo ya que no lo tenía instalado. Una vez instalado, solo tenía que ejecutar los comandos antes descritos. 

![Captura sobre el código](../../datos/ejercicio%2016.2/wmic.png)

Una vez comentado los problemas y soluciones, realizamos el comportamiento efectuado en la primera parte del ejercicio respecto a la creación de la carpeta y añadirlas a la configuración de exports mostrando los resultados.

```bash
# Creación de la carpeta de datos y configuración de exports
sudo mkdir /var/kubernetes/datos/wordpress
sudo mkdir /var/kubernetes/datos/mysql
sudo nano /etc/exports
# Reinicio de NFS y verificación
sudo systemctl restart nfs-kernel-server
sudo showmount -e 192.168.67.2
```

![Captura sobre el código](../../datos/ejercicio%2016.2/nuevas%20carpetas%20exo4.png)

![Captura sobre el código](../../datos/ejercicio%2016.2/modificar%20exports.png)

Esta captura muestras el exports de las antiguas carpetas, pero solo sería cambiar la ruta de la carpeta y todo iría correcto, además elimine los datos usados para el caso anteriores por si existía algún tipo de imcompatibilidad. También borre datos de las carpetas creadas por si acaso hubiera creado algún elemento en ellas.

![Captura sobre el código](../../datos/ejercicio%2016.2/borrar%20archivos%20de%20carpeta.png)

Una vez configurado todo el NFS con las nuevas carpetas y sin dar ya ninguna problemática podemos pasar a los archivos correspodientes que son relativos al wordpress y mysql.

Cada uno de los apartados posee un conjunto de archivos que van desde el PersistentVolume a la PersistentVolumeClaim, el Deployment y el Service que serán usados posteriormente para la creación de los contenedores. En este caso, los archivos son los siguientes:

El encargado de PersistentVolumes de wordpress es el siguiente:

```yaml
# Configuración del PersistentVolume para WordPress utilizando NFS
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-wordpress  # Nombre del PersistentVolume
spec:
  capacity:
    storage: 25Gi  # Capacidad de almacenamiento del volumen
  accessModes:
    - ReadWriteMany  # Modo de acceso que permite lecturas y escrituras desde múltiples nodos
  storageClassName: wordpress  # Clase de almacenamiento asociada al volumen
  nfs:  # Configuración para el almacenamiento NFS
    path: /var/kubernetes/datos/wordpress  # Ruta en el servidor NFS donde se almacena el volumen
    server: 192.168.67.2  # Dirección IP del servidor NFS
```
Este archivo es el encargado de crear el volumen persistente para el wordpress, en este caso , se utiliza el almacenamiento NFS y se especifica la ruta donde se almacena el volumen. Claramente este archivo posee otro que es el encargado de la PersistentVolumeClaim.

El encargado de PersistentVolumeClaims de mysql es el siguiente:

```yaml
# Configuración del PersistentVolumeClaim para WordPress
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-wordpress  # Nombre del PersistentVolumeClaim
spec:
  storageClassName: wordpress  # Clase de almacenamiento asociada al volumen
  accessModes:
    - ReadWriteMany  # Modo de acceso que permite lecturas y escrituras desde múltiples nodos
  resources:
    requests:
      storage: 20Gi  # Capacidad de almacenamiento solicitada para el volumen
```
Este archivo es el encargado de solicitar el volumen persistente para wordpress, en este caso, se utiliza la clase de almacenamiento `wordpress` y se solicita una capacidad de almacenamiento de 20 GiB. Por último, se especifica el modo de acceso que permite lecturas y escrituras desde múltiples nodos. El último archivo asociado a este apartado es el Deployment y el Service.

El encargado de Deployments de wordpress es el siguiente:

```yaml
# Deployment para WordPress
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress  # Nombre del Deployment
  labels:
    app: wordpress  # Etiqueta de aplicación
spec:
  selector:
    matchLabels:
      app: wordpress  # Selector para emparejar pods
      tier: frontend
  strategy:
    type: Recreate  # Estrategia de despliegue para reemplazar todos los pods
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.8-apache  # Imagen de WordPress
        imagePullPolicy: Never  # Política para evitar pull de la imagen
        name: wordpress  # Nombre del contenedor
        env:  # Variables de entorno para la configuración de WordPress
        - name: WORDPRESS_DB_HOST
          value: mysql  # Nombre del host de la base de datos
        - name: WORDPRESS_DB_PASSWORD
          value: password  # Contraseña de la base de datos
        ports:
        - containerPort: 80  # Puerto en el contenedor
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage  # Montaje del volumen
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage  # Volumen persistente
        persistentVolumeClaim:
          claimName: pvc-wordpress  # Referencia al PVC
---
# Service para WordPress
apiVersion: v1
kind: Service
metadata:
  name: wordpress  # Nombre del Service
  labels:
    app: wordpress
spec:
  ports:
    - port: 80  # Puerto de servicio expuesto
  selector:
    app: wordpress  # Selector para encontrar los pods de WordPress
    tier: frontend
  type: NodePort  # Tipo de servicio para acceso externo
```
Este archivo parte de la base de un Deployment y un service para WordPress. El Deployment utiliza una imagen de WordPress y la configura para utilizar una base de datos MySQL. El Service expone el servicio de WordPress para el acceso externo. El Deployment utiliza un PersistentVolume para almacenar los datos de WordPress, mientras que el Service expone el servicio de WordPress para el acceso externo. Cabe comentar que al descargar las imágenes en local debemos añadir en la sección de imagen del contenedor la etiqueta `imagePullPolicy: Never ` para evitar el pull de la imagen ya que se situa en local.

En el caso de mysql es similar empezamos con el PersistentVolume, el PersistentVolumeClaim y luego el Deployment y el Service. 

El archivo de configuración para MySQL es el siguiente respecto PV:

```yaml
# Configuración del PersistentVolume para MySQL utilizando NFS
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-mysql  # Nombre del PersistentVolume
spec:
  capacity:
    storage: 25Gi  # Capacidad de almacenamiento del volumen
  accessModes:
    - ReadWriteMany  # Modo de acceso que permite lecturas y escrituras desde múltiples nodos
  storageClassName: mysql  # Clase de almacenamiento asociada al volumen
  nfs:  # Configuración para el almacenamiento NFS
    path: /var/kubernetes/datos/mysql  # Ruta en el servidor NFS donde se almacena el volumen
    server: 192.168.67.2  # Dirección IP del servidor NFS
```
Al igual que en el caso de WordPress, el archivo de configuración para MySQL es similar, pero con la diferencia de que se utiliza la clase de almacenamiento `mysql` y se solicita una capacidad de almacenamiento de 25 GiB.

El archivo de configuración para MySQL es el siguiente respecto PVC:

```yaml
# Configuración del PersistentVolumeClaim para MySQL
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-mysql  # Nombre del PersistentVolumeClaim
spec:
  storageClassName: mysql  # Clase de almacenamiento asociada al volumen
  accessModes:
    - ReadWriteMany  # Modo de acceso que permite lecturas y escrituras desde múltiples nodos
  resources:
    requests:
      storage: 20Gi  # Capacidad de almacenamiento solicitada para el volumen
```
Este archivo de configuración define un PersistentVolumeClaim llamado `pvc-mysql` que solicita una capacidad de almacenamiento de 20 GiB y utiliza la clase de almacenamiento `mysql`. El modo de acceso es `ReadWriteMany`, lo que permite lecturas y escrituras desde múltiples nodos.

El archivo de configuración para MySQL es el siguiente respecto Deployment y Service:

```yaml
# Deployment para MySQL
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql  # Nombre del Deployment
  labels:
    app: wordpress  # Etiqueta de aplicación para WordPress
spec:
  selector:
    matchLabels:
      app: wordpress  # Selector para emparejar pods
      tier: mysql
  strategy:
    type: Recreate  # Estrategia de despliegue para reemplazar todos los pods
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6  # Imagen de MySQL
        imagePullPolicy: Never  # Política para evitar pull de la imagen
        name: mysql  # Nombre del contenedor
        env:  # Variables de entorno para la configuración de MySQL
        - name: MYSQL_ROOT_PASSWORD
          value: password  # Contraseña del usuario root de MySQL
        ports:
        - containerPort: 3306  # Puerto en el contenedor
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage  # Montaje del volumen
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage  # Volumen persistente
        persistentVolumeClaim:
          claimName: pvc-mysql  # Referencia al PVC
---
# Service para MySQL
apiVersion: v1
kind: Service
metadata:
  name: mysql  # Nombre del Service
  labels:
    app: wordpress  # Etiqueta de aplicación para WordPress
spec:
  ports:
    - port: 3306  # Puerto de servicio expuesto para MySQL
  selector:
    app: wordpress  # Selector para encontrar los pods de MySQL
    tier: mysql
  clusterIP: None  # IP de clúster fija para garantizar el acceso directo
```
En este archivo de configuración, se define un Deployment para MySQL con la imagen `mysql:5.6` y una configuración de PersistentVolume para almacenar los datos de MySQL. Adicionalmente, se define un Service para MySQL que expone el puerto 3306 para el acceso externo. La estrategia de despliegue es de tipo `Recreate`, lo que significa que se reemplazarán todos los pods existentes al actualizar el Deployment. La política de pull de la imagen es `Never`, lo que evita el pull de la imagen ya que se situa en local.

Una vez creado todos los archivos de configuración, pasamos a aplicarlos en tu clúster de Kubernetes mediante el comando `kubectl apply -f <archivo.yaml>`.

```bash
# Aplicar Persistent Volumes y Persistent Volume Claims para WordPress y MySQL
kubectl apply -f pv-wordpress.yaml
kubectl apply -f pvc-wordpress.yaml
kubectl apply -f pv-mysql.yaml
kubectl apply -f pvc-mysql.yaml

# Desplegar WordPress y MySQL
kubectl apply -f wordpress-deployment.yaml
kubectl apply -f mysql-deployment.yaml

# Verificar pods, servicios, pvs y pvcs
kubectl get pods
kubectl get svc
```
![Captura sobre el código](../../datos/ejercicio%2016.2/pv%20de%20ambos.png)

![Captura sobre el código](../../datos/ejercicio%2016.2/pvc%20de%20ambos.png)

![Captura sobre el código](../../datos/ejercicio%2016.2/deployment%20y%20sevice.png)

![Captura sobre el código](../../datos/ejercicio%2016.2/service%202.png)

Comprobamos que todo esta correcto y se observan las uniones de los pv y pvc, terminando de ejecutar todo para poder conectarnos con un port-forward y con un minikube service. Se nos mostrará lo siguiente:

![Captura sobre el código](../../datos/ejercicio%2016.2/wordpress.png)

![Captura sobre el código](../../datos/ejercicio%2016.2/crear%20usuario.png)

![Captura sobre el código](../../datos/ejercicio%2016.2/wordpress%20final.png)

![Captura sobre el código](../../datos/ejercicio%2016.2/ls%20archivos.png)

Como se observa, he configurado algo del wordpress y he mostrado los datos de las carpetas donde van los archivos de ambos servicios. 

En este punto, junto con algunos compañeros nos dimos cuenta de que al tirar los deploy y volver a levantarlo no se guardaban los datos de la base de datos, consiguiendo uno de ellos (Javier García) la persistencia de datos gracias al uso de StatefulSet ya que enumera los pods en vez de aleatorizarlos por hash

Por último, se nos pide escalar el servicio de wordpress siguiendo lo descrito en el vídeo por lo que usamos los siguientes comandos:

```bash
# Escalar WordPress
kubectl scale --replicas=2 deploy/wordpress

# Verificar pods y servicios
kubectl get pods -o wide
kubectl get svc
```

![Captura sobre el código](../../datos/ejercicio%2016.2/escalado.png)


## Notas adicionales

Captura de las antiguas carpetas

![Captura sobre el código](../../datos/ejercicio%2016.2/creacion%20de%20carpetas.png)

Cabe decir, que me instalé samba y lo configuré por si acaso lo necesitaba, pero no lo utilicé en este caso.

![Captura sobre el código](../../datos/ejercicio%2016/samba.png)


