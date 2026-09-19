# 07 · Herencia, interfaces, `object` y `sealed class`

## 7.1 Herencia: todo es `final` por defecto

En Kotlin las clases **no se pueden heredar** salvo que se marquen como `open`.
Es una decisión deliberada: heredar de una clase no diseñada para ello genera errores sutiles.

```kotlin
open class Persona(val nombre: String) {
    open fun presentarse() = "Soy $nombre"
}

class Alumno(nombre: String, val curso: Int) : Persona(nombre) {
    override fun presentarse() = "${super.presentarse()} y estudio $curso º de DAM"
}
```

- `open` en la clase y en cada método que se quiera redefinir.
- `override` es **obligatorio** (en Java `@Override` es opcional).

## 7.2 Clases abstractas

```kotlin
abstract class Figura(val nombre: String) {
    abstract fun area(): Double                 // sin cuerpo: obliga a implementarla
    fun describir() = "$nombre con área ${area()}"
}

class Circulo(val radio: Double) : Figura("Círculo") {
    override fun area() = Math.PI * radio * radio
}
```

`abstract` implica `open`: no hace falta escribir ambos.

## 7.3 Interfaces

```kotlin
interface Reproducible {
    val duracion: Int                 // propiedad abstracta
    fun reproducir()
    fun detener() = println("Detenido")   // implementación por defecto
}

class Cancion(override val duracion: Int, val titulo: String) : Reproducible {
    override fun reproducir() = println("Reproduciendo $titulo")
}
```

Una clase puede implementar **varias** interfaces pero heredar de **una sola** clase.

## 7.4 `object`: el Singleton del lenguaje

```kotlin
// Instancia única, creada de forma perezosa y segura frente a hilos.
object ConfiguracionApp {
    var tema: String = "claro"
    fun reiniciar() { tema = "claro" }
}

ConfiguracionApp.tema = "oscuro"
```

Sustituye al patrón Singleton escrito a mano en Java (constructor privado + instancia estática).

### `companion object`: los miembros estáticos

```kotlin
class Alumno(val nombre: String) {
    companion object {
        const val NOTA_MINIMA = 5.0
        fun desdeCsv(linea: String) = Alumno(linea.split(";")[0])
    }
}

println(Alumno.NOTA_MINIMA)
val a = Alumno.desdeCsv("Ana;8.5")     // patrón factoría
```

## 7.5 `enum class`

```kotlin
enum class Estado(val descripcion: String) {
    PENDIENTE("Sin corregir"),
    APROBADO("Superado"),
    SUSPENSO("No superado");

    fun esFinal() = this != PENDIENTE
}

val e = Estado.APROBADO
println(e.descripcion)
```

## 7.6 `sealed class`: jerarquías cerradas

Una clase sellada define un conjunto **cerrado y conocido** de subtipos. El compilador
sabe cuáles son todos, así que puede comprobar que un `when` los cubre todos.

```kotlin
sealed class Resultado<out T> {
    data class Exito<T>(val datos: T) : Resultado<T>()
    data class Error(val mensaje: String) : Resultado<Nothing>()
    object Cargando : Resultado<Nothing>()
}

fun mostrar(resultado: Resultado<List<String>>) = when (resultado) {
    is Resultado.Exito   -> println("Recibidos ${resultado.datos.size} elementos")
    is Resultado.Error   -> println("Error: ${resultado.mensaje}")
    Resultado.Cargando   -> println("Cargando...")
    // sin "else": si mañana añadimos un subtipo, el compilador nos avisa aquí
}
```

> **Este patrón es el estándar para modelar el estado de una pantalla en Android.**
> Lo reutilizaremos en la unidad 10: una pantalla está cargando, tiene datos o ha fallado.

## 7.7 Delegación

```kotlin
interface Repositorio { fun listar(): List<String> }

class RepositorioMemoria : Repositorio {
    override fun listar() = listOf("a", "b")
}

// "by" delega automáticamente todos los métodos de la interfaz en el objeto indicado
class RepositorioConLog(private val origen: Repositorio) : Repositorio by origen
```

## Ejercicios

1. Jerarquía `Empleado` → `Comercial` / `Tecnico` con un método `calcularSueldo()` redefinido.
2. Interfaz `Almacenable` con `guardar()` y `cargar()`, implementada por dos clases distintas.
3. `object GestorUsuarios` que mantenga una lista y ofrezca alta, baja y consulta.
4. Modela con `sealed class` el resultado de un inicio de sesión (éxito, credenciales incorrectas, sin conexión) y escribe el `when` correspondiente.
5. Convierte a `enum class` un conjunto de constantes `static final` que hayas usado en Java.

**Para profundizar:** compara `sealed class` con `sealed interface` y averigua qué es el tipo `Nothing`.
