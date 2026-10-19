# 03 · Seguridad frente a nulos

> **Convención de los ejemplos:** los ejemplos completos incluyen su propia función `main()` y van seguidos de su **Salida**. Las funciones y clases se declaran **fuera** de `main()`.

Es **la** diferencia conceptual más importante respecto a Java. En Java cualquier referencia puede
ser `null` y el error aparece en ejecución. En Kotlin el compilador lo impide.

## 3.1 Tipos anulables

```kotlin
var nombre: String = "Celia"
// nombre = null              // ERROR de compilación

var apodo: String? = "Cel"    // el ? marca el tipo como anulable
apodo = null                  // correcto
```

Si se descomenta la segunda línea:

```text
error: null cannot be a value of a non-null type 'kotlin.String'.
```

`String` y `String?` son **tipos distintos**. Todo `String` es un `String?` válido, pero no al revés.

**Cuándo usar un tipo anulable:** solo cuando «no tener valor» es una situación **válida y esperada**: un apodo que no todo el mundo tiene, el autor de un libro anónimo, un tutor sin asignar. Si un dato siempre tiene que existir (el título de un libro), no debe ser anulable.

## 3.2 Acceso seguro `?.`

```kotlin
fun main() {
    val apodo: String? = null
    // println(apodo.length)      // ERROR: puede ser null
    println(apodo?.length)        // imprime null, no lanza excepción

    val otro: String? = "Cel"
    println(otro?.length)
}
```

Salida:

```text
null
3
```

Si se descomenta la línea con `apodo.length`:

```text
error: only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type 'kotlin.String?'.
```

Se pueden encadenar:

```kotlin
val ciudad = alumno?.direccion?.ciudad   // null si algo de la cadena es null
```

## 3.3 Operador Elvis `?:`

Proporciona un valor alternativo cuando la expresión de la izquierda es `null`.

```kotlin
// También sirve para salir de una función pronto
fun procesar(texto: String?) {
    val valor = texto ?: return            // si es null, se abandona la función
    println(valor.uppercase())
}

fun main() {
    val apodo: String? = null
    val longitud = apodo?.length ?: 0
    val nombreMostrado = apodo ?: "Sin apodo"
    println("$nombreMostrado ($longitud)")

    procesar("hola")
    procesar(null)                         // no imprime nada
}
```

Salida:

```text
Sin apodo (0)
HOLA
```

**El Elvis solo actúa si la izquierda puede ser `null`.** Si la expresión de la izquierda nunca es nula, el valor de la derecha no se usa jamás (apartado 3.9).

## 3.4 `!!`: la aserción no nula

```kotlin
val longitud = apodo!!.length   // lanza NullPointerException si apodo es null
```

> **Regla del módulo:** `!!` se considera un error de diseño salvo justificación escrita.
> Si lo usáis en una práctica, explicad en un comentario por qué es seguro.

## 3.5 `let` sobre anulables

```kotlin
fun enviarCorreo(email: String?) {
    email?.let {
        // este bloque SOLO se ejecuta si email no es null
        // dentro, "it" es de tipo String (no anulable)
        println("Enviando correo a ${it.lowercase()}")
    }
}

fun main() {
    enviarCorreo("Celia@Centro.es")
    enviarCorreo(null)                     // no hace nada
}
```

Salida:

```text
Enviando correo a celia@centro.es
```

## 3.6 Comprobación y *smart cast*

```kotlin
fun saludar(nombre: String?) {
    if (nombre != null) {
        // el compilador sabe que aquí nombre es String
        println("Hola, ${nombre.uppercase()}")
    } else {
        println("Hola, desconocido")
    }
}

fun main() {
    saludar("Ana")
    saludar(null)
}
```

Salida:

```text
Hola, ANA
Hola, desconocido
```

Cuando solo se necesita un valor alternativo, el Elvis es más corto: `println("Hola, ${nombre?.uppercase() ?: "desconocido"}")`.

## 3.7 Datos opcionales en una clase

Lo más habitual en un programa real no es una variable suelta anulable, sino una **propiedad opcional** dentro de una clase: el autor de un libro, el tutor de un curso, el teléfono de un cliente. El patrón tiene tres partes:

1. Tipo anulable: `String?`.
2. Valor por defecto `null`, para no tener que indicarlo al crear el objeto.
3. El texto alternativo se decide **al mostrarlo**, con `?:`.

```kotlin
class Libro(
    val titulo: String,
    val autor: String? = null              // 1 y 2: anulable y con valor por defecto
) {
    override fun toString() = "$titulo, de ${autor ?: "Anónimo"}"   // 3
}

fun main() {
    val libros = listOf(
        Libro("Platero y yo", "Juan Ramón Jiménez"),
        Libro("Lazarillo de Tormes")       // sin autor: queda null
    )
    libros.forEach { println(it) }

    val sinAutor = libros.count { it.autor == null }
    println("Libros sin autor conocido: $sinAutor")
}
```

Salida:

```text
Platero y yo, de Juan Ramón Jiménez
Lazarillo de Tormes, de Anónimo
Libros sin autor conocido: 1
```

**Por qué se guarda `null` y no el texto `"Anónimo"`:**

- `null` significa «no se sabe»; `"Anónimo"` es un texto que podría confundirse con un autor real.
- Con `null` se puede preguntar `autor == null` para contar, filtrar o validar.
- El texto que se muestra puede cambiar (en otro idioma, en otra pantalla) sin tocar los datos.

`toString()` se explica en la unidad 06, y los valores por defecto en los constructores, en las unidades 04 y 06.

## 3.8 Conversiones seguras

```kotlin
fun main() {
    val texto = "42a"
    val numero: Int? = texto.toIntOrNull()      // null en vez de excepción
    println(numero ?: -1)

    println("42".toIntOrNull())
    println("3,5".toDoubleOrNull())             // con coma: no es un Double válido
    println("3,5".replace(',', '.').toDoubleOrNull())

    val obj: Any = "hola"
    val n = obj as? Int                         // cast seguro: null si no es Int
    println(n)
}
```

Salida:

```text
-1
42
null
3.5
null
```

## 3.9 Leer datos del usuario sin que el programa se cierre

Todo lo que escribe el usuario es texto, y puede no ser lo que esperamos. Esta es la combinación correcta:

| Paso | Función | Si falla… |
|---|---|---|
| Leer la línea | `readlnOrNull()` | Devuelve `null` si no hay entrada |
| Convertir a número | `toIntOrNull()` / `toDoubleOrNull()` | Devuelve `null` si el texto no es un número |
| Dar un valor alternativo | `?:` | Se usa cuando lo anterior es `null` |

```kotlin
fun main() {
    print("Precio: ")
    val precio = readlnOrNull()?.trim()?.replace(',', '.')?.toDoubleOrNull() ?: 10.0
    println("Precio aplicado: $precio")
}
```

Si el usuario escribe `abc` o deja la línea vacía, el precio es `10.0` y el programa sigue.

### Lo que NO funciona

```kotlin
val precio = readln().toDouble() ?: 10.0       // ¡el ?: 10.0 nunca se usa!
```

`toDouble()` **nunca devuelve `null`**: si el texto no es un número, lanza una excepción. Por tanto, la parte izquierda del Elvis no puede ser nula y el `10.0` no se usa nunca. El código compila, pero con `abc` o con la línea vacía el programa termina:

```text
Exception in thread "main" java.lang.NumberFormatException: For input string: "abc"
Exception in thread "main" java.lang.NumberFormatException: empty String
```

| Función | Con un texto no válido |
|---|---|
| `toInt()` / `toDouble()` | Lanza `NumberFormatException`: el programa se cierra si no se captura |
| `toIntOrNull()` / `toDoubleOrNull()` | Devuelve `null`: se trata con `?:` o con un `if` |

**Regla del módulo:** con lo que escribe el usuario, siempre `toIntOrNull()` o `toDoubleOrNull()`. Para volver a preguntar hasta que el dato sea válido, usad la función `leerDouble()` de la unidad 04 (apartado 4.10).

## 3.10 Resumen de operadores

| Operador | Significado | Cuándo usarlo |
|---|---|---|
| `?` | Declara tipo anulable | Cuando la ausencia de valor es un estado válido |
| `?.` | Llamada segura | Acceso a miembros de un anulable |
| `?:` | Elvis: valor por defecto | Dar alternativa o salir pronto. Solo tiene sentido si la izquierda puede ser `null` |
| `?.let { }` | Ejecutar si no es null | Bloque de varias líneas |
| `as?` | Cast seguro | Conversión de tipo insegura |
| `toIntOrNull()` / `toDoubleOrNull()` | Conversión de texto segura | Todo dato que escribe el usuario |
| `!!` | Forzar no nulo | Casi nunca |

## 3.11 Errores típicos

| Error | Qué ocurre | Solución |
|---|---|---|
| Dato opcional declarado como `String` | Hay que inventar un valor («Anónimo», `""`) para crear el objeto | `String? = null` |
| Guardar `"Anónimo"` o `""` en lugar de `null` | No se distingue «no se sabe» de un valor real | Guardar `null` y usar `?:` al mostrar |
| Dato opcional sin valor por defecto (`val autor: String?`) | Hay que escribir `null` en cada llamada | `val autor: String? = null` |
| `toInt()` o `toDouble()` con lo que escribe el usuario | Una letra o una línea vacía cierran el programa | `toIntOrNull()` / `toDoubleOrNull()` |
| `readln().toDouble() ?: 10.0` | El `?:` nunca actúa; con `abc` el programa se cierra igual | `readlnOrNull()?.toDoubleOrNull() ?: 10.0` |
| Usar `!!` «para que compile» | `NullPointerException` cuando el valor es `null` | `?.`, `?:` o `let` |
| Tipo anulable sin necesidad (`val ejemplar: Int? = 0`) | Obliga a tratar un `null` que nunca puede ocurrir | Tipo no anulable con su valor por defecto: `val ejemplar: Int = 0` |
| `if (x != null) ... else ...` solo para dar un valor alternativo | Código más largo de lo necesario | `x ?: alternativa` |

## Ejercicios

1. Función `longitudNombre(nombre: String?): Int` que devuelva 0 si es null, sin usar `if`.
2. Programa que pida un número por consola y muestre su cuadrado; si la entrada no es válida, debe mostrar `Entrada no válida` sin romperse.
3. Dada `data class Alumno(val nombre: String, val tutor: Tutor?)` y `data class Tutor(val email: String?)`, escribe una función que devuelva el email del tutor o `"desconocido"`.
4. Busca en tus prácticas de 1º un `NullPointerException` que te ocurriera y explica cómo lo habría evitado Kotlin.
5. **Predice la salida.** Sin ejecutarlo, escribe qué imprimirá cada bloque. Después compruébalo.
   ```kotlin
   // A
   val a: String? = null
   println(a?.uppercase() ?: "vacío")

   // B
   println("12".toIntOrNull() ?: 0)
   println("12€".toIntOrNull() ?: 0)

   // C
   class Socio(val nombre: String, val telefono: String? = null)
   val s = Socio("Ana")
   println("${s.nombre}: ${s.telefono ?: "sin teléfono"}")

   // D
   val d: String? = "Kotlin"
   d?.let { println(it.length) }
   val e: String? = null
   e?.let { println(it.length) }
   println("fin")
   ```
6. Crea la clase `Contacto` con nombre, teléfono **opcional** y email **opcional**, y un `toString()` que muestre «—» en los datos que falten. Crea una lista con al menos cuatro contactos y muestra cuántos no tienen ni teléfono ni email.
7. **Encuentra el error.** Este programa debería usar 10 € cuando el usuario escribe algo que no es un número, pero se cierra. Explica por qué y corrígelo.
   ```kotlin
   print("Precio: ")
   val precio = readln().toDouble() ?: 10.0
   println("Precio: $precio")
   ```

**Para profundizar:** investiga los *tipos plataforma* (`String!`) que aparecen al llamar a código Java desde Kotlin y por qué son un punto peligroso.
