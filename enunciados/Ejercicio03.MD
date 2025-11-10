# Ejercicio 03

## Objetivos 

- Dar los primeros pasos con la cli kubectl
- Dar los primeros pasos con los manifiestos de kubernetes
- Familiarizarse con los PODS, como objeto básico de Kubernetes

## Pre-requisitos

- Tener instalado y funcionando minikube
- Tener instalado kubectl

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



## Entregables

- Documentación del proceso (con capturas de pantalla)
- Codigo yaml utilizado
