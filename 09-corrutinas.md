# 09 · Corrutinas: programación asíncrona

## 9.1 El problema

En Android hay un **hilo principal** (*main thread* o hilo de UI) que dibuja la interfaz.
Si lo bloqueáis más de unos milisegundos, la aplicación se congela; si pasa de 5 segundos,
el sistema la mata con un error ANR (*Application Not Responding*).

Operaciones que **nunca** deben ir en el hilo principal: llamadas de red, acceso a base de datos,
lectura y escritura de ficheros, cálculos pesados.

La solución clásica en Java eran hilos y *callbacks* anidados. Kotlin ofrece **corrutinas**:
tareas ligeras que se suspenden sin bloquear el hilo, con un código que se lee de forma secuencial.

## 9.2 Dependencia

En `build.gradle.kts`:

```kotlin
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.8.1")
}
```

## 9.3 Funciones `suspend`

```kotlin
import kotlinx.coroutines.*

// Una función suspend puede pausarse y reanudarse sin bloquear el hilo.
// Sólo puede llamarse desde otra suspend o desde una corrutina.
suspend fun descargarDatos(): String {
    delay(2000)          // simula latencia; NO bloquea el hilo (a diferencia de Thread.sleep)
    return "Datos recibidos"
}

fun main() = runBlocking {          // runBlocking: puente main -> corrutinas (sólo para pruebas)
    println("Inicio")
    val datos = descargarDatos()     // se lee como código secuencial
    println(datos)
    println("Fin")
}
```

## 9.4 `launch` y `async`

```kotlin
fun main() = runBlocking {
    // launch: lanza una tarea y no devuelve resultado ("dispara y olvida")
    val trabajo = launch {
        delay(1000)
        println("Tarea en segundo plano terminada")
    }

    // async: lanza una tarea que SÍ devuelve un valor, recogido con await()
    val usuario  = async { descargarUsuario() }
    val ajustes  = async { descargarAjustes() }

    // ambas descargas se ejecutan en paralelo; aquí esperamos a las dos
    println("${usuario.await()} / ${ajustes.await()}")

    trabajo.join()
}
```

Si hicierais las dos descargas de forma secuencial tardaríais la suma de ambas;
con `async` tardáis lo que la más lenta.

## 9.5 `Dispatchers`: en qué hilo se ejecuta

| Dispatcher | Para qué |
|---|---|
| `Dispatchers.Main` | Actualizar la interfaz (sólo en Android) |
| `Dispatchers.IO` | Red, base de datos, ficheros |
| `Dispatchers.Default` | Cálculo intensivo (ordenaciones, procesado de imágenes) |

```kotlin
suspend fun cargarFichero(): String = withContext(Dispatchers.IO) {
    // withContext cambia de hilo y devuelve el resultado al terminar
    java.io.File("datos.txt").readText()
}
```

**Regla de oro:** la función `suspend` es responsable de elegir su propio dispatcher.
Quien la llama no debería preocuparse de en qué hilo se ejecuta.

## 9.6 Ámbito y cancelación

Una corrutina siempre vive dentro de un *scope*. Cuando el scope se cancela, todas sus
corrutinas hijas se cancelan (**concurrencia estructurada**): no quedan tareas huérfanas.

```kotlin
val scope = CoroutineScope(Dispatchers.Default)
val trabajo = scope.launch { /* ... */ }
trabajo.cancel()
```

En Android usaréis scopes ya preparados: `viewModelScope` (ligado al ViewModel) y
`lifecycleScope` (ligado a la Activity o al Fragment). Al destruirse la pantalla,
las corrutinas pendientes se cancelan solas.

## 9.7 Manejo de errores

```kotlin
suspend fun cargarSeguro(): Resultado<String> = try {
    Resultado.Exito(descargarDatos())
} catch (e: Exception) {
    Resultado.Error(e.message ?: "Error desconocido")
}
```

Fijaos en que reaprovechamos la `sealed class Resultado` de la unidad 07:
así la capa de interfaz decide qué mostrar sin capturar excepciones.

## Ejercicios

1. Simula con `delay` la descarga de tres recursos y mide el tiempo con `launch` frente a `async`.
2. Función `suspend` que lea un fichero de texto en `Dispatchers.IO` y devuelva el número de líneas.
3. Lanza una corrutina, cancélala a mitad de su ejecución y comprueba qué se imprime.
4. Escribe una función `suspend` que devuelva `Resultado<List<String>>` y trátala con un `when` exhaustivo.

**Para profundizar:** investiga `Flow` como flujo de valores asíncronos y su relación con `StateFlow` en el patrón MVVM.
