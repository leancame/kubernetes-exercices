# Solución del Ejercicio 13

## Enunciado
A partir de ahora, habrá ejercicios que no tendrán una descripción detallada. En estos casos, se espera que el estudiante pueda resolver el ejercicio a partir de los conocimientos adquiridos en el curso.

1. Crea un ConfigMap que contenga un index.html con el siguiente contenido:
```html
<html>
    <head>
    <title>Configmap</title>
    </head>
    <body>
    <h1>Mira mi index, creado con Configmap</h1>
    </body>
</html>
```
2. Crea tanto un deployment como un service, que muestre el contenido del ConfigMap desde un navegador.


## Resolución

1.- Comenzamos con el ConfigMap a través de `configmap.yaml` donde podemos describir un ConfigMap un objeto de Kubernetes que permite almacenar datos de configuración no confidenciales en forma de pares clave-valor.:

```yaml
# Configuración del ConfigMap para Nginx
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config  # Nombre del ConfigMap
data:
  index.html: |  # Archivo index.html proporcionado como parte del ConfigMap
    <html>
      <head>
        <title>Configmap</title>
      </head>
      <body>
        <h1>Mira mi index, creado con Configmap</h1>
      </body>
    </html>
``` 
En este solo configuramos un nombre de ConfigMap y el contenido del archivo index.html con la etiqueta data la cual se utiliza para proporcionar las claves/valores. 

2.- Seguimos con la configuración del deployment con `deploy_config.yaml`:
```yaml
# Configuración del Deployment para Nginx con un ConfigMap
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-config  # Nombre del deployment
spec:
  replicas: 2  # Número de réplicas de pods que se mantendrán
  selector:
    matchLabels:
      app: nginx  # Selector para identificar los pods gestionados por este deployment
  template:
    metadata:
      labels:
        app: nginx  # Etiqueta asignada a los pods para su identificación
    spec:
      containers:
        - name: nginx  # Nombre del contenedor
          image: nginx:latest  # Imagen Docker utilizada para Nginx
          ports:
            - containerPort: 80  # Puerto expuesto por el contenedor
          volumeMounts:
            - name: nginx-index  # Nombre del volumen montado
              mountPath: /usr/share/nginx/html  # Ruta donde se monta el volumen en el contenedor
      volumes:
        - name: nginx-index  # Nombre del volumen
          configMap:
            name: nginx-config  # Nombre del ConfigMap que proporciona los datos para el volumen
```

En este archivo configuramos el deployment para Nginx con un ConfigMap. Se especifica que el deployment debe tener 2 réplicas de pods, y se define un selector para identificar los pods gestionados por este deployment. La especificación del contenedor utiliza la imagen de Nginx más reciente y expone el puerto 80. Se monta un volumen llamado `nginx-index` en la ruta `/usr/share/nginx/html` del contenedor, y se proporciona el ConfigMap llamado `nginx-config` para proporcionar los datos para el volumen.

3.- Finalizamos con el servicio en el archivo `service_config.yaml`: 
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service  # Nombre del servicio
spec:
  type: NodePort  # Tipo de servicio expuesto a través de un NodePort
  selector:
    app: nginx  # Selector para asociar el servicio con los pods correspondientes
  ports:
    - protocol: TCP  # Protocolo utilizado (TCP en este caso)
      port: 80  # Puerto expuesto por el servicio
      targetPort: 80  # Puerto objetivo dentro del contenedor
```
El servicio expuesto es de tipo NodePort y se expone el puerto 80 del servicio. Se utiliza el selector `app: nginx` para asociar el servicio con los pods correspondientes. 

Tras la creación de los archivos, ejecutamos los comandos para desplegar y acceder a la aplicación a través de un tunel:

```bash
kubectl apply -f configmap.yaml
kubectl apply -f deploy_config.yaml
kubectl apply -f service_config.yaml
minikube service nginx-service
```

Pasamos a mostrar los datos con capturas de pantallas de estos comandos expuestos anteriormente descritos:

ConfigMap creado
![Captura sobre el código](../../datos/ejercicio%2013/configmap.png)

Deployment y Service creados
![Captura sobre el código](../../datos/ejercicio%2013/deploy%20y%20service.png)

Mostrar contenido servido a través de un tunel
![Captura sobre el código](../../datos/ejercicio%2013/mostrar.png)

Ejecución y acceso al servicio
![Captura sobre el código](../../datos/ejercicio%2013/ejecucion.png)



