# Solución del Ejercicio 10

## Enunciado

1. Crea un namespace con el nombre que prefieras, de manera IMPERATIVA.
2. Aplica limitaciones de recursos a este namespace, de manera DECLARATIVA, para que no pueda usar más de 1 CPU y 1GB de memoria y como mínimo 0.5 CPU y 0.5GB de memoria.
3. Despliega los elementos del ejercicio 8 en este namespace de manera DECLARATIVA (debes modificar sus manifiestos).
4. Lista todos los elementos del namespace para mostrar el resultado.
5. Haz lo necesario para que, sin borrar el deployment, no quede ningún POD levantado.
6. Lista todos los elementos del namespace para mostrar el resultado.

---

## Resolución

### Paso 1: Crear el namespace

Creamos el namespace de forma imperativa con el siguiente comando:

```bash
kubectl create namespace lcarbajo
```
![Captura sobre el código](../../datos/ejercicio%2010/crear%20el%20namespace.png)

### Paso 2: Aplicar limitaciones de recursos

Aplicamos las limitaciones de recursos de forma declarativa con el manifiesto `limitaciones.yaml` tal y como se nos solicita en el enunciado con el límite en la etiqueta default y con los recursos mínimos de defaultRequest (https://github.com/kubernetes/design-proposals-archive/blob/main/resource-management/admission_control_limit_range.md): 

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: resource-limits  # Nombre de la configuración de límites
  namespace: lcarbajo  # Namespace al que se aplica esta configuración
spec:
  limits:
  - default:
      cpu: "1"  # Límite máximo de CPU para los contenedores (1 vCPU)
      memory: "1Gi"  # Límite máximo de memoria para los contenedores (1 GiB)
    defaultRequest:
      cpu: "500m"  # Solicitud predeterminada de CPU (500 milicores)
      memory: "512Mi"  # Solicitud predeterminada de memoria (512 MiB)
    type: Container  # Tipo de recurso al que se aplican los límites (contenedores)
```

Aplicamos el manifiesto con:

```bash
kubectl apply -f limitaciones.yaml
```
![Captura sobre el código](../../datos/ejercicio%2010/limites.png)

### Paso 3: Desplegar los recursos del ejercicio 8

Modificamos los manifiestos del ejercicio 8 para que se desplieguen en el namespace `lcarbajo`. El manifiesto `deploy_color.yaml` contiene el Deployment y el Service:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: colors  # Nombre del deployment
  namespace: lcarbajo  # Namespace al que pertenece el deployment
spec:
  replicas: 3  # Número de réplicas de pods que se mantendrán
  selector:
    matchLabels:
      app: colors  # Selector para identificar los pods gestionados por este deployment
  template:
    metadata:
      labels:
        app: colors  # Etiqueta asignada a los pods para su identificación
    spec:
      containers:
      - name: coloread  # Nombre del contenedor
        image: noloknolo/colors:v1  # Imagen Docker utilizada para el contenedor
        ports:
        - containerPort: 8080  # Puerto expuesto por el contenedor
---

# Configuración del Service para la aplicación "colors" en el namespace "lcarbajo"
apiVersion: v1
kind: Service
metadata:
  name: colors-service  # Nombre del servicio
  namespace: lcarbajo  # Namespace al que pertenece el servicio
spec:
  type: ClusterIP  # Tipo de servicio (solo accesible dentro del clúster)
  ports:
  - port: 8080  # Puerto del servicio expuesto
    protocol: TCP  # Protocolo utilizado (TCP en este caso)
    targetPort: 8080  # Puerto objetivo dentro del contenedor
  selector:
    app: colors  # Selector para asociar el servicio con los pods correspondientes
```

Aplicamos el manifiesto con:

```bash
kubectl apply -f deploy_color.yaml
```
![Captura sobre el código](../../datos/ejercicio%2010/creamos%20el%20apply%20del%20color.png)

### Paso 4: Listar todos los elementos del namespace

Para verificar que los recursos están desplegados correctamente, listamos todos los elementos en el namespace:

```bash
kubectl get all -n lcarbajo
```
![Captura sobre el código](../../datos/ejercicio%2010/desplegar%20todos%20los%20recursos.png)

### Paso 5: Escalar el deployment a cero pods sin eliminarlo

Para que no quede ningún pod levantado sin borrar el deployment, podemos escalar el deployment a cero réplicas y mostrar los resultados:

```bash
kubectl scale deployment colors --replicas=0 -n lcarbajo
kubectl get pods -n lcarbajo
kubectl get deployment colors -n lcarbajo
```

![Captura sobre el código](../../datos/ejercicio%2010/punto%205.png)

### Alternativa: Pausar el deployment y eliminar pods manualmente

Si prefieres, puedes pausar el deployment y eliminar los pods manualmente, no se realizó capturas con esta, solo se muestra como alternativa:

```bash
kubectl rollout pause deployment colors -n lcarbajo
kubectl delete pods -l app=colors -n lcarbajo
kubectl get pods -n lcarbajo
kubectl get deployment colors -n lcarbajo
kubectl rollout resume deployment colors -n lcarbajo
```

### Paso 6: Verificar el estado final

Finalmente, listamos nuevamente todos los elementos para confirmar que el deployment sigue existiendo pero no hay pods en ejecución:

```bash
kubectl get all -n lcarbajo
```
![Captura sobre el código](../../datos/ejercicio%2010/punto%206.png)
