# Solución del Ejercicio 06

## Enunciado
En esta ocasión, no nos preocuparemos por los namespaces (ya llegaremos a eso) y trabajaremos en el namespace por defecto. Si no se piden otros, usaremos solo los elementos estrictamente necesarios.

1. Usando el ejercicio anterior, escala la aplicación de  tres maneras diferentes, aumentando 1, 2 y 3 réplicas respectivamente.
2. Lista los POD después de cada incremento.
3. Borra el DEPLOYMENT.


## Resolución

Podemos escalar la aplicación de tres maneras diferentes aumentando 1, 2 y 3 réplicas respectivamente. En primer lugar, se puede escalar la aplicación con el propio `yaml` modificando el campo `replicas` en el bloque `spec` del manifiesto. En segundo lugar, se puede escalar la aplicación mediante el comando `kubectl scale deployment demogat-yaml --replicas=3` y en tercer lugar, se puede escalar la aplicación mediante el uso de alguna tag de la aplicación que nos permita escalarla, por ejemplo, si tenemos un tag llamado `estado` podemos usarlo para escalar la aplicación usando `kubectl scale deployment -l estado=1 --replicas=6` donde el `-l` es un selector de etiquetas que nos permite escalar la aplicación con la etiqueta `estado=1`.

Para listar los POD después de cada incremento, podemos usar el comando `kubectl get pods` que nos mostrará la lista de PODs disponibles en el clúster. Por último, para borrar el DEPLOYMENT, podemos usar el comando `kubectl delete deployment demogat-yaml`.

Hay otra forma que es muy similar a la primera, que sería editar el deployment hecho desde comando para añadir el campo réplicas y luego hacer un `kubectl apply -f deployment.yaml` para aplicar los cambios. Este comando es el `Kubectl edit deployment demogat-yaml`

![Captura sobre el código](../../datos/ejercicio%206/creacion%20y%20primera%20escalada.png)

![Captura sobre el código](../../datos/ejercicio%206/segundo%20escalado.png)

![Captura sobre el código](../../datos/ejercicio%206/borrar%20todo.png)


