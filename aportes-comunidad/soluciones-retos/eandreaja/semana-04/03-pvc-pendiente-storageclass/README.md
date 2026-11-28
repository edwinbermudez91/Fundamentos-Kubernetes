# Reto 03: PersistentVolumeClaim atascado en Pending

## Síntoma

El PVC `bodega-reto-03-pvc` se quedaba en `Pending` y el PV `bodega-reto-03-pv` seguía en `Available`: no se enlazaban.

## Cómo lo investigué

Miré `kubectl get pv,pvc` y los `Events` de `kubectl describe pvc bodega-reto-03-pvc`, donde aparecía `storageclass.storage.k8s.io "ssd-ultra-inexistente" not found`. Después comparé lo que pide el PVC con lo que ofrece el PV.

## Causa raíz

El PVC pedía la categoría `ssd-ultra-inexistente` y 50Gi, pero el PV es de la categoría `manual` y solo tiene 1Gi.

## Solución

En el PVC puse `storageClassName: manual` y `storage: 1Gi`. No cambié el PV. Como esos campos no se pueden modificar en un PVC ya creado, lo borré y lo creé de nuevo.

## Cómo lo comprobé

El PVC y el PV quedaron en `Bound`, el PVC con el volumen `bodega-reto-03-pv` y capacidad de 1Gi.

## Qué aprendí

Para que un PVC se enlace con un PV, la `storageClassName` debe coincidir y el tamaño pedido debe ser igual o menor que la capacidad disponible.
