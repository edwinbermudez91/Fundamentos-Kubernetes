# Reto 05: ConfigMap montado como volumen con clave errónea

## Síntoma

El Pod `web-reto-05` se quedaba en `ContainerCreating` y nunca pasaba a `Running`.

## Cómo lo investigué

En los `Events` de `kubectl describe pod web-reto-05` aparecía `MountVolume.SetUp failed` con `configmap references non-existent config key: menu.html`. Después comparé con el ConfigMap `menu-html-reto-05`, que solo tiene la clave `pagina.html`.

## Causa raíz

En `items`, el volumen pedía la clave `menu.html`, que no existe en el ConfigMap.

## Solución

Cambié `key` a `pagina.html` y dejé `path: index.html`. Borré el Pod y lo creé de nuevo.

## Cómo lo comprobé

El Pod quedó `1/1 Running`, en `/usr/share/nginx/html` apareció `index.html` y una petición HTTP dentro del contenedor respondió `200 OK` con el título del menú.

## Qué aprendí

Al montar un ConfigMap como volumen, `key` indica qué clave se toma y `path` con qué nombre queda el archivo. `name` indica cuál ConfigMap usar.
