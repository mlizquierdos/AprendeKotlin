# 09 · Corrutinas: programación asíncrona

> **Convención de los ejemplos:** los ejemplos completos incluyen su propia función `main()` y van seguidos de su **Salida**. Las funciones y clases se declaran **fuera** de `main()`.
>
> En esta unidad todos los ejemplos completos empiezan con `import kotlinx.coroutines.*` y necesitan la dependencia del apartado 9.2. Las esperas se simulan con `delay`, así que los tiempos medidos pueden variar unos milisegundos en cada ejecución.

## 9.1 El problema

En Android hay un **hilo principal** (*main thread* o hilo de UI) que dibuja la interfaz.
Si lo bloqueáis más de unos milisegundos, la aplicación se congela; si pasa de 5 segundos,
el sistema la mata con un error ANR (*Application Not Responding*).

Operaciones que **nunca** deben ir en el hilo principal: llamadas de red, acceso a base de datos,
lectura y escritura de ficheros, cálculos pesados.

La solución clásica en Java eran hilos y *callbacks* anidados. Kotlin ofrece **corrutinas**:
tareas ligeras que se suspenden sin bloquear el hilo, con un código que se lee de forma secuencial.

| Concepto | Qué es |
|---|---|
| Función `suspend` | Función que puede pausarse sin bloquear el hilo |
| Corrutina | Tarea que ejecuta código `suspend`; se crea con `launch` o `async` |
| *Scope* (ámbito) | Lugar donde viven las corrutinas; al cancelarlo se cancelan todas |
| *Dispatcher* | Decide en qué hilo o grupo de hilos se ejecuta la corrutina |

## 9.2 Dependencia

Las corrutinas no forman parte de la librería estándar. En `build.gradle.kts`:

```kotlin
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.8.1")
}
```

Después de añadirla, pulsad **Sync** (el icono del elefante de Gradle) para que IntelliJ la descargue. Si no lo hacéis, `import kotlinx.coroutines.*` aparece en rojo:

```text
Main.kt:1:8: error: unresolved reference 'kotlinx'.
```

En Android Studio no hace falta añadirla a mano en la mayoría de plantillas: llega con las librerías de Jetpack.

## 9.3 Funciones `suspend`

```kotlin
import kotlinx.coroutines.*

// Una función suspend puede pausarse y reanudarse sin bloquear el hilo.
// Solo puede llamarse desde otra función suspend o desde una corrutina.
suspend fun descargarDatos(): String {
    delay(2000)          // simula la espera de la red; NO bloquea el hilo
    return "Datos recibidos"
}

fun main() = runBlocking {          // runBlocking: puente entre main y las corrutinas
    println("Inicio")
    val datos = descargarDatos()     // se lee como código secuencial
    println(datos)
    println("Fin")
}
```

Salida (la segunda línea aparece tras unos 2 segundos):

```text
Inicio
Datos recibidos
Fin
```

**`runBlocking` solo se usa en `main()` y en las pruebas.** Bloquea el hilo hasta que terminan todas las corrutinas de su interior, que es justo lo que se quiere evitar en una aplicación. En Android se usan los *scopes* del apartado 9.6.

### Llamar a una función `suspend` desde una función normal

```kotlin
fun main() {
    val datos = descargarDatos()     // main ya no está dentro de runBlocking
    println(datos)
}
```

```text
Main.kt:9:17: error: suspend function 'suspend fun descargarDatos(): String' should be called only from a coroutine or another suspend function.
```

Hay dos soluciones: convertir la función que llama en `suspend`, o llamar desde dentro de una corrutina (`runBlocking`, `launch`, `async`).

### `delay` frente a `Thread.sleep`

| | `delay(1000)` | `Thread.sleep(1000)` |
|---|---|---|
| Qué hace | **Suspende** la corrutina | **Bloquea** el hilo |
| El hilo, mientras tanto | Queda libre para otras corrutinas | No puede hacer nada más |
| Dónde se puede usar | Solo en código `suspend` | En cualquier sitio |
| En Android | Correcto | Congela la interfaz si está en el hilo principal |

Dentro de una corrutina, **nunca** `Thread.sleep`.

## 9.4 `launch` y `async`

### `launch`: lanzar una tarea sin resultado

`launch` crea una corrutina y **sigue adelante sin esperarla** («dispara y olvida»). Devuelve un `Job`, con el que se puede esperar (`join()`) o cancelar (`cancel()`).

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val trabajo = launch {
        delay(1000)
        println("Tarea en segundo plano terminada")
    }

    println("main sigue sin esperar")
    trabajo.join()                   // ahora sí: espera a que la tarea termine
    println("Fin")
}
```

Salida:

```text
main sigue sin esperar
Tarea en segundo plano terminada
Fin
```

El orden no es el del código: `launch` no espera, así que la segunda línea de `main` se imprime antes que la tarea.

### `async`: lanzar una tarea que devuelve un valor

`async` devuelve un `Deferred<T>`: una «promesa» de un valor que se recoge con `await()`. Lanzando dos `async` seguidos, las dos tareas avanzan **a la vez**.

```kotlin
import kotlinx.coroutines.*
import kotlin.system.measureTimeMillis

suspend fun descargarUsuario(): String {
    delay(1000)                      // simula 1 segundo de red
    return "Usuario: Ana"
}

suspend fun descargarAjustes(): String {
    delay(1500)                      // simula 1,5 segundos de red
    return "Tema: oscuro"
}

fun main() = runBlocking {
    // Una detrás de otra
    val secuencial = measureTimeMillis {
        val usuario = descargarUsuario()
        val ajustes = descargarAjustes()
        println("$usuario / $ajustes")
    }
    println("Secuencial: unos ${secuencial / 100 * 100} ms")

    // Las dos a la vez
    val paralelo = measureTimeMillis {
        val usuario = async { descargarUsuario() }
        val ajustes = async { descargarAjustes() }
        println("${usuario.await()} / ${ajustes.await()}")   // espera a las dos
    }
    println("Con async: unos ${paralelo / 100 * 100} ms")
}
```

Salida:

```text
Usuario: Ana / Tema: oscuro
Secuencial: unos 2500 ms
Usuario: Ana / Tema: oscuro
Con async: unos 1500 ms
```

De forma secuencial se tarda **la suma** de las dos descargas; con `async`, lo que tarda **la más lenta**. (`secuencial / 100 * 100` redondea a centenas con división entera, unidad 02, para que la salida no cambie por unos milisegundos.)

### `await()` justo después de `async` no adelanta nada

```kotlin
val usuario = async { descargarUsuario() }.await()    // espera aquí: 1000 ms
val ajustes = async { descargarAjustes() }.await()    // y luego aquí: 1500 ms
```

Cada `await()` espera a que termine su tarea **antes de lanzar la siguiente**, así que se tarda lo mismo que en la versión secuencial. Primero se lanzan todas las tareas y después se espera a los resultados.

| | `launch` | `async` |
|---|---|---|
| Devuelve | `Job` | `Deferred<T>` |
| Resultado | Ninguno | Un valor, con `await()` |
| Esperar a que termine | `join()` | `await()` |
| Uso típico | Guardar, enviar, registrar | Descargar o calcular algo que se va a usar |

## 9.5 `Dispatchers`: en qué hilo se ejecuta

| Dispatcher | Para qué |
|---|---|
| `Dispatchers.Main` | Actualizar la interfaz (solo en Android) |
| `Dispatchers.IO` | Red, base de datos, ficheros |
| `Dispatchers.Default` | Cálculo intensivo (ordenaciones, procesado de imágenes) |

`withContext` cambia de dispatcher para un bloque y devuelve su resultado al terminar. El fichero de este ejemplo lo crea el propio programa, para que funcione sin preparar nada:

```kotlin
import kotlinx.coroutines.*
import java.io.File

// La función elige su propio dispatcher: quien la llama no tiene que preocuparse
suspend fun leerFichero(ruta: String): String? = withContext(Dispatchers.IO) {
    println("Leyendo en: ${Thread.currentThread().name}")
    val fichero = File(ruta)
    if (fichero.exists()) fichero.readText() else null      // null si no existe
}

fun main() = runBlocking {
    File("datos.txt").writeText("Ana\nLuis\nMarta")
    println("Llamando desde: ${Thread.currentThread().name}")

    val texto = leerFichero("datos.txt")
    println(texto?.lines() ?: "No encontrado")

    val otro = leerFichero("no-existe.txt")
    println(otro?.lines() ?: "No encontrado")
}
```

Salida:

```text
Llamando desde: main
Leyendo en: DefaultDispatcher-worker-1
[Ana, Luis, Marta]
Leyendo en: DefaultDispatcher-worker-1
No encontrado
```

> El número del *worker* puede cambiar en cada ejecución. Lo importante es que la lectura **no** ocurre en el hilo `main`.

El fichero `datos.txt` se crea en la carpeta raíz del proyecto, la que contiene `build.gradle.kts`.

**Regla de oro:** la función `suspend` es responsable de elegir su propio dispatcher.
Quien la llama no debería preocuparse de en qué hilo se ejecuta.

## 9.6 Ámbito y cancelación

Una corrutina siempre vive dentro de un *scope*. Cuando se cancela una corrutina, se cancelan también todas sus hijas (**concurrencia estructurada**): no quedan tareas huérfanas.

### Cancelar una corrutina

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val trabajo = launch {
        repeat(10) { i ->
            println("Paso $i")
            delay(300)
        }
        println("Esta línea no se llega a imprimir")
    }

    delay(1000)                       // dejamos que avance 1 segundo
    trabajo.cancel()
    trabajo.join()                    // esperamos a que la cancelación se complete
    println("¿Cancelada? ${trabajo.isCancelled}")
}
```

Salida:

```text
Paso 0
Paso 1
Paso 2
Paso 3
¿Cancelada? true
```

Los pasos se imprimen a los 0, 300, 600 y 900 ms. A los 1000 ms la corrutina está suspendida en `delay`, y ahí es donde se detiene: **la cancelación actúa en los puntos de suspensión**.

### Cancelar el padre cancela a las hijas

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val padre = launch {
        launch { repeat(10) { println("Hija 1: $it"); delay(400) } }
        launch { repeat(10) { println("Hija 2: $it"); delay(400) } }
    }

    delay(1000)
    padre.cancel()                    // no hace falta cancelar las hijas una a una
    padre.join()
    println("Todas canceladas")
}
```

Salida:

```text
Hija 1: 0
Hija 2: 0
Hija 1: 1
Hija 2: 1
Hija 1: 2
Hija 2: 2
Todas canceladas
```

### Crear un *scope* propio

Fuera de `runBlocking`, se crea un *scope* indicando su dispatcher (fragmento):

```kotlin
val scope = CoroutineScope(Dispatchers.Default)
val trabajo = scope.launch { /* ... */ }
trabajo.cancel()          // cancela esta tarea
scope.cancel()            // cancela todas las tareas del scope
```

En Android usaréis *scopes* ya preparados: `viewModelScope` (ligado al ViewModel) y
`lifecycleScope` (ligado a la Activity o al Fragment). Al destruirse la pantalla,
las corrutinas pendientes se cancelan solas.

**`GlobalScope` no se usa.** Sus corrutinas no pertenecen a ninguna pantalla ni a ningún padre: nadie las cancela y siguen ejecutándose aunque el usuario haya salido.

## 9.7 Manejo de errores

Una función `suspend` puede lanzar excepciones igual que una normal. Para que la capa de interfaz no tenga que capturarlas, se devuelve un `Resultado` como el de la unidad 07. Este ejemplo lo define de nuevo para que funcione por sí solo:

```kotlin
import kotlinx.coroutines.*

sealed class Resultado<out T> {
    data class Exito<T>(val datos: T) : Resultado<T>()
    data class Error(val mensaje: String) : Resultado<Nothing>()
}

suspend fun descargarNotas(conexion: Boolean): List<Double> {
    delay(500)
    if (!conexion) throw IllegalStateException("Sin conexión")
    return listOf(7.5, 4.0, 9.25)
}

suspend fun cargarNotas(conexion: Boolean): Resultado<List<Double>> = try {
    Resultado.Exito(descargarNotas(conexion))
} catch (e: CancellationException) {
    throw e                          // la cancelación no es un error: se deja pasar
} catch (e: Exception) {
    Resultado.Error(e.message ?: "Error desconocido")
}

fun mostrar(resultado: Resultado<List<Double>>) = when (resultado) {
    is Resultado.Exito -> println("Recibidas ${resultado.datos.size} notas: ${resultado.datos}")
    is Resultado.Error -> println("Error: ${resultado.mensaje}")
}

fun main() = runBlocking {
    mostrar(cargarNotas(conexion = true))
    mostrar(cargarNotas(conexion = false))
    println("El programa sigue")
}
```

Salida:

```text
Recibidas 3 notas: [7.5, 4.0, 9.25]
Error: Sin conexión
El programa sigue
```

Tres detalles:

- El `when` no necesita `else`: es exhaustivo porque `Resultado` es `sealed` (unidad 07).
- **`CancellationException` se relanza.** Es la excepción con la que se cancela una corrutina (apartado 9.6). Un `catch (e: Exception)` también la captura, así que, si no se relanza, la cancelación se trata como un error más y la corrutina no se detiene cuando debe.
- El `catch` nunca va vacío: siempre se devuelve un `Resultado.Error` con un mensaje.

## 9.8 Errores típicos

| Error | Qué ocurre | Solución |
|---|---|---|
| Falta la dependencia o no se sincroniza Gradle | `unresolved reference 'kotlinx'` | Añadirla (9.2) y pulsar **Sync** |
| Llamar a una función `suspend` desde una función normal | No compila | Hacerla `suspend` o llamarla desde una corrutina |
| `Thread.sleep` dentro de una corrutina | Bloquea el hilo; en Android congela la interfaz | `delay` |
| `runBlocking` en una aplicación Android | Bloquea el hilo principal: ANR | `viewModelScope.launch` o `lifecycleScope.launch` |
| `GlobalScope.launch` | Tareas que nadie cancela | Un *scope* ligado a la pantalla o al ViewModel |
| `async { }.await()` en la misma línea, una tras otra | Se ejecutan en secuencia: no se gana tiempo | Lanzar todos los `async` y después hacer los `await()` |
| `async` sin `await()` | El resultado no se usa y los errores pueden pasar inadvertidos | Si no hay resultado, `launch`; si lo hay, `await()` |
| Red, ficheros o base de datos sin `withContext(Dispatchers.IO)` | Se ejecuta en el hilo de quien llama | La función `suspend` elige su dispatcher |
| Esperar que `launch` termine antes de la línea siguiente | El orden de la salida no es el del código | `join()` si hay que esperarla |
| `catch (e: Exception)` sin relanzar `CancellationException` | La corrutina no se cancela correctamente | `catch (e: CancellationException) { throw e }` antes |
| `catch` vacío | El error desaparece sin aviso | Devolver `Resultado.Error` o mostrar un mensaje |

## Ejercicios

1. Simula con `delay` la descarga de tres recursos (800, 1200 y 500 ms). Mide con `measureTimeMillis` cuánto se tarda de forma secuencial y con `async`, y explica la diferencia.
2. Función `suspend contarLineas(ruta: String): Int?` que lea un fichero en `Dispatchers.IO` y devuelva su número de líneas, o `null` si el fichero no existe.
3. Lanza una corrutina que imprima un contador cada 200 ms, cancélala a los 700 ms y comprueba qué se imprime. Antes de ejecutarlo, escribe cuántas líneas esperas.
4. Escribe una función `suspend` que devuelva `Resultado<List<String>>` y trátala con un `when` exhaustivo. Prueba el caso de éxito y el de error.
5. **Predice la salida.** Sin ejecutarlo, escribe qué imprimirá cada programa y en qué orden. Después compruébalo.
   ```kotlin
   // A
   fun main() = runBlocking {
       launch { println("1") }
       println("2")
   }

   // B
   fun main() = runBlocking {
       launch { delay(100); println("1") }
       launch { println("2") }
       println("3")
   }

   // C
   fun main() = runBlocking {
       val a = async { delay(300); 10 }
       val b = async { delay(100); 5 }
       println(a.await() + b.await())
   }

   // D
   fun main() = runBlocking {
       val t = launch { repeat(5) { println("Paso $it"); delay(250) } }
       delay(600)
       t.cancel()
       println("Fin")
   }
   ```
6. **Encuentra el error.** Este programa quiere hacer las dos descargas a la vez, pero tarda 2,5 segundos. Explica por qué y corrígelo.
   ```kotlin
   fun main() = runBlocking {
       val tiempo = measureTimeMillis {
           val usuario = async { descargarUsuario() }.await()
           val ajustes = async { descargarAjustes() }.await()
           println("$usuario / $ajustes")
       }
       println("$tiempo ms")
   }
   ```

**Para profundizar:** investiga `Flow` como flujo de valores asíncronos y su relación con `StateFlow` en el patrón MVVM.
