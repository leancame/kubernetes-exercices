# Ejercicio 14

## Objetivos 

- Configurar un cluster multinodo con Minikube.
- Etiquetar los nodos para diferenciarlos.
- Crear Pods que se asignen a nodos específicos usando node affinity.
- Aplicar taints a los nodos para controlar qué Pods pueden ser programados en ellos usando tolerations.

## Pre-requisitos
- Haber llegado hasta la clase 158 del curso
- Tener instalado y en funcionamiento minikube
- Tener instalado kubectl

## Enunciado

1. Configura el entorno:
    Inicia un cluster de Kubernetes con Minikube que tenga un nodo maestro y dos nodos de trabajo.

2. Etiqueta los nodos:
    Utiliza kubectl para asignar etiquetas a los nodos, de manera que puedas diferenciarlos y utilizarlos en reglas de afinidad de los Pods.

3. Asigna Pods a nodos específicos usando node affinity:
    Crea un Pod que solo pueda ejecutarse en uno de los nodos de trabajo utilizando la afinidad de nodos.

4. Controla la programación de Pods con taints & tolerations:
    Aplica un taint a uno de los nodos de trabajo y configura un Pod con las tolerancias adecuadas para que solo pueda ejecutarse en ese nodo.

5. Documentar todo el proceso.

## Entregables

- Documentación del proceso (con capturas de pantalla)
