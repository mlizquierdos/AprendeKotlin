# 05 · Colecciones y operadores funcionales

> **Convención de los ejemplos:** las clases se declaran **fuera** de `main`, a nivel de fichero. Dentro de `main` solo se crean los objetos y se usan.

## 5.1 Mutables e inmutables

Kotlin distingue explícitamente entre colecciones de **sólo lectura** y **mutables**.

```kotlin
fun main() {
    val lista = listOf("Kotlin", "Java")             // List<String>, sólo lectura
    val listaMutable = mutableListOf("Kotlin")       // MutableList<String>
    listaMutable.add("Swift")

    val conjunto = setOf(1, 2, 2, 3)                 // [1, 2, 3] sin duplicados
    val mapa = mapOf("DAM" to 2, "DAW" to 2)         // Map<String, Int>
    val mapaMutable = mutableMapOf<String, Int>()
    mapaMutable["ASIR"] = 2

    println(lista)
    println(listaMutable)
    println(conjunto)
    println(mapa)
    println(mapaMutable)
}
```

Salida:

```text
[Kotlin, Java]
[Kotlin, Swift]
[1, 2, 3]
{DAM=2, DAW=2}
{ASIR=2}
```

> Preferid siempre la versión inmutable. Sólo usad las mutables cuando la colección
> tenga que cambiar realmente de contenido.

**Dónde se declara una colección que va a crecer.** Si un programa con menú va añadiendo elementos, la colección se declara **antes** del bucle. Declarada dentro, se crea vacía de nuevo en cada vuelta y se pierde todo lo añadido:

```kotlin
val tareas = mutableListOf<String>()       // fuera del bucle: se conserva
do {
    // ... leer la opción y añadir a "tareas" ...
} while (opcion != 0)
```

## 5.2 Acceso

```kotlin
fun main() {
    val lenguajes = listOf("Kotlin", "Java", "C#")
    println(lenguajes[0])
    println(lenguajes.first())
    println(lenguajes.last())
    println(lenguajes.getOrNull(10))        // null en vez de excepción
    println(lenguajes.size)

    val notas = mapOf("Ana" to 8, "Luis" to 5)
    println(notas["Ana"])                   // 8  (devuelve Int?)
    for ((alumno, nota) in notas) println("$alumno: $nota")
}
```

Salida:

```text
Kotlin
Kotlin
C#
null
3
8
Ana: 8
Luis: 5
```

Para recorrer una lista no hacen falta índices. Si los necesitáis, usad `lista.indices` o `0 until lista.size`, **nunca** `0..lista.size`: incluye una posición de más y lanza `IndexOutOfBoundsException`.

## 5.3 La idea clave: los operadores devuelven una colección nueva

Todos los operadores de este tema (`filter`, `map`, `sortedBy`…) **devuelven una colección nueva** y **no modifican la original**. Si no se guarda o se usa lo que devuelven, el trabajo se pierde.

```kotlin
fun main() {
    val precios = listOf(10.0, 20.0, 30.0)

    precios.map { it * 1.21 }                 // se calcula... y se tira
    println(precios)                          // la original no ha cambiado

    val conIva = precios.map { it * 1.21 }    // ahora sí: se guarda el resultado
    println(conIva)
}
```

Salida:

```text
[10.0, 20.0, 30.0]
[12.1, 24.2, 36.3]
```

Esto tiene dos consecuencias prácticas:

- **Para obtener datos nuevos**, guardad el resultado: `val baratos = productos.filter { it.precio < 10 }`.
- **Para cambiar los objetos** de la lista (por ejemplo, subir el precio de todos), `map` no sirve: hay que recorrerlos con `forEach` y llamar a un método de cada objeto que los modifique (apartado 5.6).

## 5.4 Operadores funcionales

Este es el estilo que usaréis constantemente en Android.

```kotlin
data class Alumno(val nombre: String, val nota: Double, val curso: Int)

fun main() {
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

    // firstOrNull: el primero que cumple la condición, o null si no hay ninguno
    val buscado = alumnos.firstOrNull { it.nombre == "Pedro" }

    // groupBy: Map<Int, List<Alumno>>
    val porCurso = alumnos.groupBy { it.curso }

    println(aprobados.map { it.nombre })
    println(nombres)
    println(buscado ?: "No existe")
    println(porCurso.mapValues { (_, lista) -> lista.size })
}
```

Salida:

```text
[Ana, Marta, Iván]
[Ana, Luis, Marta, Iván]
No existe
{2=2, 1=2}
```

### Agregados: de una colección a un solo valor

```kotlin
data class Alumno(val nombre: String, val nota: Double, val curso: Int)

fun main() {
    val alumnos = listOf(
        Alumno("Ana", 8.5, 2),
        Alumno("Luis", 4.0, 1),
        Alumno("Marta", 9.2, 2),
        Alumno("Iván", 6.1, 1)
    )

    val sumaNotas = alumnos.sumOf { it.nota }               // suma de lo que devuelve la lambda
    val media = alumnos.map { it.nota }.average()
    val mejor = alumnos.maxByOrNull { it.nota }
    val numAprobados = alumnos.count { it.nota >= 5 }       // cuántos cumplen la condición
    val haySuspensos = alumnos.any { it.nota < 5 }
    val todosAprobados = alumnos.all { it.nota >= 5 }

    println("Suma: %.2f · Media: %.2f".format(sumaNotas, media))
    println("Mejor: ${mejor?.nombre} · Aprobados: $numAprobados")
    println("¿Hay suspensos? $haySuspensos · ¿Todos aprobados? $todosAprobados")
    println(listOf(3, 4, 5).sum())                          // sum(): para listas de números
}
```

Salida:

```text
Suma: 27,80 · Media: 6,95
Mejor: Marta · Aprobados: 3
¿Hay suspensos? true · ¿Todos aprobados? false
12
```

> Las sumas de `Double` pueden arrastrar pequeños errores de redondeo: sin `format`, la primera línea mostraría `27.799999999999997`. Para **mostrar** importes y medias, formatead con `"%.2f".format(...)`. El separador decimal depende de la configuración regional del equipo: puede salir `27,80` o `27.80`.

| Quiero obtener… | Operador |
|---|---|
| La suma de una propiedad (precios, horas, notas) | `sumOf { it.propiedad }` |
| La suma de una lista de números | `sum()` |
| Cuántos elementos cumplen una condición | `count { condición }` |
| Cuántos son de un tipo concreto (unidad 07) | `count { it is Libro }` |
| La media | `map { ... }.average()` |
| El mayor o el menor | `maxByOrNull { }` / `minByOrNull { }` |
| Si alguno o todos cumplen algo | `any { }` / `all { }` |

Con `sumOf` y `count` no hacen falta acumuladores ni bucles:

```kotlin
// En vez de esto…
var total = 0.0
for (a in alumnos) total += a.nota

// …esto
val total = alumnos.sumOf { it.nota }
```

### Buscar textos sin distinguir mayúsculas

Cuando el texto lo escribe el usuario, compara con `equals(..., ignoreCase = true)`; con `==`, «planeta» no encuentra «Planeta».

```kotlin
val buscados = productos.filter { it.marca.equals(marcaPedida, ignoreCase = true) }
if (buscados.isEmpty()) println("No hay productos de $marcaPedida")
else buscados.forEach { println(it) }
```

### Encadenamiento

Los operadores se encadenan y se leen de arriba abajo, como una tubería:

```kotlin
data class Alumno(val nombre: String, val nota: Double, val curso: Int)

fun main() {
    val alumnos = listOf(
        Alumno("Ana", 8.5, 2),
        Alumno("Luis", 4.0, 1),
        Alumno("Marta", 9.2, 2),
        Alumno("Iván", 6.1, 1)
    )

    val resumen = alumnos
        .filter { it.curso == 2 }
        .sortedByDescending { it.nota }
        .joinToString(", ") { "${it.nombre} (${it.nota})" }

    println(resumen)
}
```

Salida:

```text
Marta (9.2), Ana (8.5)
```

## 5.5 Ordenar: copia ordenada u ordenar la propia lista

Hay dos familias de funciones, y es fácil confundirlas:

| Función | ¿Qué hace? | Funciona con |
|---|---|---|
| `sorted()` | Devuelve una **copia** ordenada (orden natural) | Cualquier lista |
| `sortedBy { }` / `sortedByDescending { }` | Devuelve una **copia** ordenada por un criterio | Cualquier lista |
| `sort()` | **Ordena la propia lista**, no devuelve nada | Solo `MutableList` |
| `sortBy { }` / `sortByDescending { }` | **Ordena la propia lista** por un criterio | Solo `MutableList` |

Truco para recordarlo: las que terminan en **-ed** (*sorted*, «ordenada») devuelven una lista nueva.

```kotlin
fun main() {
    val numeros = mutableListOf(3, 1, 2)

    numeros.sorted()                    // copia ordenada... que nadie guarda
    println(numeros)                    // sigue igual

    val copia = numeros.sorted()
    println("Copia: $copia · Original: $numeros")

    numeros.sort()                      // ordena la propia lista
    println(numeros)
}
```

Salida:

```text
[3, 1, 2]
Copia: [1, 2, 3] · Original: [3, 1, 2]
[1, 2, 3]
```

**Cuál elegir:** si os piden «mostrar ordenado **sin modificar** la colección», `sortedBy` y mostrar la copia. Si os piden «ordenar la colección», `sortBy` sobre una `MutableList`.

```kotlin
productos.sortedBy { it.precio }.forEach { println(it) }   // muestra ordenado; productos no cambia
```

## 5.6 `forEach`, `map` o `filter`: cuál usar

| Quiero… | Uso | Devuelve |
|---|---|---|
| Hacer algo con cada elemento (imprimir, modificar el objeto) | `forEach { }` | Nada |
| Obtener otra lista con cada elemento transformado | `map { }` | Una lista nueva |
| Obtener otra lista solo con algunos elementos | `filter { }` | Una lista nueva |

```kotlin
class Producto(val nombre: String, precio: Double) {
    var precio = precio
        private set

    fun subirPrecio(porcentaje: Double) {
        precio *= 1 + porcentaje / 100
    }

    override fun toString() = "$nombre: %.2f €".format(precio)
}

fun main() {
    val productos = listOf(Producto("Teclado", 40.0), Producto("Ratón", 20.0))

    productos.forEach { it.subirPrecio(10.0) }         // modifica cada objeto
    productos.forEach { println(it) }                  // imprime cada objeto

    val nombres = productos.map { it.nombre }          // otra lista, con los nombres
    println(nombres)
}
```

Salida:

```text
Teclado: 44,00 €
Ratón: 22,00 €
[Teclado, Ratón]
```

> El separador decimal de `format` depende de la configuración regional del equipo: puede salir `44,00` o `44.00`.

Dos detalles que dan muchos problemas:

- `productos.forEach { it.toString() }` **no imprime nada**: genera cada texto y lo descarta. Para mostrarlos, `forEach { println(it) }`.
- `println(productos)` muestra la lista entera entre corchetes. Si la clase no sobrescribe `toString()` (unidad 06), se verán referencias del tipo `Producto@3fee733d`.

## 5.7 Resumir una colección en una `data class`

Cuando hay que devolver **varios datos calculados** sobre una colección (totales, recuentos, máximos…), se agrupan en una `data class` (unidad 06). Así una función puede devolverlos juntos, con nombre, y quien la llama puede desestructurarlos.

```kotlin
data class Venta(val producto: String, val unidades: Int, val precioUnidad: Double)

data class ResumenVentas(
    val totalSinIva: Double,
    val totalConIva: Double,
    val unidades: Int,
    val ventasGrandes: Int
)

fun resumir(ventas: List<Venta>) = ResumenVentas(
    totalSinIva = ventas.sumOf { it.unidades * it.precioUnidad },
    totalConIva = ventas.sumOf { it.unidades * it.precioUnidad * 1.21 },
    unidades = ventas.sumOf { it.unidades },
    ventasGrandes = ventas.count { it.unidades >= 10 }
)

fun main() {
    val ventas = listOf(
        Venta("Cuaderno", 12, 2.5),
        Venta("Bolígrafo", 30, 0.8),
        Venta("Mochila", 2, 25.0)
    )

    val resumen = resumir(ventas)
    println(resumen)                                   // toString() de la data class

    val (sinIva, conIva, unidades, grandes) = resumen  // destructuring
    println("Total: %.2f € (%.2f € con IVA) · %d unidades · %d ventas grandes"
        .format(sinIva, conIva, unidades, grandes))
}
```

Salida:

```text
ResumenVentas(totalSinIva=104.0, totalConIva=125.84, unidades=44, ventasGrandes=2)
Total: 104,00 € (125,84 € con IVA) · 44 unidades · 2 ventas grandes
```

## 5.8 Destructuring

```kotlin
data class Alumno(val nombre: String, val nota: Double, val curso: Int)

fun main() {
    val alumnos = listOf(Alumno("Ana", 8.5, 2), Alumno("Luis", 4.0, 1))
    val (nombre, nota, curso) = alumnos[0]
    println("$nombre ha sacado $nota en $curso.º")
}
```

Salida:

```text
Ana ha sacado 8.5 en 2.º
```

## 5.9 Equivalencia con Java Streams

| Kotlin | Java |
|---|---|
| `lista.filter { }` | `stream().filter().collect()` |
| `lista.map { }` | `stream().map().collect()` |
| `lista.forEach { }` | `forEach()` |
| `lista.sortedBy { }` | `stream().sorted(Comparator.comparing())` |
| `lista.sumOf { }` | `stream().mapToDouble().sum()` |
| `lista.count { }` | `stream().filter().count()` |

En Kotlin no hace falta abrir ni cerrar el *stream*: los operadores trabajan directamente sobre la colección.

## 5.10 Errores típicos

| Error | Qué ocurre | Solución |
|---|---|---|
| `lista.sorted()` o `lista.map { }` sin guardar el resultado | La lista original no cambia y el resultado se pierde | `val nueva = lista.sorted()`, o usar el resultado directamente |
| Usar `map` para «modificar» los objetos | Se crea otra lista; los objetos siguen igual | `forEach { it.metodoQueModifica() }` |
| `forEach { it.toString() }` o `forEach { it.mostrar() }` | No se imprime nada | `forEach { println(it) }` |
| `sortBy` cuando se pide no modificar la colección | Se pierde el orden original | `sortedBy` y mostrar la copia |
| Totales con `var` y un bucle | Funciona, pero es más largo y propenso a errores | `sumOf { }` y `count { }` |
| Buscar textos con `==` | «planeta» no encuentra «Planeta» | `equals(texto, ignoreCase = true)` |
| `for (i in 0..lista.size)` | `IndexOutOfBoundsException` en la última vuelta | `for (x in lista)`, `lista.indices` o `0 until lista.size` |
| Declarar la lista dentro del bucle del menú | Se vacía en cada vuelta | Declararla antes del bucle |
| Imprimir una lista de objetos sin `toString()` | Se ven referencias `Clase@3fee733d` | Sobrescribir `toString()` (unidad 06) |

## Ejercicios

1. A partir de una lista de 10 alumnos, calcula: media de la clase, porcentaje de aprobados y nombre del mejor alumno.
2. Agrupa una lista de palabras por su primera letra (`groupBy`).
3. Dada una lista de precios, obtén la suma total aplicando un 21 % de IVA, en una única cadena de operadores. Resuélvelo primero con `map` y `sum()` y después solo con `sumOf`.
4. Implementa un pequeño inventario con `MutableMap<String, Int>` que permita añadir, restar y listar existencias.
5. **Predice la salida.** Sin ejecutarlo, escribe qué imprimirá cada bloque. Después compruébalo.
   ```kotlin
   // A
   val a = mutableListOf(5, 2, 8)
   a.sortedDescending()
   println(a)

   // B
   val b = mutableListOf(5, 2, 8)
   b.sortDescending()
   println(b)

   // C
   val c = listOf("Ana", "Luis", "Eva")
   c.forEach { it.uppercase() }
   println(c)

   // D
   val d = listOf(4.0, 6.0, 10.0)
   println(d.sumOf { it / 2 })

   // E
   val e = listOf("Kotlin", "java", "Swift")
   println(e.count { it.equals("JAVA", ignoreCase = true) })
   ```
6. Con una lista de al menos cinco películas (`data class Pelicula(val titulo: String, val genero: String, val minutos: Int)`):
   - Muestra las películas ordenadas por duración **sin modificar** la lista original, y comprueba después que la original conserva su orden.
   - Pide un género por consola y muestra las películas de ese género sin distinguir mayúsculas; si no hay ninguna, muestra un aviso.
   - Crea una `data class ResumenCatalogo` con el total de minutos, la duración media y el número de películas de más de dos horas, y muéstrala con *destructuring*.

**Para profundizar:** compara `map` con `flatMap` y estudia las *sequences* (`asSequence()`) para colecciones muy grandes.
