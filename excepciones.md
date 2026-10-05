# Excepciones en Kotlin

> **Material complementario.** Ya conocéis las excepciones de Java (1º de DAM): el mecanismo es el mismo.
> Este documento se centra en **lo que cambia en Kotlin**, que es poco pero importante.

## 1. Repaso rápido

Una excepción es un objeto que representa un error ocurrido **mientras el programa se ejecuta**. Interrumpe el flujo normal hasta que alguien la **captura**. Si nadie lo hace, el programa termina (en Android, la aplicación se cierra).

Kotlin usa **las mismas clases que Java**:

```text
Throwable
├── Error                          fallos graves de la JVM (no se capturan)
└── Exception
    ├── IOException                entrada/salida
    └── RuntimeException
        ├── IllegalArgumentException
        │   └── NumberFormatException
        ├── IllegalStateException
        ├── IndexOutOfBoundsException
        ├── ArithmeticException
        └── NullPointerException
```

Las más habituales (`Exception`, `IllegalArgumentException`, `IllegalStateException`, `NumberFormatException`…) se pueden usar **sin importar nada**. Las de entrada/salida, como `IOException`, sí necesitan `import java.io.IOException`.

## 2. Lanzar una excepción: `throw`

```kotlin
fun dividir(a: Int, b: Int): Int {
    if (b == 0) throw IllegalArgumentException("El divisor no puede ser 0")
    return a / b
}

println(dividir(10, 2))
```

Salida:

```text
5
```

Diferencia con Java: **no se escribe `new`**. Las excepciones se crean como cualquier otro objeto de Kotlin.

### `throw` es una expresión

En Kotlin, `throw` se puede usar donde se espera un valor. Combinado con el operador Elvis (unidad 03) permite exigir que un dato no sea nulo en una sola línea:

```kotlin
fun obtenerNombre(nombre: String?): String {
    return nombre ?: throw IllegalArgumentException("El nombre es obligatorio")
}

println(obtenerNombre("Ana"))
```

Si `nombre` es `null`, se lanza la excepción; si no, se devuelve su valor ya como `String` (no anulable).

## 3. Capturar excepciones: `try` / `catch` / `finally`

La estructura es idéntica a la de Java:

```kotlin
try {
    val n = "abc".toInt()
    println(n)
} catch (e: NumberFormatException) {
    println("No es un número: ${e.message}")
} finally {
    println("Esto se ejecuta siempre")
}
```

Salida:

```text
No es un número: For input string: "abc"
Esto se ejecuta siempre
```

- `e.message` contiene el mensaje de la excepción.
- `finally` se ejecuta **siempre**, se haya lanzado o no una excepción.

### Varios `catch`

```kotlin
fun leerElemento(lista: List<Int>, texto: String): Int {
    try {
        val indice = texto.toInt()
        return lista[indice]
    } catch (e: NumberFormatException) {
        println("El índice no es un número")
    } catch (e: IndexOutOfBoundsException) {
        println("El índice no existe en la lista")
    } catch (e: Exception) {
        println("Error inesperado: ${e.message}")
    }
    return -1
}

println(leerElemento(listOf(10, 20, 30), "1"))
println(leerElemento(listOf(10, 20, 30), "x"))
println(leerElemento(listOf(10, 20, 30), "9"))
```

Salida:

```text
20
El índice no es un número
-1
El índice no existe en la lista
-1
```

Los `catch` se revisan **de arriba abajo** y gana el primero que encaja. Por eso van **de lo más específico a lo más general**, y `Exception` (lo más general) siempre el último.

> **Ojo:** en Kotlin **no existe el multi-catch** de Java (`catch (A | B e)`). Si dos excepciones se tratan igual, se escriben dos bloques `catch` o se captura su clase común.

### ¿Qué ocurre si nadie captura la excepción?

El programa termina y se imprime la **traza de la pila** (*stack trace*). Sabérsela leer es fundamental, porque en Android es lo que vais a ver en el Logcat cuando la aplicación falle:

```text
5
Exception in thread "main" java.lang.IllegalArgumentException: El divisor no puede ser 0
	at MainKt.dividir(Main.kt:2)
	at MainKt.main(Main.kt:8)
	at MainKt.main(Main.kt)
```

Cómo leerla:

1. **Primera línea:** el tipo de excepción y su mensaje. Ya sabéis *qué* ha pasado.
2. **Líneas siguientes:** la pila de llamadas, de la más reciente a la más antigua.
3. **Buscad la primera línea que sea de vuestro código** (`Main.kt:2`): ahí se lanzó. La siguiente (`Main.kt:8`) indica quién la llamó.

## 4. `try` es una expresión

Igual que `if` y `when`, `try` **devuelve un valor**: el de la última expresión del bloque `try` o, si hubo excepción, del `catch`.

```kotlin
val texto = "42x"

val numero = try {
    texto.toInt()
} catch (e: NumberFormatException) {
    0
}
println(numero)
```

Salida:

```text
0
```

Para este caso concreto existe una alternativa más corta y sin excepciones, que ya conocéis de la unidad 03:

```kotlin
val numero2 = texto.toIntOrNull() ?: 0
println(numero2)
```

## 5. La gran diferencia con Java: no hay excepciones comprobadas

En Java hay dos tipos de excepciones:

- **Comprobadas** (*checked*, como `IOException`): el compilador te **obliga** a capturarlas o declararlas con `throws`.
- **No comprobadas** (*unchecked*, las `RuntimeException`): no hay obligación.

**Kotlin elimina esa distinción: ninguna excepción es comprobada.** El compilador nunca obliga a capturar nada ni existe la palabra `throws`.

```kotlin
import java.io.File

fun leerFichero(ruta: String): String {
    return File(ruta).readText()     // puede lanzar IOException, pero compila sin más
}
```

En Java, el equivalente exigiría esto:

```java
String leerFichero(String ruta) throws IOException {
    return Files.readString(Path.of(ruta));
}
```

¿Por qué se decidió así? La experiencia demostró que, con las comprobadas, mucho código acababa con bloques `catch` vacíos «para que compile», lo que es peor que no capturar nada.

**La consecuencia es una responsabilidad nueva para vosotros:** como el compilador no avisa, **tenéis que saber qué puede fallar** (leyendo la documentación) y decidir dónde capturarlo.

| Java | Kotlin |
|---|---|
| `new IllegalArgumentException("...")` | `IllegalArgumentException("...")` |
| `throws IOException` en la firma | No existe `throws` |
| El compilador obliga a tratar las comprobadas | El compilador **nunca** obliga |
| `catch (A \| B e)` | No existe: dos `catch` |
| `try (var r = ...) { }` (try-with-resources) | `.use { }` (apartado 8) |

## 6. Validar con `require`, `check` y `error`

Kotlin incluye funciones que lanzan la excepción adecuada en una sola línea. Ya habéis usado `require` en la unidad 06; esto es lo que hace en realidad:

| Función | Lanza | Se usa para validar… |
|---|---|---|
| `require(condición) { "mensaje" }` | `IllegalArgumentException` | los **argumentos** que recibe una función |
| `check(condición) { "mensaje" }` | `IllegalStateException` | el **estado** del objeto |
| `error("mensaje")` | `IllegalStateException` | situaciones que «no deberían pasar nunca» |
| `requireNotNull(x) { "mensaje" }` | `IllegalArgumentException` | que un argumento no sea `null` |

La regla para elegir: si el **fallo es de quien llama** (le ha pasado un dato incorrecto), `require`. Si el **fallo es del momento** (el objeto no está en un estado que permita la operación), `check`.

```kotlin
class Cuenta(private var saldo: Double, private val abierta: Boolean = true) {
    fun retirar(cantidad: Double) {
        require(cantidad > 0) { "La cantidad debe ser positiva" }   // argumento incorrecto
        check(abierta) { "La cuenta está cerrada" }                  // estado incorrecto
        saldo -= cantidad
    }
}

try {
    Cuenta(100.0).retirar(-5.0)
} catch (e: IllegalArgumentException) {
    println("Argumento incorrecto: ${e.message}")
}

try {
    Cuenta(100.0, abierta = false).retirar(10.0)
} catch (e: IllegalStateException) {
    println("Estado incorrecto: ${e.message}")
}
```

Salida:

```text
Argumento incorrecto: La cantidad debe ser positiva
Estado incorrecto: La cuenta está cerrada
```

`requireNotNull` tiene un bonus: tras comprobarlo, el compilador **ya sabe que el valor no es nulo**.

```kotlin
fun saludar(nombre: String?) {
    val limpio = requireNotNull(nombre) { "El nombre no puede ser null" }
    println("Hola, ${limpio.uppercase()}")     // aquí limpio es String, no String?
}

saludar("Ana")
```

Salida:

```text
Hola, ANA
```

## 7. Excepciones propias

Se crean heredando de `Exception`, como en Java, pero en muchas menos líneas:

```kotlin
class SaldoInsuficienteException(val saldo: Double, val pedido: Double) :
    Exception("Saldo insuficiente: tienes $saldo € y pides $pedido €")

class Monedero(private var saldo: Double) {
    fun retirar(cantidad: Double) {
        if (cantidad > saldo) throw SaldoInsuficienteException(saldo, cantidad)
        saldo -= cantidad
    }
}

try {
    Monedero(50.0).retirar(80.0)
} catch (e: SaldoInsuficienteException) {
    println(e.message)
    println("Faltan ${e.pedido - e.saldo} €")
}
```

Salida:

```text
Saldo insuficiente: tienes 50.0 € y pides 80.0 €
Faltan 30.0 €
```

Fijaos en que la excepción **lleva datos** (`saldo` y `pedido`) como cualquier otra clase, y quien la captura puede usarlos. Sin constructores, sin `super(...)` y sin *getters*: la cabecera lo resuelve todo.

## 8. `use`: el try-with-resources de Kotlin

Cuando se trabaja con ficheros (que conocéis de 1º), hay que **cerrar** el recurso aunque falle algo. En Java se usaba try-with-resources; en Kotlin, la función `use`:

```kotlin
import java.io.File

fun contarLineas(ruta: String): Int {
    return File(ruta).bufferedReader().use { lector ->
        lector.readLines().size
    }
}
```

`use` ejecuta el bloque y **cierra el recurso automáticamente al terminar**, tanto si todo va bien como si se lanza una excepción. No hace falta `finally` ni llamar a `close()`.

## 9. ¿Excepción o valor de retorno?

Las excepciones son para lo **excepcional**. Cuando un fallo es algo **esperable** (el usuario se equivoca, un dato no existe), es mejor devolver un valor que lo indique y evitar el `try/catch`:

| Situación | Mejor enfoque |
|---|---|
| El usuario escribe `"abc"` donde va un número | `toIntOrNull()` |
| Buscar un elemento que puede no estar | `firstOrNull()` o `getOrNull()` |
| Un argumento imposible (cantidad negativa) | `require` |
| Operar sobre un objeto en un estado no permitido | `check` |
| Falla la lectura de un fichero o la red | `try/catch` en la capa de datos, devolviendo un `Resultado.Error` (unidad 07) |

Es justo el criterio que se aplica en el proyecto final (unidad 11): las excepciones de entrada/salida se capturan **en un único sitio**, la capa de datos, y se convierten en un `Resultado.Error` que el resto del programa sabe tratar.

## 10. Excepciones en Android

Una excepción **no capturada** en el hilo principal **cierra la aplicación** (el típico «La aplicación se ha detenido»). La traza aparece en la pestaña **Logcat** de Android Studio, y se lee exactamente igual que la del apartado 3.

Las que más veréis:

| Excepción | Causa habitual |
|---|---|
| `NullPointerException` | Un `!!` sobre un valor `null`, o un `null` que llega desde código Java |
| `IllegalStateException` | Un `check`/`error()`, o una API de Android usada en un momento incorrecto |
| `IndexOutOfBoundsException` | Acceder a una posición que no existe en una lista |
| `NumberFormatException` | `toInt()` sobre un texto que no es un número |
| `UninitializedPropertyAccessException` | Usar una propiedad `lateinit` antes de asignarla |
| `NetworkOnMainThreadException` | Hacer una llamada de red en el hilo principal (unidad 09) |

> **Mala práctica habitual:** `catch (e: Exception) { }` con el bloque vacío. La excepción desaparece y nadie sabrá nunca qué falló. Como mínimo, dejad constancia en el registro:
>
> ```kotlin
> try {
>     // ...
> } catch (e: Exception) {
>     Log.e("MiApp", "Fallo al cargar los datos", e)   // solo disponible en Android
> }
> ```

## Ejercicios

1. Escribe un programa que pida un número por consola y lo convierta con `toInt()`, capturando `NumberFormatException`. Si la entrada no es válida, debe volver a pedirla. Después, reescríbelo usando `toIntOrNull()` y compara las dos versiones.
2. Crea una función `dividir(a: Int, b: Int): Int` que lance `IllegalArgumentException` si `b` es 0. Llámala desde `main` capturando la excepción y mostrando su mensaje.
3. Crea la clase `Ascensor` con una planta actual (de 0 a 9). El método `irAPlanta(n: Int)` debe validar con `require` que la planta existe y con `check` que las puertas no están abiertas.
4. Crea la excepción propia `StockInsuficienteException` (producto, disponible y pedido) y una clase `Almacen` que la lance al retirar más unidades de las que hay.
5. Escribe una función que cuente las líneas de un fichero usando `use` y pruébala con una ruta que no exista, capturando `FileNotFoundException` (hay que importarla de `java.io`).
6. Reflexiona por escrito (5 líneas): ¿qué ventajas e inconvenientes tiene que Kotlin no tenga excepciones comprobadas?

**Para profundizar:** investiga `runCatching` y la clase `Result`, la anotación `@Throws` (necesaria cuando Kotlin y Java se mezclan), y cómo se propagan las excepciones dentro de las corrutinas (`CoroutineExceptionHandler`).
