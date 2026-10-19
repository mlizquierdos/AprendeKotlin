# 06 · POO en Kotlin: clases y `data class`

> **Convención de los ejemplos:** las clases se declaran **fuera** de `main`, a nivel de fichero (o en ficheros propios). Dentro de `main` solo se crean los objetos y se usan.

## 6.1 Clase básica

```kotlin
// El constructor principal va en la propia cabecera de la clase.
// val/var en los parámetros crea directamente las propiedades.
class Alumno(val nombre: String, var nota: Double) {

    // bloque de inicialización: se ejecuta al construir el objeto
    init {
        require(nota in 0.0..10.0) { "Nota fuera de rango" }
    }

    fun estaAprobado(): Boolean = nota >= 5.0
}

fun main() {
    val a = Alumno("Ana", 8.0)     // sin "new"
    println(a.nombre)              // acceso por propiedad, no por getter
    a.nota = 9.0
    println(a.estaAprobado())
}
```

Salida:

```text
Ana
true
```

Ese código equivale a unas 30 líneas de Java con constructor, campos privados, *getters* y *setters*.

> 📄 `require` comprueba una condición y, si no se cumple, **lanza una excepción** (`IllegalArgumentException`).
> Todavía no hemos visto cómo funcionan en Kotlin: leed [Excepciones en Kotlin](excepciones.md)
> para entender qué hace exactamente y en qué se diferencia de `check` y `error`.

## 6.2 Parámetros del constructor: con `val` o sin `val`

En la cabecera de la clase se pueden escribir dos cosas distintas, y conviene no confundirlas:

| En la cabecera | Qué es | ¿Se puede usar después como `objeto.dato`? |
|---|---|---|
| `val nombre: String` | **Propiedad** de solo lectura | Sí |
| `var nota: Double` | **Propiedad** modificable | Sí, y también reasignarla |
| `curso: Int` (sin `val` ni `var`) | **Parámetro** del constructor | **No**: solo existe mientras se construye el objeto |

Un parámetro sin `val` solo se puede usar en el bloque `init` o para dar valor inicial a una propiedad:

```kotlin
class Matricula(val alumno: String, curso: Int) {
    val grupo = "${curso}º DAM"        // el parámetro curso se usa aquí...
}

fun main() {
    val m = Matricula("Ana", 2)
    println(m.grupo)
    // println(m.curso)                // ...pero no existe como propiedad
}
```

Salida:

```text
2º DAM
```

Si se descomenta la última línea:

```text
error: unresolved reference 'curso'.
```

**Regla práctica:** si vas a consultar el dato después de crear el objeto, declara el parámetro con `val` (o con `var` si debe poder cambiar).

## 6.3 Valores por defecto y datos opcionales

Los parámetros por defecto de la unidad 04 funcionan igual en los constructores, y sustituyen a la sobrecarga de constructores de Java:

```kotlin
class Libro(
    val titulo: String,
    val autor: String? = null,         // opcional: puede no conocerse
    val paginas: Int = 100,
    val precio: Double = 10.0
) {
    fun ficha() = "$titulo de ${autor ?: "autor desconocido"}, $paginas págs., $precio €"
}

fun main() {
    println(Libro("Platero y yo", "Juan Ramón Jiménez", 160, 12.5).ficha())
    println(Libro("Lazarillo de Tormes").ficha())                // todo por defecto
    println(Libro("El Cid", paginas = 250).ficha())               // argumento con nombre
}
```

Salida:

```text
Platero y yo de Juan Ramón Jiménez, 160 págs., 12.5 €
Lazarillo de Tormes de autor desconocido, 100 págs., 10.0 €
El Cid de autor desconocido, 250 págs., 10.0 €
```

Fijaos en el **dato opcional**:

- Se declara con tipo anulable y valor por defecto `null`: `val autor: String? = null`. Así no hace falta indicarlo al crear el objeto.
- Se guarda `null`, **no** un texto como `"Anónimo"` o `""`. El texto alternativo se decide al **mostrarlo**, con el operador Elvis de la unidad 03: `autor ?: "autor desconocido"`.
- Guardar `null` permite distinguir «no se conoce» de cualquier valor real, y preguntarlo con `if (libro.autor == null)`.

## 6.4 Propiedades con lógica

```kotlin
class Rectangulo(val base: Double, val altura: Double) {

    // propiedad calculada: no ocupa memoria, se evalúa al leerla
    val area: Double
        get() = base * altura

    // getter/setter personalizados
    var nombre: String = ""
        set(value) {
            field = value.trim().uppercase()   // "field" es el campo de respaldo
        }
}

fun main() {
    val r = Rectangulo(2.0, 3.0)
    r.nombre = "  salón "
    println("${r.nombre}: ${r.area} m²")
}
```

Salida:

```text
SALÓN: 6.0 m²
```

### Leer desde fuera, modificar solo desde dentro: `private set`

A menudo un dato debe poder **consultarse** desde cualquier sitio, pero solo debe **cambiar** a través de un método de la clase que lo valide. Para eso se declara la propiedad en el cuerpo con `private set`:

```kotlin
class Producto(val nombre: String, precio: Double) {   // precio sin val: es un parámetro

    var precio = precio                // la propiedad se inicia con el parámetro
        private set                    // desde fuera se lee, pero no se asigna

    fun subirPrecio(porcentaje: Double) {
        require(porcentaje >= 0) { "Porcentaje no válido: $porcentaje" }
        precio *= 1 + porcentaje / 100
    }
}

fun main() {
    val p = Producto("Teclado", 40.0)
    p.subirPrecio(10.0)
    println(p.precio)
    // p.precio = 0.0                  // ERROR: el setter es privado
}
```

Salida:

```text
44.0
```

> Ojo con los tipos: `porcentaje / 100` funciona porque `porcentaje` es `Double`. Con un `Int`, `10 / 100` daría `0` (división entera) y el precio no cambiaría.

## 6.5 `toString()`: cómo se describe un objeto

Todas las clases heredan de `Any`, que ya trae un método `toString()`. Es el que se usa cuando se imprime un objeto o se mete en una plantilla de cadena. Sin sobrescribirlo, muestra el nombre de la clase y un código *hash*:

```kotlin
class Mascota(val nombre: String, val especie: String)

fun main() {
    val m = Mascota("Toby", "perro")
    println(m)
    println("Mi mascota: $m")
}
```

Salida:

```text
Mascota@34c45dca
Mi mascota: Mascota@34c45dca
```

(El número después de `@` cambia en cada ejecución.)

Para que el objeto se describa con sus datos, se **sobrescribe** `toString()`:

```kotlin
class Mascota(val nombre: String, val especie: String, val chip: String? = null) {
    override fun toString() = "$nombre ($especie) · chip: ${chip ?: "sin registrar"}"
}

fun main() {
    val mascotas = listOf(Mascota("Toby", "perro", "941000024680"), Mascota("Misi", "gato"))
    mascotas.forEach { println(it) }
}
```

Salida:

```text
Toby (perro) · chip: 941000024680
Misi (gato) · chip: sin registrar
```

Tres ideas importantes:

- `override` es obligatorio: estás redefiniendo un método que la clase ya tiene.
- `toString()` **devuelve** el texto; **no** lo imprime. Quien lo usa decide qué hacer con él: `println(m)`, guardarlo en una variable o, en Android, mostrarlo en pantalla, donde `println` no se ve.
- No hagáis un método `mostrar()` que imprima por su cuenta: `println(m.mostrar())` imprimiría los datos y, después, `kotlin.Unit`, porque la función no devuelve nada.

> Las `data class` (apartado 6.8) generan este `toString()` automáticamente. En una **jerarquía de clases** (unidad 07) la clase base no puede ser `data class`, así que allí lo escribiréis vosotros y lo reutilizaréis con `super.toString()`.

## 6.6 Constructores secundarios

```kotlin
class Usuario(val nombre: String, val email: String) {
    constructor(nombre: String) : this(nombre, "$nombre@centro.es")
}

fun main() {
    println(Usuario("celia").email)
}
```

Salida:

```text
celia@centro.es
```

En la práctica se usan poco: los **parámetros por defecto** (apartado 6.3) los sustituyen casi siempre.

## 6.7 Visibilidad

| Modificador | Alcance |
|---|---|
| `public` | Por defecto. Visible desde cualquier sitio |
| `private` | Sólo dentro de la clase (o del fichero) |
| `protected` | Clase y subclases |
| `internal` | Dentro del mismo módulo |

```kotlin
class Cuenta(private var saldo: Double) {
    fun ingresar(cantidad: Double) {
        require(cantidad > 0) { "La cantidad debe ser positiva" }
        saldo += cantidad
    }
    fun consultar() = saldo
}

fun main() {
    val c = Cuenta(100.0)
    c.ingresar(50.0)
    println(c.consultar())
}
```

Salida:

```text
150.0
```

Aquí `saldo` es `private` del todo: ni siquiera se puede leer desde fuera, por eso hace falta `consultar()`. Si solo queréis impedir que se **modifique** desde fuera, usad `private set` (apartado 6.4).

## 6.8 `data class`

Para clases cuyo cometido es **transportar datos** (modelos, respuestas de API, entidades, resúmenes de resultados):

```kotlin
data class Alumno(
    val nombre: String,
    val nota: Double,
    val curso: Int = 1
)
```

El compilador genera automáticamente:

- `equals()` / `hashCode()` basados en las propiedades del constructor principal
- `toString()` legible: `Alumno(nombre=Ana, nota=8.5, curso=2)`
- `copy()` para crear copias modificadas
- `componentN()` para el *destructuring*

```kotlin
data class Alumno(val nombre: String, val nota: Double, val curso: Int = 1)

fun main() {
    val ana = Alumno("Ana", 8.5, 2)
    println(ana)                                 // toString() generado

    val anaRevisada = ana.copy(nota = 9.0)       // se crea un objeto nuevo; ana no cambia
    println(anaRevisada)

    println(ana == Alumno("Ana", 8.5, 2))        // true: compara CONTENIDO
    println(ana === Alumno("Ana", 8.5, 2))       // false: distinta referencia

    val (nombre, nota) = ana                     // destructuring
    println("$nombre: $nota")
}
```

Salida:

```text
Alumno(nombre=Ana, nota=8.5, curso=2)
Alumno(nombre=Ana, nota=9.0, curso=2)
true
false
Ana: 8.5
```

Una `data class` también es la forma natural de **devolver varios datos juntos** desde una función, por ejemplo un resumen calculado sobre una lista:

```kotlin
data class Estadisticas(val media: Double, val aprobados: Int, val suspensos: Int)

fun calcular(notas: List<Double>) = Estadisticas(
    media = notas.average(),
    aprobados = notas.count { it >= 5 },
    suspensos = notas.count { it < 5 }
)

fun main() {
    val (media, aprobados, suspensos) = calcular(listOf(7.5, 4.0, 9.0, 3.5))
    println("Media: $media · Aprobados: $aprobados · Suspensos: $suspensos")
}
```

Salida:

```text
Media: 6.0 · Aprobados: 2 · Suspensos: 2
```

> **Patrón clave para Android:** los modelos de datos son `data class` inmutables
> y los cambios se expresan con `.copy()`. Así el estado de la interfaz es siempre
> predecible: nadie puede modificar un objeto "por debajo".

> 📄 Antes de seguir, repasad [Diferencia entre val y var](val-vs-var.md): ahí se explica
> con detalle que `val` impide reasignar la referencia, pero no convierte el objeto en
> inmutable (por ejemplo, una `val lista = mutableListOf(...)` se puede seguir modificando).
> Esa distinción es la que hace necesario el patrón `copy()` que acabáis de ver.

## 6.9 Comparación rápida con Java

| Java | Kotlin |
|---|---|
| `new Alumno(...)` | `Alumno(...)` |
| `getNombre()` / `setNota()` | `alumno.nombre` / `alumno.nota = ...` |
| Varios constructores sobrecargados | Un constructor con valores por defecto |
| `@Override public String toString()` | `override fun toString() = ...` |
| POJO con 60 líneas | `data class` de 4 líneas |
| `record` (Java 16+) | `data class` (con `copy()` además) |

## 6.10 Errores típicos

| Error | Qué ocurre | Solución |
|---|---|---|
| Parámetro sin `val` que luego se consulta | `objeto.dato` → `unresolved reference` | Declararlo `val dato: Tipo` |
| No sobrescribir `toString()` en una clase normal | Se imprime `Mascota@34c45dca` | `override fun toString() = ...` |
| Un método `mostrar()` que imprime en vez de devolver | `println(objeto.mostrar())` añade `kotlin.Unit` | Devolver el texto con `toString()` |
| Guardar `"Anónimo"` o `""` para un dato que falta | No se distingue «sin dato» de un valor real | `String? = null` y `?:` al mostrarlo |
| Dato opcional sin valor por defecto (`val autor: String?`) | Hay que escribir `null` en cada llamada | `val autor: String? = null` |
| Propiedad `var` pública que solo debería cambiar con validación | Cualquier parte del programa puede poner un valor inválido | `var precio = precio` con `private set` y un método que valide |
| Precios o porcentajes como `Int` | `10 / 100` vale `0` | Usar `Double` |
| Crear objetos de una clase **dentro** de esa misma clase | `StackOverflowError`: cada objeto crea otro sin fin | Los objetos se crean en `main` |

## Ejercicios

1. Crea la clase `Libro` con título, autor **opcional**, año y disponibilidad. Añade un método `prestar()` que cambie el estado (que solo se pueda modificar con ese método) y un `toString()` que muestre «Autor desconocido» cuando no haya autor.
2. Convierte a `data class` un POJO `Producto` de tus prácticas de 1º y comprueba `equals`, `toString` y `copy`.
3. Clase `CuentaBancaria` con saldo privado, métodos `ingresar`/`retirar` y validación con `require`.
4. `data class Punto(val x: Int, val y: Int)` con una propiedad calculada `distanciaAlOrigen`.
5. **Predice la salida.** Sin ejecutarlo, escribe qué imprimirá cada bloque. Después compruébalo.
   ```kotlin
   // A
   class Coche(val marca: String, km: Int)
   val c = Coche("Seat", 1000)
   println(c.km)

   // B
   class Ciudad(val nombre: String)
   println(Ciudad("Ávila"))

   // C
   data class Ciudad2(val nombre: String)
   println(Ciudad2("Ávila"))

   // D
   class Socio(val nombre: String, val apodo: String? = null) {
       override fun toString() = "$nombre (${apodo ?: "sin apodo"})"
   }
   println(Socio("Marta"))
   ```
6. Función `resumen(precios: List<Double>)` que devuelva una `data class` con el total, el precio más alto y el número de productos. Muéstralo usando *destructuring*.

**Para profundizar:** investiga `lateinit`, `by lazy` y en qué situaciones de Android se usa cada uno.
