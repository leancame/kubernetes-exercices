## Ejercicios de Kubernetes para iniciarse en el Mundo de DevOps

¡Bienvenido/a a los ejercicios básicos de Kubernetes para iniciarse en el mundo de DevOps!

Este repositorio contiene una serie de ejercicios diseñados para ayudarte a familiarizarte con los conceptos básicos de Kubernetes. 
Los ejercicios están diseñados para ser completados en un entorno local o en un clúster de Kubernetes, y cubren una variedad de temas, desde la creación de pods y servicios hasta la implementación de aplicaciones en Kubernetes.

## Objetivos

El propósito principal de estos ejercicios es proporcionarte una introducción práctica a los conceptos clave de Kubernetes que son esenciales para cualquier persona interesada en trabajar en el área de DevOps. Al completar estos ejercicios, esperamos que adquieras experiencia práctica con:

- Creación de pods y servicios en Kubernetes.
- Implementación de aplicaciones en Kubernetes.
- Escalado y actualización de aplicaciones en Kubernetes.
- Configuración de almacenamiento persistente en Kubernetes.
- Implementación de Ingress
- Configuración de recursos y límites en Kubernetes.
- Monitoreo y resolución de problemas en Kubernetes.

## Estructura del Repositorio

Este repositorio está organizado de la siguiente manera:


- El directorio `enunciados/` contiene los ejercicios propuestos en este repositorio. Cada archivo proporciona una descripción detallada de los objetivos del ejercicio, así como instrucciones paso a paso sobre lo que se espera que completes.
- El archivo `ASSIGMENT.md` proporciona instrucciones sobre cómo enviar tus soluciones una vez que completes los ejercicios.
- El directorio `soluciones/` será el lugar donde almacenar las soluciones a los ejercicios.
- El directorio `auxiliar/` (opcional) contiene los archivos necesarios para completar los ejercicios. Puedes incluir scripts, archivos de configuración, archivos de texto para edición, etc.
- El directorio `datos/` (opcional) contiene conjuntos de datos u otros archivos de entrada que pueden ser necesarios para los ejercicios.
- El archivo `CONTRIBUTING.md` proporciona instrucciones sobre cómo contribuir al repositorio.

## Estructura del directorio `soluciones/`
Las soluciones a los ejercicios deben respetar la siguiente estructura *mínima* para lograr una correcta entrega de todos los elementos y facilitar las correcciones a los tutores. Los ejercicios que no respeten esta estructura *mínima*, serán considerados como `no entregados`. Si considera que debe agregar algún fichero o directorio a la estructura, puede hacerlo, siempre que la original esté presente.
```
Ejercicio01
    └── README_ej01.md
Ejercicio02
    |── README_ej02.md
    └── <pod>.yaml
Ejercicio03
    ├── README_ej03.md
    └── <pod>.yaml
Ejercicio04
    └── README_ej04.md
Ejercicio05
    ├── README_ej05.md
    └── <deployment>.yaml
Ejercicio06
    └── README_ej06.md
Ejercicio07
    ├── README_ej07.md
    └── <directorio_con_manifiestos_del_video>
Ejercicio08
    ├── README_ej08.md
    ├── <deployment>.yaml
    └── <service>.yaml
Ejercicio09
    ├── README_ej09.md
    └── <directorio_con_manifiestos_del_video>
Ejercicio10
    ├── README_ej10.md
    ├── <deployment>.yaml
    └── <service>.yaml
Ejercicio11
    ├── README_ej11.md
    ├── <deployment>.yaml
    └── <deployment_new_tag>.yaml
Ejercicio12
    ├── README_ej12.md
    ├── <deployment>.yaml
    └── <deployment_new_tag>.yaml
Ejercicio13
    ├── README_ej13.md
    ├── <deployment>.yaml
    ├── <service>.yaml
    └── <configmap>.yaml
Ejercicio14
    |── README_ej14.md
    |── <pod_affinity>.yaml
    └── <pod_toleration>.yaml
Ejercicio15
    ├── README_ej15.md
    ├── <deployment>.yaml
    └── <service>.yaml
Ejercicio16
    ├── README_ej16.md
    └── <directorio_con_manifiestos_del_video>
Ejercicio17
    ├── README_ej17.md
    └── <deployment>.yaml
```

## Contribución

¡Tus contribuciones son bienvenidas! Si tienes ideas para nuevos ejercicios o mejoras para los existentes, no dudes en abrir un issue o abrir un pull request.
