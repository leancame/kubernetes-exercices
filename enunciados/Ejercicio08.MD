### Ejercicio 08

#### Objetivos
Familiarizarse con los SERVICES y su funcionamiento en Kubernetes.

#### Pre-requisitos
- Haber finalizado la sección 9 del curso.
- Tener instalado y en funcionamiento **minikube**.
- Tener instalado **kubectl**.

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

#### Pista
Esta imagen es de mi repositorio de Docker Hub.

#### Entregables
Documentación del proceso (con capturas de pantalla que muestren el comportamiento al recargar el navegador).

METERSE EN EL TAG DE DOCKER