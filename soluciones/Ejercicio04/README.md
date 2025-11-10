# Solución del Ejercicio 04

## Enunciado
En esta ocasión no nos preocuparemos por los namespaces (ya llegaremos a eso) y trabajaremos en el namespace por defecto. Si no se piden otros, usaremos solo los elementos estricatemente necesarios.

1. Crea un objeto DEPLOYMENT con las siguientes caracteristicas:
- Nombre: "demogato" 
- Imagen: tomcat
- Créalo de manera IMPERATIVA.
1. Lista los POD.
1. Lista los DEPLOYMENT.
1. Borra el DEPLOYMENT.
1. Lista los POD.


## Resolución

Una vez visto ejercicios referentes a un Pod, ahora vamos a trabajar con Deployments. Los Deployment son objetos que nos permiten crear, actualizar y mantener aplicaciones en Kubernetes. Estos objetos son el responsable de la replicación de los Pods y de la actualización de las aplicaciones. Para nuestro caso concreto, vamos a crear un Deployment con el nombre "demogato" y la imagen "tomcat" de forma imperativa, es decir, a través de comandos por consola.

```bash
kubectl create deployment demogato --image=tomcat
```

![Captura sobre el código](../../datos/ejercicio%204/crear%20deployment.png)

Podemos ver que se ha creado un Deployment con el nombre "demogato" y con la imagen "tomcat". Luego de la creación de este, pasamos a realizar el resto de comando que también han sido solicitados en el enunciado y que hemos realizado en ejercicios anteriores.

Primero, listamos los Pods con el comando `kubectl get pods`, el cual nos muestra la lista de Pods existentes en nuestro clúster. En segundo lugar, listamos los Deployments con el comando `kubectl get deployments`, el cual nos muestra la lista de Deployments existentes en nuestro clúster. Por ultimo, borramos el Deployment con el comando `kubectl delete deployment demogato` y luego listamos los Pods con el comando `kubectl get pods` para ver que el Deployment ha sido eliminado y que los Pods también han sido eliminados.

![Captura sobre el código](../../datos/ejercicio%204/mix%20de%20todo%20lo%20solicitado.png)
