# Prompts — LUGONES CARLOS - TP 2 (turnos y vacantes)

## Prompt 1 (fija los cuatro endpoints principales y la separación input/output)

Quiero que generes un archivo `openapi.yaml`, versión OpenAPI 3.1.0, para una API que describe turnos de guardia y sus vacantes.

Reglas del dominio:
- Un **turno** representa el bloque de guardia de un médico: tiene `id`, `medico_titular` (string), `fecha_inicio` (date-time) y `fecha_fin` (date-time).
- Una **vacante** representa que un turno quedó sin cobertura. Una vacante **no existe fuera de un turno**: siempre pertenece a uno y solo uno. Tiene `id`, `turno_id`, `estado` (string, siempre "libre" al crearse) y `motivo` (string, opcional).
- El `id` de cualquier recurso lo genera el servidor, nunca lo manda el cliente.
- Cuando un dato ya viaja en el path de la URL (como `turnoId` en `/turnos/{turnoId}/vacantes`), ese mismo dato no debe volver a pedirse en el body del POST.
- Los schemas de entrada (lo que el cliente manda) y de salida (lo que el servidor devuelve) tienen que ser schemas distintos y separados: `TurnoInput`/`Turno` y `VacanteInput`/`Vacante`. El schema de entrada nunca incluye `id`, ni el `turno_id` cuando ya está en el path, ni el `estado` (que siempre lo asigna el servidor).

Necesito los siguientes tres paths y cinco endpoints:

1. `/turnos`
   - `GET`: lista todos los turnos. Responde `200` con un array de `Turno`.
   - `POST`: crea un turno. Recibe `TurnoInput` en el body. Responde `201` con el `Turno` creado, o `400` si falta `medico_titular`, `fecha_inicio` o `fecha_fin`.

2. `/turnos/{turnoId}/vacantes`
   - `GET`: lista las vacantes de ese turno. Responde `200` con un array de `Vacante`, o `404` si el turno no existe.
   - `POST`: genera una vacante para ese turno. Recibe `VacanteInput` en el body. Responde `201` con la `Vacante` creada, o `404` si el turno no existe.

3. `/turnos/{turnoId}/vacantes/{vacanteId}`
   - `DELETE`: cancela/libera esa vacante. Responde `204` sin cuerpo, o `404` si la vacante no existe.

Generá el YAML completo, con la sección `components/schemas` incluyendo `Turno`, `TurnoInput`, `Vacante` y `VacanteInput`, cada uno con sus campos `required` bien definidos según lo que describí arriba.

## Prompt 2 (revisión y corrección, si hace falta)

Releé el yaml completo generado y revisá específicamente:
- Que `TurnoInput` no tenga el campo `id`.
- Que `VacanteInput` no tenga los campos `id`, `turno_id` ni `estado` — ninguno de los tres, porque `turno_id` ya viaja en el path y `id`/`estado` los define el servidor.
- Que el `DELETE` devuelva `204` sin `content` definido.
- Que los `404` de ambos paths con `{turnoId}` (o `{vacanteId}`) estén presentes y no se hayan reemplazado por una lista vacía en el `GET`.

Si encontrás alguna inconsistencia con estos puntos, corregila y mostrame el yaml final completo.
