# Solución del Ejercicio 15 - Autoescalado en Kubernetes

## Enunciado

1. Realizaremos el ejercicio del vídeo de la clase 178. En nuestro caso lo realizaremos con minikube.
2. Documentar todo el proceso.


## Resolución

Se nos pide autoescalar a través de un recurso de Kubernetes llamado Horizontal Pod Autoscaler (HPA). Para ello, debemos crear un Deployment y un Service para el contenedor Apache. Para ello, utilizaremos los archivos `deploy_autoescalada.yaml` y `service.yaml`.

1.- Primero, creamos el Deployment llamado `apache-deployment` con el archivo `deploy_autoescalada.yaml`:

```yaml
# Configuración del Deployment para Apache con soporte para autoescalado
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apache-autoescalada  # Nombre del deployment
spec:
  selector:  
    matchLabels:
      run: apache  # Selector para identificar los pods gestionados por este deployment
  replicas: 1  # Número inicial de réplicas de pods
  template:   
    metadata:
      labels:
        run: apache  # Etiqueta asignada a los pods para su identificación
    spec:
      containers:
      - name: apache  # Nombre del contenedor
        image: registry.k8s.io/hpa-example  # Imagen Docker utilizada
        ports:
        - containerPort: 80  # Puerto expuesto por el contenedor
        resources:
          limits:
              cpu: 500m  # Límite máximo de CPU asignado al contenedor
          requests:
              cpu: 200m  # Solicitud inicial de CPU para el contenedor
```
Se le añade una serie de límites a la CPU del contenedor para que el autoescalado funcione correctamente. El límite máximo de CPU es de 500 milisegundos y la solicitud inicial es de 200 milisegundos.

2.- Luego, creamos el Service llamado `apache-service` con el archivo `service.yaml`:

```yaml
# Configuración del Service para Apache
apiVersion: v1
kind: Service
metadata:
  name: apache  # Nombre del servicio
  labels: 
    run: apache  # Etiqueta asociada al servicio
spec:
  ports: 
  - port: 80  # Puerto expuesto por el servicio
  selector:
    run: apache  # Selector para asociar el servicio con los pods correspondientes
```
Respecto al servicio es un selector que se utiliza para asociar el servicio con los pods gestionados por el deployment con un derterminado puerto.

Tras la creación de ambos recursos, debemos aplicar los recursos:

```bash
kubectl apply -f deploy_autoescalada.yaml
kubectl apply -f service.yaml
```
![Captura sobre el código](../../datos/ejercicio%2015/creacion%20deploy%20y%20service.png)

Aunque hayamos creado los archivos a la hora que usar el comando correspondiente no nos muestra el resultado porque debemos instalar el Metrics Server necesario para el autoescalado. Se puede hacer de dos formas posibles, siendo la primera una instalacion del mismo y modificarlo añadiendo los comandos de descarga y aplicación de los recursos y la segunda es hacerlo directamente descargando los recursos de GitHub, modificarlo y aplicarlo. En este caso, se opta por la segunda opción. Para ello, se descarga el archivo `components.yaml` de GitHub con el siguiente comando:

```bash
curl -LO https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl apply -f components.yaml
kubectl get pods -n kube-system
```

El primero nos descargar el archivo correspondiente, el segundo nos aplica el archivo y el tercero nos muestra los pods de la namespace `kube-system` donde se encuentra el Metrics Server.

![Captura sobre el código](../../datos/ejercicio%2015/comnados%20de%20descarga.png)

Dentro del archivo `components.yaml` debemos añadir varias líneas para que el Metrics Server se pueda utilizar con el autoescalado. Incluimos `kubelet-insecure-tls` y `kubelet-preferred-address-types=InternalIP`. El primero desactiva la validación de TLS entre el Metrics Server y los kubelets, permitiendo que el Metrics Server recopile métricas aunque los certificados no sean confiables. El segundo indica que se utilicen las direcciones internas de los kubelets en lugar de las de los nodos externos.

![Captura sobre el código](../../datos/ejercicio%2015/metrics.png)

A continuación, debemos aplicar el comando `Kubectl autoescale deployment apache-autoescalada --cpu-percent=40 --min=1 --max=8` para que el autoescalado funcione. En la primera parte se indica el Deployment que se desea escalar, luego se indica el porcentaje de CPU que se desea mantener en todos los pods y las opciones de escalamiento. Posteriormente, se indica el comando `kubectl get hpa` para ver el estado del Horizontal Pod Autoscaler (HPA). Como he dicho en la captura que aporto como no había instalado el Metrics Server, el HPA no muestra los porcentajes ni funcionamiento ninguno.

![Captura sobre el código](../../datos/ejercicio%2015/autoescalado.png)

Al instalarlo, pasamos a ejecutar el comando que nos aporta el vídeo para forzar el aumento de CPU y que el autoescalado se active. Para ello, se ejecuta los siguientes comandos:

```bash
kubectl run carga --rm -it --image=busybox:1.28 --restart=Never
while sleep 0.01; do wget -q -O- http://apache; done
```

Si se necesita entrar al pod de carga:

```bash
kubectl exec -it carga -- /bin/sh
```
Cabe decir que el comando se ha modificado para que funcione en Windows ya que esteba creado para Linux. Para visualizar el estado del Horizontal Pod Autoscaler (HPA) en tiempo real <usé el PowerShell con el siguiente comando:

```bash
while ($true) { cls; kubectl get hpa; Start-Sleep -Seconds 2 }
```

![Captura sobre el código](../../datos/ejercicio%2015/ejecucion.png)

![Captura sobre el código](../../datos/ejercicio%2015/ejecucion%202.png)