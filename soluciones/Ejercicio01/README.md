# Solución del Ejercicio 01

## Enunciado
En esta ocasión no nos preocuparemos por los namespaces (ya llegaremos a eso) y trabajaremos en el namespace por defecto.

1. Crea un POD con las siguientes características
- Nombre "apache" 
- Imagen httpd
- Hazlo de manera IMPERATIVA.
1. Lista los POD.
1. Describe el POD.
1. Borra el POD.

## Resolución

Antes de la resolución del propio ejercicio, es importante la instalación de minikube y kubectl. Desde la propia pagina oficial de minikube podemos descargar el instalador para nuestro sistema operativo (en mi caso Windows) el cual puede aparecer como un archivo de desconocidos, no obstante, es de la pagina oficial de minikube y es seguro.

![Captura sobre el código](../../datos/ejercicio%201/instalacion.png)

Una vez instalado, podemos abrir la terminal y ejecutar el comando:

```bash
minikube start
```
![Captura sobre el código](../../datos/ejercicio%201/minikube%20start.png)

Esto nos permitirá crear un entorno de Kubernetes. Sin embargo, este comando crea el entorno por defecto ya que no hemos especificado el driver deseado. Esto en  mi caso me ha supuesto un posterior problema a la hora de querer crear el POD por lo que he decidido concretar el driver deseado para la creación del entorno. Para ello, he utilizado el siguiente comando:

```bash
minikube start --driver=docker
```
Este permite usar el driver docker para crear el entorno (debemos tener instalado docker en nuestro sistema operativo y el dashbord activo). Tras la instalación de los recursos y la creación del entorno en docker como un contenedor, podemos centrarnos en la creación del POD de forma imperativa.

![Captura sobre el código](../../datos/ejercicio%201/minikube%20start%20docker.png)

![Captura sobre el código](../../datos/ejercicio%201/docker%20container.png)

```bash
kubectl run apache --image=httpd
```
![Captura sobre el código](../../datos/ejercicio%201/creacion%20de%20%20apache.png)


Este comando crea un POD con el nombre "apache" y utiliza la imagen "httpd" haciendolo de manera imperativa, es decir, de forma directa a través de líneas de comando sin necesidad de un archivo de configuración. También podemos utilizar la misma función con el `create` en lugar de `run`. Ahora solo falta la lista de los POD, la descripción del POD y la eliminación del POD.

```bash
kubectl get pods
kubectl describe pod apache
kubectl delete pod apache
```
Todos los comandos relacionados con el POD irán precedido de `kubectl` que es la palabra clave que nos permite interactuar con el entorno de Kubernetes. En este caso, hemos utilizado el comando `get` para obtener la lista de los POD, el comando `describe` para obtener la descripción del POD y el comando `delete` para eliminar el POD. Tras estas acciones, añadiremos el término `pod o pods` y el nombre del POD para obtener la información solicitadan tanto cuando queremos describir el POD como cuando queremos eliminarlo.

Cuando ejecutamos el comando `kubectl get pods` obtenemos la siguiente información:

![Captura sobre el código](../../datos/ejercicio%201/listar%20pods.png)

Lo cual nos enseña la lista de pods que tenemos creados. En este caso, solo tenemos el POD "apache". 

Cuando ejecutamos el comando `kubectl describe pod apache` obtenemos la siguiente información:

![Captura sobre el código](../../datos/ejercicio%201/decribir%20el%20pod.png)

Se nos muestras datos generales de este POD como el nombre, la prioridad, etc. Luego se nos da los datos de los contenerdores, de las condiciones y los volumenes entre otros puntos. Y por último, se nos da los eventos del propio POD.

Cuando ejecutamos el comando `kubectl delete pod apache` obtenemos la eliminación del POD:

![Captura sobre el código](../../datos/ejercicio%201/eliminacion%20del%20pod.png)




