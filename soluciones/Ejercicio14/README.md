# Solución del Ejercicio 14

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


## Resolución

Antes de referirnos a la materia, por error propio  debido a desconocimiento plantee un `taint` creyendo que no le había dado una clave/valor cosa que no era así y por tanto a la hora de declarar la `toleration` en el deployment no funcionaba esta porque no existia esa posibilidad. Todo este error me llevó a intentar suprimir el `taint` pero no me funciono, por lo que tuve que crear un nuevo nodo y seguir con el ejercicio usando una nueva condición clave/valor.

Cabe adherir una problemática sucedida a lo largo de este ejercicio sobre las tolerancias. Cuando aplicamos un `taint` a un nodo y configuramos un `toleration` en el deployment buscando que se ejecute en ese nodo solamente, este no tiene porque ejecutarse en ese nodo, sino que puede ejecutarse en cualquier otro nodo aunque no tenga ese taint. Esto se debe a que Kubernetes busca ejecutar el Pod al nodo más apropiado cuyo recursos sean los más cercanos a los solicitados en el deployment. Por lo tanto, para que queramos que un pod se ejecute concretamente en un nodo con una determinada tolerancia, debemos asegurarnos que también se cumplan las condiciones de afinidad de los nodos. Si otorgamos la afinidad, se ejecutará en el nodo con la tolerancia adecuada y en caso de no poseer esta tolerancia no se ejecutara en el nodo como tal. 

1.- Configuración del cluster

Inicia un cluster de Minikube con 3 nodos (1 master y 2 workers) cuyo nombre va a ser `lcarbajo` y posteriormente mostraremos los nodos y los perfiles disponibles:

```bash
minikube start --driver=docker --nodes=3 --profile=lcarbajo
minikube profile list
kubectl get nodes
```

![Captura sobre el código](../../datos/ejercicio%2014/creacion%20del%20cluster.png)

![Captura sobre el código](../../datos/ejercicio%2014/listar%20cluster.png)

![Captura sobre el código](../../datos/ejercicio%2014/ver%20nodos.png)


2.- Etiquetado de nodos

Asignamos las etiquetas a los nodos para diferenciarlos dando en este caso la etiqueta `role` con los valores `master` y `worker-1`, `worker-2` respectivamente y luego mostramos los nodos con sus etiquetas:

```bash
kubectl label node lcarbajo role=master
kubectl label node lcarbajo-m02 role=worker-1
kubectl label node lcarbajo-m03 role=worker-2
kubectl get nodes --show-labels
```

![Captura sobre el código](../../datos/ejercicio%2014/etiquetar.png)


3.- Afinidad de nodos (Node Affinity)

Cuando hablamos de afinidad es una forma de decirle al scheduler de Kubernetes en qué nodos preferimos que se ejecuten los Pods, basándonos en etiquetas (labels) que tienen los nodos. Para este caso, he creado un Pod que solo se ejecute en el nodo con etiqueta `role=worker-1` usando el siguiente manifiesto llamado `afinity-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-worker1  # Nombre del pod
spec:
  affinity:
    nodeAffinity:  # Afinidad basada en nodos
      requiredDuringSchedulingIgnoredDuringExecution:  # Requisito obligatorio para programar el pod
        nodeSelectorTerms:
        - matchExpressions:
          - key: role  # Clave que se verifica en las etiquetas del nodo
            operator: In  # El operador especifica que el valor debe coincidir
            values:
            - worker-1  # Valor requerido en la etiqueta del nodo
  containers:
  - name: nginx  # Nombre del contenedor
    image: nginx  # Imagen Docker utilizada para Nginx
```
En este Pod, se especifica que solo se puede ejecutar en un nodo con la etiqueta `role=worker-1`, por lo que el scheduler de Kubernetes no podrá programarlo en otro nodo. Además, existen etiquetas como `afinity` que se utilizan para especificar la afinidad de los Pods, en este caso, se utiliza `nodeAffinity` para especificar que el Pod solo se ejecute en un nodo con la etiqueta determinada; encontramos la etiqueta `requiredDuringSchedulingIgnoredDuringExecution` que indica que el scheduler debe programar el Pod en un nodo con la etiqueta especificada y si no se encuentra un nodo con esa etiqueta, el scheduler no programará el Pod. Por último, encontramos `nodeSelectorTerms` que es una lista de términos que se utilizan para especificar las condiciones que el scheduler debe cumplir para programar el Pod en un nodo. En este caso, se utiliza `matchExpressions` para especificar que el scheduler debe programar el Pod en un nodo con la etiqueta `role=worker-1`. 

Aplicamos el pod, verificamos y describimos el pod para saber si ha funcionado en el nodo concreto:

```bash
kubectl apply -f afinity_pod.yaml
kubectl get pods -o wide
kubectl describe pod pod-worker1
```

![Captura sobre el código](../../datos/ejercicio%2014/afinity.png)

4.- Taints y Tolerations

En este caso, realicé dos `taints` siendo el primero un fallo mío al creer que no le había dado valor a la clave al nodo `lcarbajo-m03`. El primero como he comentado es en el nodo `lcarbajo-m03` y para encontrar si se ha creado correctamente lo hago con los siguientes comandos:

```bash
kubectl taint nodes lcarbajo-m03 key=special:NoSchedule
kubectl describe node lcarbajo-m03 | findstr Taints
```
Como se observa, el nodo `lcarbajo-m03` tiene un `taint` con la clave `ley` y el valor `special`, lo que significa que en caso de querer hacer un deployment o pod que haga referencia a esta condición debe compararse el valor y la clave, cosa que no he hecho.

![Captura sobre el código](../../datos/ejercicio%2014/taint.png)

![Captura sobre el código](../../datos/ejercicio%2014/buscar%20etiquetas.png)

Crea un Pod con tolerancia para el taint `special`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-worker2  # Nombre del pod
spec:
  tolerations:  # Configuración de tolerancias
  - key: "special"  # Clave de la tolerancia
    operator: "Exists"  # Operador que permite tolerar cualquier valor con esta clave
    effect: "NoSchedule"  # Efecto que evita la programación en nodos con esta taint, salvo que exista la tolerancia
  containers:
  - name: nginx  # Nombre del contenedor
    image: nginx  # Imagen Docker utilizada para Nginx
```
Este era el modelo usado para este ejemplo, pero como se observa uso el `operator: "Exists"` para que el pod pueda ejecutarse en cualquier nodo que tenga la taint `special`. Sin embargo, yo he dado una clave llamada `key` con el valor `special` en el `taint` del nodo, por lo que este pod nunca va a funcionar. Por mi desconocimiento o error, decidí eliminar el `taint`, pero me llevó a una serie de errores en cadenas finalizando con la creación de un nuevo nodo con otro `taint`.

Aplica el manifiesto y verifica:

```bash
kubectl apply -f tolerations_pod.yaml
kubectl get pods -o wide
kubectl describe pod pod-worker2
```
Además de mi error, este pod seguramente irá a otro nodo porque como he planteado al inicio, el hecho de añadir una condición no es suficiente para que el pod se ejecute en el nodo concreto. Para que se ejecute en el nodo concreto, es necesario que el pod tenga una tolerancia y además se debe cumplir la condición de afinidad de los nodos.

![Captura sobre el código](../../datos/ejercicio%2014/fallo%20de%20taint%20.png)

Antes de continuar y entregar la solución, quiero mostrar todo el proceso de realización del nuevo nodo para la ejecución de otra condición. Como comenté, además de mi error anterior al intentar crear el nuevo nodo me confundí y cree un clúster entero por lo que tenía que eliminar este para poder usarlo como nuevo nodo.

El comando para poder crear el nuevo nodo es `minikube node add --profile lcarbajo-m04` y a este se le ha dado un `taint` con la clave `memoria` y el valor `grande` usando el comando `kubectl taint nodes lcarbajo-m04 memoria=grande:NoSchedul`. Asimismo, para poder crear el nuevo nodo con la etiqueta `role=worker-3` se ha usado el comando `kubectl label nodes lcarbajo-m04 role=worker-3`. Aquí podemos ver el trayecto que he seguido para poder crear el nuevo nodo con la problemática descrita.

![Captura sobre el código](../../datos/ejercicio%2014/datos%20raros.png)

![Captura sobre el código](../../datos/ejercicio%2014/datos%20raros%202.png)


Siguiendo el último punto y mostrando la correcta solución al problema de las tolerancias a un nodo concreto como es el nodo `lcarbajo-m04` con un pod con tolerancia para un taint concreto como es el archivo o manifisesto `tolerations_pod_2.yaml`, debemos combinar la tolerancia con la afinidad para poder ejecutar el pod en el nodo concreto que posea el taint correspondiente. Por lo tanto, el archivo `tolerations_pod_2.yaml` es el siguiente:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-worker2-conafinity  # Nombre del pod
spec:
  affinity:
    nodeAffinity:  # Afinidad basada en nodos
      requiredDuringSchedulingIgnoredDuringExecution:  # Requisito obligatorio para programar el pod
        nodeSelectorTerms:
          - matchExpressions:
              - key: role  # Clave que se verifica en las etiquetas del nodo
                operator: In  # El operador especifica que el valor debe coincidir
                values:
                  - worker-3  # Valor requerido en la etiqueta del nodo
  tolerations:  # Configuración de tolerancias
    - key: "memoria"  # Clave de la tolerancia
      operator: "Equal"  # Operador para comparar valores
      value: "grande"  # Valor que se debe tolerar
      effect: "NoSchedule"  # Efecto que permite programar el pod en nodos con la taint correspondiente
  containers:
  - name: nginx  # Nombre del contenedor
    image: nginx  # Imagen Docker utilizada para Nginx

```
Como se observa, el archivo `tolerations_pod_2.yaml` tiene una tolerancia para el taint con la clave `memoria` y el valor `grande` y un efecto de `No Schedule` para que el pod pueda programarse en nodos con la taint correspondiente. Además, el archivo `tolerations_pod_2.yaml` tiene una afinidad para programar el pod en el nodo concreto que posea la etiqueta `role=worker-3`. Por lo tanto, el pod se ejecutará en el nodo `lcarbajo-m04` que posee la etiqueta `role=worker-3` y el taint con la clave `memoria` y el valor `grande`. 

Aplicamos el manifiesto y verificamos que sea todo correcto enseñando los pods y describiendo aquel que acabamos de crear:

```bash
kubectl apply -f tolerations_pod_2.yaml
kubectl get pods -o wide
kubectl describe pod pod-worker2-conaffinity
```

![Captura sobre el código](../../datos/ejercicio%2014/tolerancia%20concreta.png)


