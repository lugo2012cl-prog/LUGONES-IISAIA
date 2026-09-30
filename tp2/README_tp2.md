# TP 2 — API de turnos y vacantes

Un `openapi.yaml` que describe una API donde cada turno de guardia puede tener una vacante asociada, y una vacante no existe fuera de un turno. Cinco endpoints, tres paths, sin nada implementado: el entregable es el contrato.

## Cómo se lee

Pegar el contenido de `openapi.yaml` en [editor.swagger.io](https://editor.swagger.io/). Aparece la documentación navegable del lado derecho, con cada endpoint desplegable.

## Qué me propuse construir

El mismo tipo de jerarquía que el ejemplo del docente (proyectos/tareas), pero con un dominio propio: turnos de guardia y vacantes. Necesitaba dos recursos con una relación de pertenencia real, no una relación opcional resuelta con un filtro, para que anidar el path fuera la decisión correcta y no una preferencia estética.

Turnos y vacantes cumple: una vacante es, por definición, la ausencia de cobertura de un turno puntual. No existe "una vacante" sin un turno del cual sea la vacante — es el mismo tipo de dependencia estructural que "una tarea sin proyecto".

## Decisiones que tomé yo

**Anidar `vacantes` dentro de `turnos` en vez de `/vacantes?turno=4`.**
La decisión de fondo del contrato. Elegí anidar porque la pertenencia es estructural: si el turno se cancela o se elimina, la vacante asociada no tiene sentido por sí sola. Si hubiera querido que una vacante pudiera consultarse o filtrarse independientemente del turno (por ejemplo, para un listado global de "todas las vacantes abiertas del mes"), la forma correcta hubiera sido un path plano con filtro. Las dos son válidas; lo que no es válido es elegir sin darse cuenta de que se está eligiendo.

**Schemas de entrada y de salida separados.**
`TurnoInput` no tiene `id` (lo genera el servidor). `VacanteInput` no tiene `turno_id` (ya viaja en el path, en `/turnos/{turnoId}/vacantes`) ni `id` ni `estado` (el estado inicial lo asigna el servidor al crearla, siempre en `libre`). Colapsar esto en un solo schema con campos opcionales escondería qué es lo que el cliente realmente puede mandar.

**`204` sin cuerpo al liberar/cancelar una vacante (`DELETE`).**
Si la vacante ya no existe, devolver el objeto borrado en el cuerpo de la respuesta describe algo que dejó de estar. Mismo criterio que en el ejemplo del docente.

**`404` en el `GET` de vacantes de un turno inexistente, no lista vacía.**
Pedir las vacantes del turno 99, que no existe, y devolver `[]` sugiere falsamente que el turno existe y no tiene vacantes. Son dos situaciones distintas.

**`estado` no viaja en el input, nunca.**
El estado de una vacante (`libre`) lo define el servidor al crearla; no es algo que el cliente declare. Evita que alguien intente crear una vacante ya "confirmada" desde el request inicial, algo que no tiene sentido de negocio: una vacante recién creada siempre nace libre.

**`medico_titular` requerido en el turno, `motivo` opcional en la vacante.**
Un turno sin médico titular asignado no es un turno. Una vacante puede generarse sin explicar el motivo (a veces el coordinador no necesita ese dato para el contrato mínimo).

## Endpoints (cinco, en tres paths)

| Método | Path | Qué hace |
|---|---|---|
| GET | `/turnos` | Lista todos los turnos |
| POST | `/turnos` | Crea un turno |
| GET | `/turnos/{turnoId}/vacantes` | Lista las vacantes de un turno |
| POST | `/turnos/{turnoId}/vacantes` | Genera una vacante para ese turno |
| DELETE | `/turnos/{turnoId}/vacantes/{vacanteId}` | Cancela/libera una vacante |

## Alcance explícito (qué queda afuera a propósito)

Siguiendo el criterio "acotado" del enunciado: no hay `PUT`/`PATCH` de nada, no hay endpoint para "tomar" una vacante (eso implicaría modelar el semáforo completo y la cola de candidatos, que es lógica de negocio para el TP final, no un contrato de recursos). Este TP2 modela solo la estructura de pertenencia y el ciclo crear/consultar/eliminar, igual que hizo el docente con proyectos y tareas.
