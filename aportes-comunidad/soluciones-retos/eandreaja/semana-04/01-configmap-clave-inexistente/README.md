# Reto 01: ConfigMap con clave inexistente

## Síntoma

El Pod `barista-reto-01` no arrancaba: quedaba en `0/1` con el estado `CreateContainerConfigError`.

## Cómo lo investigué

Revisé la sección `Events` de `kubectl describe pod barista-reto-01`, donde aparecía `couldn't find key PRECIO_ESPRESSO in ConfigMap default/menu-precios-cfg`. Después comparé con las claves del ConfigMap: solo existen `PRECIO_TINTO` y `MONEDA`.

## Causa raíz

El campo `key` de `configMapKeyRef` pedía `PRECIO_ESPRESSO`, una clave que no existe en el ConfigMap.

## Solución

Cambié la `key` a `PRECIO_TINTO`, que existe y trae un precio. Como un Pod no permite cambiar sus variables de entorno, tuve que borrarlo y crearlo de nuevo.

## Cómo lo comprobé

El Pod quedó `1/1 Running`, `kubectl logs barista-reto-01` mostró `Precio: 3500` y `kubectl exec barista-reto-01 -- printenv PRECIO_CAFE` devolvió `3500`.

## Qué aprendí

`PRECIO_CAFE` es el nombre que la variable tiene dentro del contenedor y lo decide el Pod. La `key` es distinta: tiene que existir en el ConfigMap.
