# 02 · Sintaxis, tipos y control de flujo

## 2.1 `val` y `var`

```kotlin
val pi = 3.1416      // referencia INMUTABLE (equivale a final en Java)
var contador = 0     // referencia mutable
contador++
// pi = 3.15         // ERROR de compilación
```

**Norma del módulo: usad `val` siempre por defecto.** Sólo cambiad a `var` cuando el compilador
os obligue. La inmutabilidad reduce errores y es imprescindible al trabajar con concurrencia (unidad 09).

## 2.2 Inferencia y tipos básicos

```kotlin
val edad: Int = 25        // tipo explícito
val edad2 = 25            // inferido: Int
val precio = 19.99        // Double
val precioF = 19.99f      // Float
val inicial = 'C'         // Char
val activo = true         // Boolean
val nombre = "Celia"      // String
```

No existen tipos primitivos visibles: `Int`, `Double`... son objetos. El compilador ya usa `int`
o `double` internamente cuando puede, así que no hay penalización de rendimiento.

**No hay conversión implícita:**

```kotlin
val i = 10
// val l: Long = i        // ERROR
val l: Long = i.toLong()  // conversión explícita
```

## 2.3 Cadenas

```kotlin
val a = 5
val b = 3
println("La suma de $a y $b es ${a + b}")

val multilinea = """
    Consulta SQL:
    SELECT * FROM alumnos
    WHERE curso = 2
""".trimIndent()
```

## 2.4 `if` es una expresión

```kotlin
val nota = 7
// En Kotlin if DEVUELVE un valor, por lo que sustituye al operador ternario de Java
val resultado = if (nota >= 5) "Apto" else "No apto"
```

## 2.5 `when`: el `switch` mejorado

```kotlin
val nota = 7

val calificacion = when {
    nota < 5  -> "Insuficiente"
    nota < 6  -> "Suficiente"
    nota < 7  -> "Bien"
    nota < 9  -> "Notable"
    else      -> "Sobresaliente"
}

// Con argumento y múltiples valores por rama
val diaSemana = 6
when (diaSemana) {
    1, 2, 3, 4, 5 -> println("Laborable")
    6, 7          -> println("Fin de semana")
    else          -> println("Día no válido")
}

// Comprobación de tipos: el "smart cast" evita el casting manual
fun describir(x: Any): String = when (x) {
    is Int    -> "Entero cuyo doble es ${x * 2}"   // aquí x YA es Int
    is String -> "Cadena de ${x.length} caracteres"
    else      -> "Tipo desconocido"
}
```

## 2.6 Bucles y rangos

```kotlin
for (i in 1..5) print(i)            // 12345
for (i in 1 until 5) print(i)       // 1234  (excluye el 5)
for (i in 5 downTo 1) print(i)      // 54321
for (i in 0..10 step 2) print(i)    // 0246810

val lenguajes = listOf("Kotlin", "Java", "C#")
for (l in lenguajes) println(l)
for ((indice, valor) in lenguajes.withIndex()) println("$indice -> $valor")

var n = 3
while (n > 0) { println(n); n-- }
```

## 2.7 Comparación con Java

| Concepto | Java | Kotlin |
|---|---|---|
| Constante local | `final int x = 1;` | `val x = 1` |
| Ternario | `a > b ? a : b` | `if (a > b) a else b` |
| `switch` | sólo ciertos tipos, `break` | `when`, cualquier tipo, sin `break` |
| Casting tras `instanceof` | manual | *smart cast* automático |
| Igualdad de contenido | `.equals()` | `==` |
| Igualdad de referencia | `==` | `===` |

## Ejercicios

1. Programa que lea un número del 1 al 12 y devuelva el nombre del mes usando `when`.
2. Programa que muestre la tabla de multiplicar de un número usando rangos.
3. Función `clasificarEdad(edad: Int): String` que devuelva "Menor", "Adulto" o "Jubilado" usando `when` con rangos (`in 0..17`).
4. Reescribe en Kotlin este código Java, eliminando toda variable mutable innecesaria:
   ```java
   String msg;
   if (temp > 30) { msg = "Calor"; } else { msg = "Templado"; }
   ```

**Para profundizar:** investiga `when` con `in` sobre rangos y colecciones, y el uso de `when` sin `else` cuando el compilador puede demostrar que todos los casos están cubiertos.
