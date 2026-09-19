# 11 · Proyecto integrador

## Enunciado

Desarrollad una aplicación Android de **gestión de tareas** («Mis tareas») que aplique
todo lo trabajado en las unidades anteriores.

## Fase 1 — Modelo y lógica (consola, IntelliJ)

- `data class Tarea(val id: Int, val titulo: String, val descripcion: String? = null, val completada: Boolean = false, val prioridad: Prioridad = Prioridad.MEDIA)`
- `enum class Prioridad { ALTA, MEDIA, BAJA }`
- `object GestorTareas` con: añadir, eliminar, marcar como completada (usando `copy()`), listar, filtrar por prioridad y estadísticas (`count`, `groupBy`).
- Menú por consola.

**Requisito:** ninguna colección mutable expuesta al exterior y ningún uso de `!!`.

## Fase 2 — Persistencia y asincronía

- Guardar y leer las tareas en un fichero de texto o JSON.
- Las operaciones de fichero deben ir en funciones `suspend` con `Dispatchers.IO`.
- El resultado se devuelve con `sealed class Resultado<out T>`.

## Fase 3 — Aplicación Android

- Pantalla de lista con `LazyColumn`.
- Pantalla o diálogo de alta de tarea.
- `ViewModel` con `StateFlow` y estado modelado con `sealed class`.
- Filtro por prioridad y contador de tareas pendientes.

## Fase 4 — Ampliación (voluntaria)

- Persistencia con Room.
- Tema claro/oscuro.
- Fecha de vencimiento y ordenación por fecha.

## Criterios de evaluación

| Criterio | Peso |
|---|---|
| Uso correcto de `val`, inmutabilidad y `copy()` | 15 % |
| Seguridad frente a nulos sin `!!` | 15 % |
| Modelado con `data class`, `enum` y `sealed class` | 15 % |
| Operadores funcionales sobre colecciones | 10 % |
| Corrutinas y elección correcta de dispatcher | 15 % |
| Separación en capas (modelo / datos / vista) | 20 % |
| Comentarios que justifiquen las decisiones de diseño | 10 % |

## Entrega

- Repositorio Git propio con *commits* significativos (no un único commit final).
- `README.md` describiendo la estructura, cómo ejecutarlo y las decisiones de diseño tomadas.
- Código comentado allí donde la decisión no sea evidente.
