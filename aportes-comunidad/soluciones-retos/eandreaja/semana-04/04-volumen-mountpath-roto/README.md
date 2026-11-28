# Reto 04: volumeMounts no coincide con volumes

## Síntoma

El API Server rechazó el Pod `pod-volumen-reto-04`: nunca llegó a crearse.

## Cómo lo investigué

Leí el mensaje de `kubectl apply -f`: `spec.containers[0].volumeMounts[0].name: Not found: "datos-kubecafe"`. Después comparé ese nombre con el declarado en `volumes`, que era `volumen-bodega`.

## Causa raíz

`volumeMounts` pedía un volumen llamado `datos-kubecafe`, y no había ningún volumen con ese nombre.

## Solución

En `volumeMounts` cambié el nombre a `volumen-bodega`.

## Cómo lo comprobé

El Pod quedó `1/1 Running`, `kubectl describe pod` mostró `/datos from volumen-bodega` y pude escribir y leer un archivo de prueba en `/datos`.

## Qué aprendí

El nombre es lo que une `volumes` (qué almacenamiento hay) con `volumeMounts` (dónde aparece dentro del contenedor), así que deben ser iguales. Un `emptyDir` es temporal: dura lo mismo que el Pod.
