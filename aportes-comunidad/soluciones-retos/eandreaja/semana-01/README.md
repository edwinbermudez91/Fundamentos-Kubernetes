# Retos rotos de la semana 01: arquitectura de Kubernetes y Pods

Mis soluciones de los 4 retos de la semana 01 (@eandreaja). Los probé en un clúster local de kind (Kubernetes v1.37.0, un solo nodo) y comprobé cada arreglo con `kubectl`.

## Resumen

| Reto | Síntoma | Causa | Arreglo |
| :--- | :--- | :--- | :--- |
| 01 | `Invalid value: 80809` y después `ErrImagePull` | Puerto con un 0 de más e imagen con una etiqueta que no existe | `containerPort: 80` e `image: nginx:alpine` |
| 02 | `CrashLoopBackOff` | El comando usa `echho` en vez de `echo` | Corregir el comando |
| 03 | `Pending` | El Pod pide 64 CPU y 500Gi, más de lo que tiene el nodo | `cpu: "500m"` y `memory: "500Mi"` |
| 04 | Error al hacer `apply` (en dos pasos) | `apiVersion: v2`, `imaeg` y `port` mal escritos | `v1`, `image` y `containerPort` |

## Reto 01: el Pod que no encuentra su ropa

**Síntoma.** Al aplicar el archivo, el clúster lo rechazó:

`The Pod "reto-01-imagen" is invalid: spec.containers[0].ports[0].containerPort: Invalid value: 80809: must be between 1 and 65535, inclusive`

**Cómo lo investigué.**
1. El mensaje trae la ruta exacta del campo (`containerPort`) y la regla (entre 1 y 65535). Con `kubectl get pod reto-01-imagen` comprobé que el Pod ni siquiera se había creado (`NotFound`).
2. Corregí el puerto a `80`, que es el puerto de Nginx dentro del contenedor (el `8080` de `-p 8080:80` es el lado de mi PC y no va en `containerPort`).
3. Al aplicar de nuevo, el Pod se creó pero quedó en `ErrImagePull` / `ImagePullBackOff`. Con `kubectl describe pod reto-01-imagen`, en `Events`: `Failed to pull image "nginx:version-fantasma-999" ... not found`.

**Causa.** Dos errores en capas distintas: un puerto fuera de rango (lo rechaza el API) y una etiqueta de imagen inexistente (falla al descargar).

**Arreglo.** `containerPort: 80` e `image: nginx:alpine`. `kubectl apply` dijo `configured` (la imagen sí se puede cambiar en un Pod que ya existe) y el Pod pasó a `Running`.

## Reto 02: el Pod que nace y muere

**Síntoma.** `CrashLoopBackOff` con reinicios cada vez más espaciados (`RESTARTS 1 (5s ago)`).

**Cómo lo investigué.** `kubectl describe` no basta porque Kubernetes sí arrancó el contenedor; lo que falló fue el programa de adentro. Con `kubectl logs reto-02-crash`: `sh: line 0: echho: not found`.

**Causa.** El comando es `sh -c "echho ... && sleep 3600"`. Como `echho` no existe, el `&&` impide que corra `sleep`, la shell termina, el contenedor se apaga y Kubernetes lo reinicia una y otra vez.

**Arreglo.** Escribir `echo`. Aplicar el cambio sobre el Pod existente dio `Forbidden`: en un Pod que ya existe solo se pueden cambiar `image`, `activeDeadlineSeconds`, `tolerations` y `terminationGracePeriodSeconds`. Tuve que borrar el Pod y crearlo de nuevo; quedó en `Running` y los logs muestran `¡Hola Cloud Native Medellín!`.

## Reto 03: el Pod demasiado exigente

**Síntoma.** El Pod se crea pero queda en `Pending`.

**Cómo lo investigué.** `kubectl describe pod reto-03-exigente`, en `Events`: `FailedScheduling: 0/1 nodes are available: 1 Insufficient cpu, 1 Insufficient memory`. Comparé lo que pedía el Pod con lo que tiene mi nodo: 2 CPU y unos 3,8 Gi repartibles, de los cuales ya había reservados 1050m de CPU y 450 Mi de memoria.

**Causa.** Los `requests` (64 CPU y 500Gi) superan por mucho la capacidad del nodo. Un request es una reserva: el despachador no asigna el Pod a un nodo donde no cabe.

**Arreglo.** `memory: "500Mi"` y `cpu: "500m"`, que caben de sobra y alcanzan para un `nginx:alpine`. Igual que en el reto 02, `apply` sobre el Pod existente dio `Forbidden` (`resources` no se puede cambiar), así que lo borré y lo recreé: `Running`.

## Reto 04: el YAML que el API Server no deja entrar

**Síntoma.** `kubectl apply` rechaza el archivo, y los errores aparecen por capas.

**Cómo lo investigué.**
1. Primero: `no matches for kind "Pod" in version "v2"`. kubectl tiene que saber qué recurso es antes de revisar los campos. Corregí `apiVersion: v1`.
2. Después: `strict decoding error: unknown field "spec.containers[0].imaeg", unknown field "spec.containers[0].ports[0].port"`. La revisión estricta junta todos los campos desconocidos en un solo mensaje.

**Causa.** Tres errores de escritura: `apiVersion: v2`, `imaeg` y `port`.

**Arreglo.** `apiVersion: v1`, `image` y `containerPort`. Con los tres corregidos, `apply` dijo `created` y el Pod quedó en `Running`.

## Lo que me llevo: los errores vienen en capas

| Quién revisa | Qué revisa | Dónde se ve el error |
| :--- | :--- | :--- |
| kubectl y el API Server | Que el YAML sea válido (versión, tipo, campos, rangos) | En la salida de `apply` |
| El scheduler | Que haya un nodo con capacidad | `describe`, sección `Events` (`FailedScheduling`) |
| El kubelet | Que se pueda descargar la imagen y arrancar | `describe`, sección `Events` (`ErrImagePull`) |
| El contenedor | Que el programa de adentro funcione | `logs` |
