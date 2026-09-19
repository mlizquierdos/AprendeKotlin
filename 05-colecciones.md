# 05 · Colecciones y operadores funcionales

## 5.1 Mutables e inmutables

Kotlin distingue explícitamente entre colecciones de **sólo lectura** y **mutables**.

```kotlin
val lista = listOf("Kotlin", "Java")             // List<String>, sólo lectura
val listaMutable = mutableListOf("Kotlin")       // MutableList<String>
listaMutable.add("Swift")

val conjunto = setOf(1, 2, 2, 3)                 // [1, 2, 3] sin duplicados
val mapa = mapOf("DAM" to 2, "DAW" to 2)         // Map<String, Int>
val mapaMutable = mutableMapOf<String, Int>()
mapaMutable["ASIR"] = 2
```

> Preferid siempre la versión inmutable. Sólo usad las mutables cuando la colección
> tenga que cambiar realmente de contenido.

## 5.2 Acceso

```kotlin
val lenguajes = listOf("Kotlin", "Java", "C#")
println(lenguajes[0])
println(lenguajes.first())
println(lenguajes.last())
println(lenguajes.getOrNull(10))        // null en vez de excepción
println(lenguajes.size)

val notas = mapOf("Ana" to 8, "Luis" to 5)
println(notas["Ana"])                   // 8  (devuelve Int?)
for ((alumno, nota) in notas) println("$alumno: $nota")
```

## 5.3 Operadores funcionales

Este es el estilo que usaréis constantemente en Android.

```kotlin
data class Alumno(val nombre: String, val nota: Double, val curso: Int)

val alumnos = listOf(
    Alumno("Ana", 8.5, 2),
    Alumno("Luis", 4.0, 1),
    Alumno("Marta", 9.2, 2),
    Alumno("Iván", 6.1, 1)
)

// filter: selecciona los que cumplen una condición
val aprobados = alumnos.filter { it.nota >= 5 }

// map: transforma cada elemento
val nombres = alumnos.map { it.nombre }

// sortedBy / sortedByDescending
val ranking = alumnos.sortedByDescending { it.nota }

// groupBy: Map<Int, List<Alumno>>
val porCurso = alumnos.groupBy { it.curso }

// agregados
val media = alumnos.map { it.nota }.average()
val mejor = alumnos.maxByOrNull { it.nota }
val haySuspensos = alumnos.any { it.nota < 5 }
val todosAprobados = alumnos.all { it.nota >= 5 }
val numAprobados = alumnos.count { it.nota >= 5 }

// Encadenamiento: se lee de arriba abajo como una tubería
val resumen = alumnos
    .filter { it.curso == 2 }
    .sortedByDescending { it.nota }
    .joinToString(", ") { "${it.nombre} (${it.nota})" }
```

### Equivalencia con Java Streams

| Kotlin | Java |
|---|---|
| `lista.filter { }` | `stream().filter().collect()` |
| `lista.map { }` | `stream().map().collect()` |
| `lista.forEach { }` | `forEach()` |
| `lista.sortedBy { }` | `stream().sorted(Comparator.comparing())` |

En Kotlin no hace falta abrir ni cerrar el *stream*: los operadores trabajan directamente sobre la colección.

## 5.4 Destructuring

```kotlin
val (nombre, nota, curso) = alumnos[0]
println("$nombre ha sacado $nota")
```

## Ejercicios

1. A partir de una lista de 10 alumnos, calcula: media de la clase, porcentaje de aprobados y nombre del mejor alumno.
2. Agrupa una lista de palabras por su primera letra (`groupBy`).
3. Dada una lista de precios, obtén la suma total aplicando un 21 % de IVA, en una única cadena de operadores.
4. Implementa un pequeño inventario con `MutableMap<String, Int>` que permita añadir, restar y listar existencias.

**Para profundizar:** compara `map` con `flatMap` y estudia las *sequences* (`asSequence()`) para colecciones muy grandes.
