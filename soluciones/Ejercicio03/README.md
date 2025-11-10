# Solución del Ejercicio 03

## Enunciado
En esta ocasión no nos preocuparemos por los namespaces (ya llegaremos a eso) y trabajaremos en el namespace por defecto. Si no se piden otros, usaremos solo los elementos estricatemente necesarios.

1. Crea un manifiesto de un POD con estas caracteristicas:
  - Nombre "apache-yaml"
  - Imagen *httpd* 
  - Incluye en este manifiesto los elementos necesarios para que cuente con una pólítica de reincio, para que *siempre* se reincie.
  - Despliégalo de manera DECLARATIVA.
1. Lista los POD mostrando toda la información posible.
1. Conecta el puerto del POD a un puerto de tu equipo para poder acceder a él.
1. Borrar el POD 


# Resolución

La creación de un Pod que cumpla con los requerimientos solicitado y que sea de manera declarativa deberá tener la siguiente estructura:

```yaml
# Especifica la versión de la API que se usará para manejar este recurso
apiVersion: v1 
# Define el tipo de recurso, en este caso, un Pod
kind: Pod 
# Metadatos del Pod, como su nombre único dentro del namespace
metadata:
  name: apache  # Nombre del Pod
# Especificación del Pod
spec:
  restartPolicy: Always  # Política de reinicio: siempre reiniciar el contenedor si falla
  containers:
    - name: apache-yaml  # Nombre del contenedor dentro del Pod
      image: httpd  # Imagen Docker para este contenedor (servidor Apache HTTP)
      ports:
      - containerPort: 80  # Puerto expuesto por el contenedor
```
Un punto a tener en cuenta en este archivo yaml es la especificación de la política de reinicio. En este caso, se ha especificado que el contenedor siempre se reinicie si falla con la cláusula `restartPolicy: Always`. Esto garantiza que el contenedor siempre esté disponible y funcionando incluso si falla. Además se ha añadido un puerto al contenedor para que pueda ser accedido desde fuera del contenedor y así poder conectarse a el.

Desplegamos el `pod` con el comando `kubectl apply -f apache.yaml` en la terminal. 

![Captura sobre el código](../../datos/ejercicio%203/crear.png)

Tras el despliegue, podemos ver la información del `pod` con el comando `kubectl get pods` en la terminal.

![Captura sobre el código](../../datos/ejercicio%203/listar.png)

Para poder conectarnos al `pod` con el puerto expuesto, usaremos el comando `kubectl port-forward pod/apache 8080:80` en la terminal. Esto nos permite conectar el puerto 8080 de nuestro equipo con el puerto 80 del `pod` que hemos creado. Podemos acceder a el con el navegador web con la URL `http://localhost:8080/`. Otra manera de poder realizar este paso es con el comando `Kubectl expose pod apache --name=apache-yaml --port=80`. La diferencia radica en que el primero crea una espeice de tunel local reenviando las solicitudes de nuestra máquina local al Pod escuchando por el puerto 8080 y pasando estos datos al 80 del Pod, mientras que el segundo crea un servicio que expone el puerto 80 del Pod. Además el primero es temporal y solo se puede acceder desde la máquina local, mientras que el segundo es un servicio que se puede acceder desde otros recursos del clúster y será persistente hasta que se elimine el propio servicio. 

![Captura sobre el código](../../datos/ejercicio%203/mostrar%20el%20puerto%20y%20formatearlo.png)

Podemos borrar el `pod` con el comando `kubectl delete pod apache` en la terminal. Esto eliminará el `pod` y todos sus recursos asociados.

![Captura sobre el código](../../datos/ejercicio%203/borrar.png)



