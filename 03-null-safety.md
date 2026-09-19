# 03 · Seguridad frente a nulos

Es **la** diferencia conceptual más importante respecto a Java. En Java cualquier referencia puede
ser `null` y el error aparece en ejecución. En Kotlin el compilador lo impide.

## 3.1 Tipos anulables

```kotlin
var nombre: String = "Celia"
// nombre = null              // ERROR de compilación

var apodo: String? = "Cel"    // el ? marca el tipo como anulable
apodo = null                  // correcto
```

`String` y `String?` son **tipos distintos**. Todo `String` es un `String?` válido, pero no al revés.

## 3.2 Acceso seguro `?.`

```kotlin
val apodo: String? = null
// println(apodo.length)      // ERROR: puede ser null
println(apodo?.length)        // imprime null, no lanza excepción
```

Se pueden encadenar:

```kotlin
val ciudad = alumno?.direccion?.ciudad   // null si algo de la cadena es null
```

## 3.3 Operador Elvis `?:`

Proporciona un valor alternativo cuando la expresión de la izquierda es `null`.

```kotlin
val longitud = apodo?.length ?: 0
val nombreMostrado = apodo ?: "Sin apodo"

// También sirve para salir de una función pronto
fun procesar(texto: String?) {
    val valor = texto ?: return            // si es null, se abandona la función
    println(valor.uppercase())
}
```

## 3.4 `!!`: la aserción no nula

```kotlin
val longitud = apodo!!.length   // lanza NullPointerException si apodo es null
```

> **Regla del módulo:** `!!` se considera un error de diseño salvo justificación escrita.
> Si lo usáis en una práctica, explicad en un comentario por qué es seguro.

## 3.5 `let` sobre anulables

```kotlin
val email: String? = leerEmail()

email?.let {
    // este bloque SOLO se ejecuta si email no es null
    // dentro, "it" es de tipo String (no anulable)
    println("Enviando correo a ${it.lowercase()}")
}
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
```

## 3.7 Conversiones seguras

```kotlin
val texto = "42a"
val numero: Int? = texto.toIntOrNull()      // null en vez de excepción
println(numero ?: -1)

val obj: Any = "hola"
val n = obj as? Int                         // cast seguro: null si no es Int
```

## 3.8 Resumen de operadores

| Operador | Significado | Cuándo usarlo |
|---|---|---|
| `?` | Declara tipo anulable | Cuando la ausencia de valor es un estado válido |
| `?.` | Llamada segura | Acceso a miembros de un anulable |
| `?:` | Elvis: valor por defecto | Dar alternativa o salir pronto |
| `?.let { }` | Ejecutar si no es null | Bloque de varias líneas |
| `as?` | Cast seguro | Conversión de tipo insegura |
| `!!` | Forzar no nulo | Casi nunca |

## Ejercicios

1. Función `longitudNombre(nombre: String?): Int` que devuelva 0 si es null, sin usar `if`.
2. Programa que pida un número por consola y muestre su cuadrado; si la entrada no es válida, debe mostrar `Entrada no válida` sin romperse.
3. Dada `data class Alumno(val nombre: String, val tutor: Tutor?)` y `data class Tutor(val email: String?)`, escribe una función que devuelva el email del tutor o `"desconocido"`.
4. Busca en tus prácticas de 1º un `NullPointerException` que te ocurriera y explica cómo lo habría evitado Kotlin.

**Para profundizar:** investiga los *tipos plataforma* (`String!`) que aparecen al llamar a código Java desde Kotlin y por qué son un punto peligroso.
