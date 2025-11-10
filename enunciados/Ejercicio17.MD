# Ejercicio 17

## Objetivo

El objetivo de este ejercicio es que los estudiantes se familiaricen con la configuración y uso de las sondas `livenessProbe` y `readinessProbe` en Kubernetes para supervisar el estado de los contenedores. A través de la implementación de estas sondas, los alumnos podrán detectar si un contenedor está vivo y si está listo para recibir tráfico, lo cual es esencial para el mantenimiento de la disponibilidad y estabilidad de las aplicaciones en un entorno de Kubernetes.

## ¡Importante!

Ten mucho cuidado con los valores sensibles que puedas exponer en tus soluciones. Recuerda que otros pueden acceder a los datos en tus entregables, lo que podría comprometer la seguridad de la información. ¡Siempre asegúrate de proteger tus datos y los de tus clientes!

## Enunciado

En este ejercicio, deberás utilizar la imagen `dboffelli/testing_probes:2.0`. Es una aplicación web hecha en GO y está diseñada para probar el comportamiento de las sondas `readinessProbe` y `livenessProbe` en Kubernetes. 

### Requisitos del despliegue:

1. **Desplegar el contenedor:**
   - Utilizar la imagen proporcionada para desplegar un Deployment. Evaluar cuánto demora en levantar el servidor. Consultar los endpoints `/` y `/healthy`.

2. **Configurar las sondas:**
   - **Liveness Probe:** Configura una `livenessProbe` para comprobar que el servidor está corriendo. A la hora de configurarla, ten en cuenta el tiempo que demora en levantar. Esta sonda debería reflejar la respuesta del endpoint `/`.
   - **Readiness Probe:** Configura una `readinessProbe` para verificar que el servidor está listo para recibir flujo de trabajo. Esta sonda debería reflejar el estado del contenedor (fallando y recuperándose según la respuesta que obtenga del endpoint `/healthy`).

3. **Pruebas de despliegue:**
   - Realizar un despliegue y observar el comportamiento del contenedor con las sondas configuradas a lo largo de un minuto.

## Entregables

- Readme los comandos ejecutados y con capturas de pantalla y comentarios sobre los resultados de las sondas. 
- Archivos de configuración de Kubernetes (`.yaml`) usados para el despliegue, con las sondas configuradas.
