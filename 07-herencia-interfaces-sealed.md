# 07 · Herencia, clases abstractas, interfaces, `object` y `sealed class`

## El mapa: seis herramientas, seis preguntas

Hasta ahora habéis creado clases sueltas. Un programa real tiene muchas clases **relacionadas entre sí**. Kotlin ofrece varias herramientas para ello, y cada una responde a una pregunta distinta:

| Si te preguntas… | La herramienta es… | Ejemplo |
|---|---|---|
| «Y es una idea **incompleta**: solo existen sus variantes concretas» | Clase abstracta (`abstract class`) | No existe una `Figura` a secas, sí un `Circulo` |
| «X **es un** tipo especial de Y, y Y **también** tiene sentido por sí misma» | Herencia de una clase abierta (`open class`) | Un `Alumno` *es una* `Persona`, y también hay personas que no son alumnos |
| «X **sabe hacer** esto» (da igual qué *es*) | Interfaz (`interface`) | Una `Cancion` y un `Video` *saben* reproducirse |
| «De esto solo debe haber **uno**» o «esto es de la clase, no de cada objeto» | `object` / `companion object` | La configuración de la app |
| «Solo existen **estas** opciones y ninguna más» | `enum class` / `sealed class` | Estado de una pantalla: cargando, con datos, con error |
| «X **tiene un** Y y le encarga parte del trabajo» | Delegación (`by`) | Un repositorio con registro que reenvía a otro repositorio |

**La pregunta que decide entre `abstract` y `open`:** *¿tiene sentido crear un objeto de la clase base?*
Si la respuesta es «no» (¿qué área tiene una figura cualquiera?, ¿qué IVA tiene una publicación cualquiera?), la clase es `abstract`. Es el caso más habitual cuando modeláis tipos de cosas, y por eso lo vemos primero.

Volved a esta tabla al terminar la unidad: si la entendéis, habéis entendido la unidad.

> **Convención de los ejemplos:** las clases se declaran **fuera** de `main`, a nivel de fichero (o en ficheros propios). Dentro de `main` solo se crean los objetos y se usan.

---

## 7.1 Todo es `final` por defecto

En Kotlin una clase **no se puede heredar** salvo que lo permitáis de forma explícita. Es una decisión deliberada: heredar de una clase que no se diseñó para ello genera errores sutiles.

```kotlin
class Producto(val nombre: String)
class Oferta(nombre: String) : Producto(nombre)   // ERROR: Producto es final
```

```text
error: this type is final, so it cannot be extended.
```

Para poder heredar, la clase base tiene que ser `abstract` (apartado 7.2) u `open` (apartado 7.6).

## 7.2 Clases abstractas

Pensad en `Figura`. Existen círculos, rectángulos, triángulos… pero **no existe «una figura a secas»**: ¿qué área tendría? Una clase abstracta es una plantilla **incompleta a propósito**:

- Define lo que **todas** las hijas comparten (propiedades y métodos con cuerpo).
- Deja **huecos**, los métodos `abstract`, que cada hija **está obligada** a rellenar.
- **No se puede instanciar**: solo se crean objetos de sus hijas.

```kotlin
abstract class Figura(val nombre: String) {
    abstract fun area(): Double                     // sin cuerpo: cada hija DEBE implementarlo

    fun describir() = "$nombre con área %.2f".format(area())   // común a todas
}

class Circulo(val radio: Double) : Figura("Círculo") {
    override fun area() = Math.PI * radio * radio
}

class Rectangulo(val base: Double, val altura: Double) : Figura("Rectángulo") {
    override fun area() = base * altura
}

fun main() {
    val figuras = listOf(Circulo(1.0), Rectangulo(2.0, 3.0))
    figuras.forEach { println(it.describir()) }     // cada una usa SU área: polimorfismo
}
```

Salida:

```text
Círculo con área 3,14
Rectángulo con área 6,00
```

> El separador decimal de `format` depende de la configuración regional del equipo: puede salir `3,14` o `3.14`.

`abstract` implica `open`: no hace falta escribir ambos.

### Lo que el compilador hace por vosotros

Esta es la gran ventaja frente a `open`: si os equivocáis, **no compila**.

```kotlin
class Triangulo(val base: Double, val altura: Double) : Figura("Triángulo")
// ERROR: falta implementar area()

val f = Figura("Genérica")
// ERROR: no se puede crear un objeto de una clase abstracta
```

```text
error: class 'Triangulo' is not abstract and does not implement abstract base class member 'area'.
error: cannot create an instance of an abstract class.
```

Comparad con lo que pasa si la base es `open` y el método tiene un cuerpo «de relleno»:

```kotlin
open class FiguraAbierta(val nombre: String) {
    open fun area(): Double = 0.0                    // valor inventado
}
class TrianguloAbierto(nombre: String) : FiguraAbierta(nombre)   // olvida sobrescribir area()

fun main() {
    println(TrianguloAbierto("Triángulo").area())
}
```

Salida:

```text
0.0
```

Compila sin avisos y da un resultado falso. **Si un método no tiene una respuesta razonable en la clase base, debe ser `abstract`.**

| Método en la clase base | La subclase… |
|---|---|
| `fun` (sin modificador) | No puede sobrescribirlo |
| `open fun` con cuerpo | Puede sobrescribirlo, pero **no está obligada** |
| `abstract fun` sin cuerpo | **Está obligada** a sobrescribirlo |

## 7.3 Constructores en una jerarquía

Cada dato viaja desde la llamada hasta la propiedad: la subclase lo **recibe** en su constructor y se lo **pasa** a la clase base entre los paréntesis que siguen a los dos puntos.

```kotlin
const val PRECIO_BASE = 100.0

abstract class Curso(
    val titulo: String,                      // con val: se guarda como propiedad
    precio: Double = PRECIO_BASE             // sin val: solo sirve para iniciar la propiedad
) {
    var precio = precio
        private set                          // se lee desde fuera, pero solo la clase lo cambia

    init {
        require(titulo.isNotBlank()) { "El título no puede estar vacío" }
        require(precio > 0) { "Precio no válido: $precio" }
    }

    abstract fun precioFinal(): Double
}

class CursoOnline(
    titulo: String,                          // 1. se recibe…
    val plataforma: String = "Moodle",       // dato propio con valor por defecto
    precio: Double = PRECIO_BASE             // el valor por defecto se repite AQUÍ
) : Curso(titulo, precio) {                  // 2. …y se pasa a la clase base
    override fun precioFinal() = precio * 0.85            // 15 % de descuento
}
```

Seguid el recorrido de `CursoOnline("Kotlin", precio = 80.0)`:

| Paso | Dónde | `titulo` | `precio` |
|---|---|---|---|
| 1 | Llamada | `"Kotlin"` | `80.0` (argumento con nombre) |
| 2 | Parámetros de `CursoOnline` | `"Kotlin"` | `80.0` |
| 3 | `: Curso(titulo, precio)` | se pasa | se pasa |
| 4 | Propiedades del objeto | `titulo = "Kotlin"` | `precio = 80.0` |

Con `CursoOnline("Kotlin")`, en el paso 2 `precio` toma el valor por defecto **del constructor de `CursoOnline`**.

### Tres reglas que conviene no olvidar

1. **El valor por defecto tiene que estar en el constructor que se llama.** Quien crea el objeto llama al constructor de la subclase; si ahí `precio` no tiene valor por defecto, es obligatorio indicarlo, aunque la base sí lo tenga.
2. **Un parámetro sin `val` ni `var` no es una propiedad.** Solo existe mientras se construye el objeto. Si quieres consultarlo después (`curso.plataforma`), decláralo con `val`.
3. **Si la subclase recibe un dato, tiene que pasárselo a la base.** Un parámetro que no aparece en `: Curso(...)` ni se guarda con `val` se pierde sin ningún aviso.

### Argumentos con nombre

Con varios valores por defecto, los argumentos con nombre permiten indicar solo los que cambian, en cualquier orden:

```kotlin
CursoOnline("Kotlin")                                    // todo por defecto
CursoOnline("Kotlin", precio = 80.0)                     // salta plataforma
CursoOnline(precio = 80.0, titulo = "Kotlin", plataforma = "Teams")
```

## 7.4 `toString()` en una jerarquía

Todas las clases heredan de `Any`, que ya trae un `toString()`. Sin sobrescribirlo, imprimir un objeto muestra el nombre de la clase y un código *hash*:

```kotlin
class SinToString(val titulo: String)

fun main() {
    println(SinToString("Kotlin"))
}
```

Salida:

```text
SinToString@34c45dca
```

(El número después de `@` cambia en cada ejecución.)

> Las `data class` (unidad 06) generan un `toString()` legible automáticamente, pero **una clase abstracta o abierta no puede ser `data class`**. En una jerarquía, el `toString()` lo escribís vosotros.

La clase base describe los datos comunes y cada hija **añade** los suyos reutilizando el de la base con `super.toString()`:

```kotlin
// En Curso
override fun toString() =
    "$titulo | %.2f € | final: %.2f €".format(precio, precioFinal())

// En CursoOnline
override fun toString() = "[Online]     ${super.toString()} | $plataforma"
```

`toString()` **devuelve** el texto; no lo imprime. Quien lo usa decide qué hacer con él: `println(curso)`, guardarlo en una variable o, en Android, mostrarlo en pantalla.

## 7.5 Ejemplo completo

Todo lo anterior junto: clase abstracta, constructores con valores por defecto, un dato opcional y `toString()` con `super`.

```kotlin
const val PRECIO_BASE = 100.0

abstract class Curso(
    val titulo: String,
    precio: Double = PRECIO_BASE
) {
    var precio = precio
        private set

    init {
        require(titulo.isNotBlank()) { "El título no puede estar vacío" }
        require(precio > 0) { "Precio no válido: $precio" }
    }

    abstract fun precioFinal(): Double

    fun subirPrecio(porcentaje: Double) {
        require(porcentaje >= 0) { "Porcentaje no válido: $porcentaje" }
        precio *= 1 + porcentaje / 100
    }

    override fun toString() =
        "$titulo | %.2f € | final: %.2f €".format(precio, precioFinal())
}

class CursoPresencial(
    titulo: String,
    val aula: String,
    val tutor: String? = null,               // opcional: puede no estar asignado
    precio: Double = PRECIO_BASE
) : Curso(titulo, precio) {
    override fun precioFinal() = precio + MATERIAL

    override fun toString() =
        "[Presencial] ${super.toString()} | Aula $aula | Tutor: ${tutor ?: "Sin asignar"}"

    companion object {
        private const val MATERIAL = 20.0    // constante con nombre, no un número suelto
    }
}

class CursoOnline(
    titulo: String,
    val plataforma: String = "Moodle",
    precio: Double = PRECIO_BASE
) : Curso(titulo, precio) {
    override fun precioFinal() = precio * (1 - DESCUENTO)

    override fun toString() = "[Online]     ${super.toString()} | $plataforma"

    companion object {
        private const val DESCUENTO = 0.15
    }
}

fun main() {
    val cursos = listOf(
        CursoPresencial("Android básico", "A12", "Marta", 150.0),
        CursoPresencial("Git y GitHub", "B03"),              // sin tutor ni precio
        CursoOnline("Kotlin desde Java"),                     // todo por defecto
        CursoOnline("Jetpack Compose", precio = 120.0)
    )

    cursos.forEach { println(it) }

    println("--- Tras subir un 10 %")
    cursos.forEach { it.subirPrecio(10.0) }
    cursos.forEach { println(it) }
}
```

Salida:

```text
[Presencial] Android básico | 150,00 € | final: 170,00 € | Aula A12 | Tutor: Marta
[Presencial] Git y GitHub | 100,00 € | final: 120,00 € | Aula B03 | Tutor: Sin asignar
[Online]     Kotlin desde Java | 100,00 € | final: 85,00 € | Moodle
[Online]     Jetpack Compose | 120,00 € | final: 102,00 € | Moodle
--- Tras subir un 10 %
[Presencial] Android básico | 165,00 € | final: 185,00 € | Aula A12 | Tutor: Marta
[Presencial] Git y GitHub | 110,00 € | final: 130,00 € | Aula B03 | Tutor: Sin asignar
[Online]     Kotlin desde Java | 110,00 € | final: 93,50 € | Moodle
[Online]     Jetpack Compose | 132,00 € | final: 112,20 € | Moodle
```

Fijaos en tres detalles:

- `cursos` es una `List<Curso>`: la lista no sabe qué tipo concreto hay en cada posición, pero cada objeto usa **su** `precioFinal()` y **su** `toString()`.
- `precio` tiene `private set`: desde `main` se puede leer, pero solo se modifica mediante `subirPrecio()`, que valida el dato.
- `subirPrecio` divide entre `100` un `Double`. Con un porcentaje de tipo `Int`, `10 / 100` daría `0` (división entera) y el precio no cambiaría.

## 7.6 `open`: cuando la clase base también tiene sentido

Hay casos en los que la clase base **sí** representa objetos reales. Una `Persona` existe por sí misma, y un `Alumno` es una persona con algo más. Entonces la base es `open` y se marca también `open` cada método que las hijas **pueden** redefinir.

```kotlin
open class Persona(val nombre: String) {
    open fun presentarse() = "Soy $nombre"
}

class Alumno(nombre: String, val curso: Int) : Persona(nombre) {
    override fun presentarse() = "${super.presentarse()} y estudio $curso.º de DAM"
}

fun main() {
    println(Persona("Lucía").presentarse())
    println(Alumno("Hugo", 2).presentarse())
}
```

Salida:

```text
Soy Lucía
Soy Hugo y estudio 2.º de DAM
```

- `override` es **obligatorio** (en Java, `@Override` es opcional, y en Kotlin no se usa).
- `super.presentarse()` llama a la versión de la clase base y permite **ampliarla** en lugar de repetirla.

Conexión con Android: `class MainActivity : ComponentActivity()` hereda de una clase de la librería, y `override fun onCreate(...)` redefine uno de sus métodos `open`. Es lo primero que veréis en Android Studio.

## 7.7 Errores típicos

Todos estos errores aparecieron en la prueba práctica de esta unidad.

| Error | Qué ocurre | Solución |
|---|---|---|
| Clase base `open` con un método `open` «de relleno» (`return precio`, `0.0`…) | Una subclase que olvide sobrescribirlo compila y da resultados falsos | Clase y método `abstract` |
| Valor por defecto solo en la clase base | `Hija("x")` no compila: `no value passed for parameter 'precio'` | Repetir el valor por defecto en el constructor de la hija, mejor con una constante con nombre |
| Parámetro de la hija que no se pasa a la base | El dato se pierde: todos los objetos quedan con el valor fijo de la base | Pasarlo en `: Base(titulo, precio)` |
| Parámetro sin `val` | `objeto.autor` → `unresolved reference 'autor'` | Declararlo `val autor: String` |
| Crear objetos de una clase **dentro** de esa misma clase | `StackOverflowError`: cada objeto crea otro sin fin | Los objetos de prueba se crean en `main` |
| No sobrescribir `toString()` | Se imprime `Libro@3fee733d` | `override fun toString()` en la base, ampliado con `super` en las hijas |
| Un método «mostrar» que imprime en vez de devolver | `println(objeto.mostrar())` imprime los datos y después `kotlin.Unit` | Devolver el texto (`toString()`) y que quien llama decida si lo imprime |
| Método de cálculo que **modifica** el objeto | Cada llamada vuelve a aplicar el IVA o el descuento | Los métodos que calculan devuelven un valor y no cambian propiedades |

## 7.8 Interfaces

La herencia responde a *«¿qué **es**?»*. Una interfaz responde a *«¿qué **sabe hacer**?»*: es un **contrato**, una lista de capacidades que una clase promete tener. Piensa en un carnet de conducir: no dice quién eres, dice qué puedes hacer.

Una `Cancion`, un `Video` y un `Podcast` no son «parientes», pero los tres **se pueden reproducir**. Esa capacidad común es la interfaz `Reproducible`.

```kotlin
interface Reproducible {
    val duracion: Int                         // propiedad abstracta: cada clase la proporciona
    fun reproducir(): String
    fun detener() = "Detenido"                // implementación por defecto
}

class Cancion(val titulo: String, override val duracion: Int) : Reproducible {
    override fun reproducir() = "Reproduciendo «$titulo» ($duracion s)"
}

class Video(val url: String, override val duracion: Int) : Reproducible {
    override fun reproducir() = "Reproduciendo vídeo $url"
}

fun main() {
    val cola: List<Reproducible> = listOf(Cancion("Mediterráneo", 210), Video("youtu.be/abc", 95))
    cola.forEach { println(it.reproducir()) }
    println("Duración total: ${cola.sumOf { it.duracion }} s")
    println(cola.first().detener())
}
```

Salida:

```text
Reproduciendo «Mediterráneo» (210 s)
Reproduciendo vídeo youtu.be/abc
Duración total: 305 s
Detenido
```

Una clase puede implementar **varias** interfaces pero heredar de **una sola** clase. Las interfaces no guardan estado: si las hijas comparten propiedades con valor, lo que necesitas es una clase abstracta.

## 7.9 `object`: el Singleton del lenguaje

A veces no necesitas una plantilla para **crear muchos** objetos, sino **un único objeto** que exista durante toda la aplicación: la configuración, un registro de mensajes, un gestor. En Kotlin lo declaras con `object` y el propio lenguaje crea esa única instancia por ti.

```kotlin
// Instancia única, creada de forma perezosa y segura frente a hilos.
object ConfiguracionApp {
    var tema: String = "claro"
    fun reiniciar() { tema = "claro" }
}

fun main() {
    ConfiguracionApp.tema = "oscuro"
    println(ConfiguracionApp.tema)
    ConfiguracionApp.reiniciar()
    println(ConfiguracionApp.tema)
}
```

Salida:

```text
oscuro
claro
```

Sustituye al patrón Singleton escrito a mano en Java (constructor privado + instancia estática).

### `companion object`: los miembros estáticos

Kotlin no tiene la palabra `static`. Lo que en Java sería estático (constantes, funciones que no dependen de un objeto concreto) se coloca en un **`companion object`**, dentro de la clase. Ya lo habéis visto en el ejemplo del apartado 7.5 para guardar el descuento y el coste del material.

```kotlin
class Alumno(val nombre: String) {
    companion object {
        const val NOTA_MINIMA = 5.0
        fun desdeCsv(linea: String) = Alumno(linea.split(";")[0])
    }
}

fun main() {
    println(Alumno.NOTA_MINIMA)
    val a = Alumno.desdeCsv("Ana;8.5")     // patrón factoría
    println(a.nombre)
}
```

Salida:

```text
5.0
Ana
```

## 7.10 `enum class`

Cuando una variable solo puede valer **una de unas pocas opciones conocidas** (días de la semana, estado de una corrección…), usar un `String` o un `Int` invita a errores: `"APROBADO"`, `"Aprobado"`, `"aprovado"`. Con un `enum`, el compilador **solo acepta las opciones que tú hayas definido**.

```kotlin
enum class Estado(val descripcion: String) {
    PENDIENTE("Sin corregir"),
    APROBADO("Superado"),
    SUSPENSO("No superado");

    fun esFinal() = this != PENDIENTE
}

fun main() {
    val e = Estado.APROBADO
    println("${e.descripcion} (final: ${e.esFinal()})")
}
```

Salida:

```text
Superado (final: true)
```

## 7.11 `sealed class`: jerarquías cerradas

Una clase sellada define un conjunto **cerrado y conocido** de subtipos. El compilador sabe cuáles son todos, así que puede comprobar que un `when` los cubre todos.

```kotlin
sealed class Resultado<out T> {
    data class Exito<T>(val datos: T) : Resultado<T>()
    data class Error(val mensaje: String) : Resultado<Nothing>()
    object Cargando : Resultado<Nothing>()
}

fun mostrar(resultado: Resultado<List<String>>) = when (resultado) {
    is Resultado.Exito -> println("Recibidos ${resultado.datos.size} elementos")
    is Resultado.Error -> println("Error: ${resultado.mensaje}")
    Resultado.Cargando -> println("Cargando...")
    // sin "else": si mañana añadimos un subtipo, el compilador nos avisa aquí
}

fun main() {
    mostrar(Resultado.Cargando)
    mostrar(Resultado.Exito(listOf("Ana", "Luis")))
    mostrar(Resultado.Error("Sin conexión"))
}
```

Salida:

```text
Cargando...
Recibidos 2 elementos
Error: Sin conexión
```

> **Este patrón es el estándar para modelar el estado de una pantalla en Android.**
> Lo reutilizaremos en la unidad 10: una pantalla está cargando, tiene datos o ha fallado.

## 7.12 Delegación

La herencia no siempre es la solución. Imaginad una secretaria que atiende llamadas: la mayoría las pasa al jefe tal cual, y en algunas añade algo (apunta la hora). La secretaria **no es** el jefe: **colabora** con él y le **reenvía** el trabajo.

Eso es delegar. En vez de heredar de otra clase, tu clase **recibe un objeto** y le reenvía lo que no quiere hacer por sí misma. En diseño de software se resume como **«prefiere composición a herencia»**: una relación «**tiene un**» (un coche *tiene un* motor) en lugar de «**es un**».

Hacerlo a mano obliga a escribir un método de reenvío por cada función de la interfaz, algo muy aburrido. Kotlin lo automatiza con la palabra `by`.

```kotlin
interface Repositorio {
    fun listar(): List<String>
    fun contar(): Int
}

class RepositorioMemoria : Repositorio {
    private val datos = listOf("a", "b", "c")
    override fun listar() = datos
    override fun contar() = datos.size
}

// "by" reenvía automáticamente a "origen" todo lo que no sobrescribamos
class RepositorioConLog(private val origen: Repositorio) : Repositorio by origen {
    override fun listar(): List<String> {
        println("[log] listar()")
        return origen.listar()
    }
    // contar() no se escribe: se delega tal cual
}

fun main() {
    val repo = RepositorioConLog(RepositorioMemoria())
    println(repo.listar())
    println(repo.contar())
}
```

Salida:

```text
[log] listar()
[a, b, c]
3
```

## Ejercicios

1. **Jerarquía de empleados.** Clase abstracta `Empleado` con `nombre`, `sueldoBase` (1200 € por defecto) y el método **abstracto** `calcularSueldo(): Double`. Subclases:
   - `Comercial`, con `ventas: Double`: sueldo base más un 5 % de las ventas.
   - `Tecnico`, con `horasExtra: Int = 0`: sueldo base más 15 € por hora extra.
   - Ambas con el sueldo base por defecto **también en su constructor**, `toString()` que reutilice el de `Empleado` con `super`, y un `responsable: String?` opcional en `Tecnico` que se muestre como «Sin asignar».

   Comprueba que `Tecnico("Iván")` compila y que `Empleado("X", 1000.0)` no.
2. **Predice la salida.** Sin ejecutarlo, escribe qué imprimirá cada bloque. Después compruébalo.
   ```kotlin
   // A: con la jerarquía del apartado 7.5
   println(CursoOnline("Kotlin").precioFinal())

   // B
   open class Base(val nombre: String) { open fun saludo() = "Hola" }
   class Hija(nombre: String) : Base(nombre)
   println(Hija("Ana").saludo())

   // C
   class Mascota(val nombre: String)
   println(Mascota("Toby"))

   // D
   fun mostrar(c: Curso) { println(c.titulo) }
   println(mostrar(CursoOnline("Kotlin")))
   ```
3. Interfaz `Almacenable` con `guardar()` y `cargar()`, implementada por dos clases distintas.
4. `object GestorUsuarios` que mantenga una lista y ofrezca alta, baja y consulta.
5. Modela con `sealed class` el resultado de un inicio de sesión (éxito, credenciales incorrectas, sin conexión) y escribe el `when` correspondiente.
6. Convierte a `enum class` un conjunto de constantes `static final` que hayas usado en Java.

**Para profundizar:** compara `sealed class` con `sealed interface` y averigua qué es el tipo `Nothing`. Investiga también por qué es arriesgado llamar a un método `open` o `abstract` desde el bloque `init` de la clase base.
