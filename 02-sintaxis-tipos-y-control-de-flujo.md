# 02 · Sintaxis, tipos y control de flujo

> **Convención de los ejemplos:** los ejemplos completos incluyen su propia función `main()` y van seguidos de su **Salida**. Los fragmentos breves, sin `main()`, se prueban colocando sus líneas dentro de una.

## 2.1 `val` y `var`

```kotlin
val pi = 3.1416      // referencia INMUTABLE (equivale a final en Java)
var contador = 0     // referencia mutable
contador++
// pi = 3.15         // ERROR de compilación
```

**Norma del módulo: usad `val` siempre por defecto.** Sólo cambiad a `var` cuando el compilador
os obligue. La inmutabilidad reduce errores y es imprescindible al trabajar con concurrencia (unidad 09).

> 📄 Material complementario: [Diferencia entre val y var](val-vs-var.md), con ejemplos adicionales
> (arrays, comparación con Java, y por qué `val` no implica que el objeto sea inmutable).

## 2.2 Inferencia y tipos básicos

```kotlin
fun main() {
    val edad: Int = 25        // tipo explícito
    val edad2 = 25            // inferido: Int
    val precio = 19.99        // Double
    val precioF = 19.99f      // Float
    val inicial = 'C'         // Char
    val activo = true         // Boolean
    val nombre = "Celia"      // String

    println("$nombre, $edad años, inicial $inicial, activo: $activo, precio: $precio")
}
```

Salida:

```text
Celia, 25 años, inicial C, activo: true, precio: 19.99
```

No existen tipos primitivos visibles: `Int`, `Double`... son objetos. El compilador ya usa `int`
o `double` internamente cuando puede, así que no hay penalización de rendimiento.

**No hay conversión implícita:**

```kotlin
val i = 10
// val l: Long = i        // ERROR
val l: Long = i.toLong()  // conversión explícita
```

### Qué tipo usar para cada dato

| Dato | Tipo | Por qué |
|---|---|---|
| Cantidades que se cuentan (unidades, plazas, número de ejemplar) | `Int` | No tienen decimales |
| Importes, precios, notas, porcentajes, medidas | `Double` | Necesitan decimales y se dividen |
| Textos | `String` | |
| Sí / no | `Boolean` | |

**Un precio nunca es `Int`:** 4,95 € no cabe en un entero, y al calcular un porcentaje se perderían los decimales (apartado 2.3).

## 2.3 Operaciones aritméticas y división entera

Los operadores son los de Java: `+`, `-`, `*`, `/` y `%` (resto). Y también hay que tener cuidado con lo mismo que en Java: **si los dos operandos de `/` son enteros, el resultado es entero** y se descartan los decimales.

```kotlin
fun main() {
    println(7 / 2)          // Int / Int  -> Int
    println(7 % 2)          // resto de la división entera
    println(7.0 / 2)        // Double / Int -> Double
    println(7 / 2.0)        // Int / Double -> Double
    println(10 / 100)       // ¡cuidado!
    println(10.0 / 100)
}
```

Salida:

```text
3
1
3.5
3.5
0
0.1
```

Basta con que **uno** de los dos operandos sea `Double` para que el resultado lo sea.

### El caso de los porcentajes

El error más frecuente aparece al calcular porcentajes con enteros:

```kotlin
fun main() {
    val precio = 20.0

    val porcentajeInt = 10                               // Int
    val subidaMal = precio * (porcentajeInt / 100)       // 10 / 100 = 0 -> 20.0 * 0
    println(subidaMal)

    val porcentaje = 10.0                                // Double
    val subidaBien = precio * (porcentaje / 100)         // 10.0 / 100 = 0.1
    println(subidaBien)

    println(precio * (1 + porcentaje / 100))             // precio ya subido
}
```

Salida:

```text
0.0
2.0
22.0
```

Con `porcentajeInt / 100` la subida es **siempre 0** para cualquier porcentaje menor que 100, y el programa no da ningún error: simplemente calcula mal. Para un 150 %, `150 / 100` da `1` y la subida sería del 100 %, no del 150 %.

Tres formas de evitarlo:

1. Declarar el porcentaje como `Double` desde el principio (lo recomendable).
2. Dividir entre `100.0` en lugar de `100`: `porcentajeInt / 100.0`.
3. Convertir antes: `porcentajeInt.toDouble() / 100`.

> **No convirtáis a `Int` lo que tiene decimales:** `porcentaje.toInt() / 100` reproduce el problema aunque `porcentaje` sea `Double`.

### Mostrar decimales

Las operaciones con `Double` pueden arrastrar pequeños errores de redondeo. Para **mostrar** importes, formatead con dos decimales:

```kotlin
fun main() {
    val total = 0.1 + 0.2
    println(total)
    println("%.2f €".format(total))
}
```

Salida:

```text
0.30000000000000004
0,30 €
```

> El separador decimal de `format` depende de la configuración regional del equipo: puede salir `0,30` o `0.30`.

## 2.4 Cadenas

```kotlin
fun main() {
    val a = 5
    val b = 3
    println("La suma de $a y $b es ${a + b}")

    val multilinea = """
        Consulta SQL:
        SELECT * FROM alumnos
        WHERE curso = 2
    """.trimIndent()
    println(multilinea)
}
```

Salida:

```text
La suma de 5 y 3 es 8
Consulta SQL:
SELECT * FROM alumnos
WHERE curso = 2
```

## 2.5 `if` es una expresión

```kotlin
fun main() {
    val nota = 7
    // En Kotlin if DEVUELVE un valor, por lo que sustituye al operador ternario de Java
    val resultado = if (nota >= 5) "Apto" else "No apto"
    println(resultado)
}
```

Salida:

```text
Apto
```

## 2.6 `when`: el `switch` mejorado

```kotlin
// Comprobación de tipos: el "smart cast" evita el casting manual
fun describir(x: Any): String = when (x) {
    is Int    -> "Entero cuyo doble es ${x * 2}"   // aquí x YA es Int
    is String -> "Cadena de ${x.length} caracteres"
    else      -> "Tipo desconocido"
}

fun main() {
    val nota = 7.5

    val calificacion = when {
        nota < 5  -> "Insuficiente"
        nota < 6  -> "Suficiente"
        nota < 7  -> "Bien"
        nota < 9  -> "Notable"
        else      -> "Sobresaliente"
    }
    println(calificacion)

    // Con argumento y múltiples valores por rama
    val diaSemana = 6
    when (diaSemana) {
        1, 2, 3, 4, 5 -> println("Laborable")
        6, 7          -> println("Fin de semana")
        else          -> println("Día no válido")
    }

    // Con rangos
    val edad = 70
    val grupo = when (edad) {
        in 0..17  -> "Menor"
        in 18..64 -> "Adulto"
        else      -> "Jubilado"
    }
    println(grupo)

    println(describir(21))
    println(describir("Kotlin"))
}
```

Salida:

```text
Notable
Fin de semana
Jubilado
Entero cuyo doble es 42
Cadena de 6 caracteres
```

**Cuándo usar `when` en lugar de `if`:** en cuanto haya más de dos caminos. Una cadena de `if`/`else if` se lee peor y es más fácil dejarse un caso; `when` además puede devolver directamente el valor.

## 2.7 Bucles y rangos

```kotlin
fun main() {
    for (i in 1..5) print(i)            // 12345
    println()
    for (i in 1 until 5) print(i)       // 1234  (excluye el 5)
    println()
    for (i in 5 downTo 1) print(i)      // 54321
    println()
    for (i in 0..10 step 2) print(i)    // 0246810
    println()

    val lenguajes = listOf("Kotlin", "Java", "C#")
    for (l in lenguajes) println(l)
    for ((indice, valor) in lenguajes.withIndex()) println("$indice -> $valor")

    var n = 3
    while (n > 0) { println(n); n-- }
}
```

Salida:

```text
12345
1234
54321
0246810
Kotlin
Java
C#
0 -> Kotlin
1 -> Java
2 -> C#
3
2
1
```

**Cuidado con `..` y los índices:** `0..lista.size` **incluye** `lista.size`, que es una posición que no existe (las posiciones van de `0` a `size - 1`). Para recorrer posiciones, usad `0 until lista.size` o `lista.indices`; y si no necesitáis la posición, recorred los elementos directamente: `for (x in lista)`.

### Un menú con `do`/`while` y `when`

Patrón habitual en los programas de consola. Lo que tiene que **conservarse** entre una vuelta y otra (una lista, un contador) se declara **antes** del bucle:

```kotlin
fun main() {
    val historial = mutableListOf<String>()       // fuera del bucle: se conserva
    val entradas = listOf("1", "2", "x", "0")     // simula lo que escribiría el usuario
    var i = 0

    do {
        val opcion = entradas[i++].toIntOrNull()  // en un programa real: readlnOrNull()
        when (opcion) {
            1 -> historial.add("Saludo")
            2 -> println("Historial: $historial")
            0 -> println("Fin")
            else -> println("Opción no válida")
        }
    } while (opcion != 0)
}
```

Salida:

```text
Historial: [Saludo]
Opción no válida
Fin
```

Si `historial` se declarase dentro del `do`, se crearía vacía en cada vuelta y la opción 2 mostraría siempre `[]`. La lectura segura de datos con `readlnOrNull()` y `toIntOrNull()` se ve en las unidades 01, 03 y 04.

## 2.8 Comparación con Java

| Concepto | Java | Kotlin |
|---|---|---|
| Constante local | `final int x = 1;` | `val x = 1` |
| Ternario | `a > b ? a : b` | `if (a > b) a else b` |
| `switch` | sólo ciertos tipos, `break` | `when`, cualquier tipo, sin `break` |
| Casting tras `instanceof` | manual | *smart cast* automático |
| Igualdad de contenido | `.equals()` | `==` |
| Igualdad de referencia | `==` | `===` |
| División entre enteros | `7 / 2` da `3` | **Igual**: `7 / 2` da `3` |
| Conversión entre tipos numéricos | Implícita si no se pierde precisión | Siempre explícita: `toLong()`, `toDouble()` |

## 2.9 Errores típicos

| Error | Qué ocurre | Solución |
|---|---|---|
| Porcentaje `Int` dividido entre `100` | La subida o el descuento valen 0 para cualquier porcentaje menor que 100 | Porcentaje `Double`, o dividir entre `100.0` |
| `porcentaje.toInt() / 100` | Mismo problema, aunque el porcentaje fuera `Double` | No convertir a `Int` lo que tiene decimales |
| Precios o importes como `Int` | No caben los céntimos y los cálculos pierden decimales | `Double` |
| Mostrar un `Double` sin formato | Salen cosas como `0.30000000000000004` | `"%.2f".format(valor)` |
| `for (i in 0..lista.size)` | `IndexOutOfBoundsException` en la última vuelta | `0 until lista.size`, `lista.indices` o `for (x in lista)` |
| Cadena de `if`/`else if` para varios casos | Difícil de leer; es fácil dejarse un caso | `when` |
| `var` donde basta `val` | Variables que pueden cambiar sin querer | `val` por defecto |
| Variable que debe conservarse declarada dentro del bucle | Se reinicia en cada vuelta | Declararla antes del bucle |

## Ejercicios

1. Programa que lea un número del 1 al 12 y devuelva el nombre del mes usando `when`.
2. Programa que muestre la tabla de multiplicar de un número usando rangos.
3. Función `clasificarEdad(edad: Int): String` que devuelva "Menor", "Adulto" o "Jubilado" usando `when` con rangos (`in 0..17`).
4. Reescribe en Kotlin este código Java, eliminando toda variable mutable innecesaria:
   ```java
   String msg;
   if (temp > 30) { msg = "Calor"; } else { msg = "Templado"; }
   ```
5. **Predice la salida.** Sin ejecutarlo, escribe qué imprimirá cada línea. Después compruébalo.
   ```kotlin
   println(9 / 4)
   println(9 % 4)
   println(9 / 4.0)
   println(50 * (20 / 100))
   println(50 * (20 / 100.0))
   println(50 * 20 / 100)
   ```
6. **Encuentra el error.** Este programa debería aplicar un 15 % de descuento a un precio de 80 € y mostrar `68.0`, pero muestra `80.0`. Explica por qué y corrígelo de dos formas distintas.
   ```kotlin
   val precio = 80.0
   val descuento = 15
   val final = precio - precio * (descuento / 100)
   println(final)
   ```
7. Programa que pida el importe de una compra y aplique un descuento según el importe usando `when` con rangos: menos de 50 €, sin descuento; de 50 a 100 €, un 5 %; más de 100 €, un 10 %. Muestra el resultado con dos decimales.

**Para profundizar:** investiga `when` con `in` sobre rangos y colecciones, y el uso de `when` sin `else` cuando el compilador puede demostrar que todos los casos están cubiertos. Averigua también qué es `BigDecimal` y por qué las aplicaciones bancarias no usan `Double` para el dinero.
