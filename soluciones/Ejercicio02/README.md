# Solucion del Ejercicio 02

## Enunciado
En esta ocasión no nos preocuparemos por los namespaces (ya llegaremos a eso) y trabajaremos en el namespace por defecto. Si no se piden otros, usaremos solo los elementos estricatemente necesarios.

1. Con las siguientes características
- Nombre "apache" 
- Imagen httpd
- Despliégalo de forma DECLARATIVA.
1. Lista los POD.
1. Describe el POD.
1. Elimina el POD.


## Resolución

Este caso es similar al ejercicio anterior pero con la diferencia de que hemos utilizado el comando `create` en vez de `apply`ya que queremos crear un objeto de forma declarativa, es decir, crearemos el POD a partir de un archivo de configuración llamado `pod_apache.yaml` con el siguiente contenido:

```yaml
# Define la versión de la API de Kubernetes utilizada para este recurso.
apiVersion: v1

# Especifica el tipo de recurso de Kubernetes. En este caso, es un Pod.
kind: Pod 

# Metadatos asociados al Pod, como su nombre.
metadata:
  # Nombre del Pod.
  name: apache

# Especifica las características del Pod y sus contenedores.
spec:
  # Lista de contenedores que se ejecutarán dentro del Pod.
  containers:
    - # Nombre del contenedor dentro del Pod.
      name: apache
      # Imagen que se utilizará para el contenedor, en este caso, la imagen oficial de Apache HTTP Server.
      image: httpd
```

En este código, creamos un POD llamado `apache` con una imagen de `httpd` y un contenedor llamado `apache` dentro del POD. Para crear el POD de forma declarativa, ejecutamos el comando `kubectl apply -f pod_apache.yaml` en la terminal.

![Captura sobre el código](../../datos/ejercicio%202/crear%20pod%20declarativo.png)

Continuamos con la lista de los POD existentes en el cluster con el comando `kubectl get pods`:

![Captura sobre el código](../../datos/ejercicio%202/listar.png)

En este caso, solo tenemos el POD "apache".

Ahora vamos a describir el POD con el comando `kubectl describe pod apache`:

![Captura sobre el código](../../datos/ejercicio%202/describir%20el%20pod.png)

Por último, vamos a eliminar el POD con el comando `kubectl delete pod apache`:

![Captura sobre el código](../../datos/ejercicio%202/borrar.png)
