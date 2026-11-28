# Reto 02: Secret con referencia rota

## Síntoma

El Pod `caja-reto-02` no arrancaba: quedaba en `0/1` con el estado `CreateContainerConfigError`.

## Cómo lo investigué

Con `kubectl get secrets` vi que solo existe `credenciales-kubecafe`. En la sección `Events` de `kubectl describe pod caja-reto-02` aparecía `secret "secreto-inexistente" not found`. Después comparé la referencia del Pod con lo que define el Secret: la key también estaba mal, aunque el error no la mencionaba.

## Causa raíz

`secretKeyRef` apuntaba a un Secret (`secreto-inexistente`) y a una key (`TOKEN_SECRETO`) que no existen.

## Solución

Corregí la referencia del Pod: `name: credenciales-kubecafe` y `key: API_TOKEN`. No cambié el Secret. Borré el Pod y lo creé de nuevo.

## Cómo lo comprobé

El Pod quedó `1/1 Running`, el log mostró `Token cargado` y `kubectl exec caja-reto-02 -- printenv TOKEN` devolvió el valor de ejemplo del Secret.

## Qué aprendí

Se corrige el pedido del Pod, no el Secret que ya existe. En `key` va el nombre de la casilla, no su contenido. Y base64 no es cifrado: un Secret real no se sube a GitHub.
