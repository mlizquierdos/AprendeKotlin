# 08 · Funciones de extensión y funciones de alcance

> **Convención de los ejemplos:** los ejemplos completos incluyen su propia función `main()` y van seguidos de su **Salida**. Las funciones y clases se declaran **fuera** de `main()`.

## 8.1 Funciones de extensión

Permiten **añadir métodos a clases que no podemos modificar** (de la librería estándar, de terceros o de Java).

```kotlin
// Añadimos un método a String sin heredar ni modificar su código
fun String.esEmailValido(): Boolean =
    this.contains("@") && this.contains(".")

fun main() {
    println("celia@centro.es".esEmailValido())
    println("celia.centro.es".esEmailValido())
}
```

Salida:

```text
true
false
```

Dentro de una extensión, `this` es el objeto sobre el que se llama (aquí, el texto). Se puede omitir, igual que dentro de una clase.

```kotlin
import kotlin.math.round

fun Double.aEuros(): String = "%.2f €".format(this)

fun List<Double>.mediaRedondeada(): Double =
    if (isEmpty()) 0.0 else round(average() * 100) / 100     // dos decimales

fun main() {
    println(19.5.aEuros())
    println(listOf(7.25, 8.0, 6.1).mediaRedondeada())
    println(emptyList<Double>().mediaRedondeada())
}
```

Salida:

```text
19,50 €
7.12
0.0
```

> El separador decimal de `format` depende de la configuración regional del equipo: `aEuros()` puede devolver `19,50 €` o `19.50 €`. `mediaRedondeada()` no usa `format` y siempre muestra punto.

**Cómo funciona por dentro:** no se modifica la clase. El compilador genera una función
estática que recibe el objeto como primer parámetro. Por tanto:

- No pueden acceder a miembros `private` de la clase.
- Se resuelven **estáticamente**: no hay polimorfismo en extensiones (apartado 8.4).
- Si la clase ya tiene un método con el mismo nombre y parámetros, **gana el de la clase** y la extensión no se usa.

En Android se usan muchísimo para simplificar código repetitivo (fragmento, solo funciona dentro de un proyecto Android):

```kotlin
fun Context.toast(mensaje: String) =
    Toast.makeText(this, mensaje, Toast.LENGTH_SHORT).show()

// dentro de una Activity:  toast("Guardado")
```

## 8.2 Propiedades de extensión

```kotlin
val String.primeraLetra: Char?
    get() = this.firstOrNull()

fun main() {
    println("Kotlin".primeraLetra)
    println("".primeraLetra)
}
```

Salida:

```text
K
null
```

Una propiedad de extensión no puede guardar datos: siempre se calcula con un `get()`, porque no se puede añadir memoria a una clase que no es nuestra.

## 8.3 Funciones de alcance (*scope functions*)

Cinco funciones de la librería estándar que ejecutan un bloque sobre un objeto.
La diferencia está en **cómo se refiere al objeto** y en **qué devuelven**.

| Función | Referencia | Devuelve | Uso habitual |
|---|---|---|---|
| `let` | `it` | el resultado del bloque | trabajar con anulables, transformar |
| `run` | `this` | el resultado del bloque | calcular algo sobre un objeto |
| `with` | `this` | el resultado del bloque | varias operaciones sobre un objeto |
| `apply` | `this` | **el propio objeto** | configurar un objeto recién creado |
| `also` | `it` | **el propio objeto** | efectos secundarios (log, validación) |

Los ejemplos de este apartado usan esta clase:

```kotlin
class Alumno(val nombre: String) {
    var curso: Int = 1
    var activo: Boolean = false
    override fun toString() = "$nombre ($curso.º, activo: $activo)"
}
```

### `let`

```kotlin
fun dominio(email: String?): String =
    email?.let { it.substringAfter("@") } ?: "sin dominio"

fun main() {
    println(dominio("celia@centro.es"))
    println(dominio(null))
}
```

Salida:

```text
centro.es
sin dominio
```

### `apply` — configuración

```kotlin
class Alumno(val nombre: String) {
    var curso: Int = 1
    var activo: Boolean = false
    override fun toString() = "$nombre ($curso.º, activo: $activo)"
}

fun main() {
    val alumno = Alumno("Ana").apply {
        curso = 2          // "this" es el alumno: acceso directo a sus propiedades
        activo = true
    }
    // apply devuelve el alumno ya configurado
    println(alumno)
}
```

Salida:

```text
Ana (2.º, activo: true)
```

### `also` — efectos secundarios sin romper la cadena

```kotlin
fun main() {
    val notas = listOf(7.0, 5.5, 9.0)
        .also { println("Procesando ${it.size} notas") }
        .filter { it >= 6 }
    println(notas)
}
```

Salida:

```text
Procesando 3 notas
[7.0, 9.0]
```

### `run` y `with`

```kotlin
class Alumno(val nombre: String) {
    var curso: Int = 1
    var activo: Boolean = false
}

fun main() {
    val alumno = Alumno("Ana").apply { curso = 2 }

    val descripcion = alumno.run {
        "$nombre está en $curso.º"       // acceso directo, devuelve la cadena
    }
    println(descripcion)

    with(alumno) {
        println(nombre)
        println(curso)
    }
}
```

Salida:

```text
Ana está en 2.º
Ana
2
```

### Cómo elegir

1. ¿Quiero **devolver el mismo objeto**? → `apply` (configurar) o `also` (efecto secundario).
2. ¿Quiero **devolver otra cosa**? → `let` (con `it`) o `run` (con `this`).
3. ¿Estoy trabajando con un **anulable**? → `?.let { }`.

> No anidéis funciones de alcance: el código se vuelve ilegible enseguida.
> Si necesitáis tres niveles, extraed una función con nombre.

### Lo que devuelve cada una, en un solo ejemplo

Es la diferencia que más se confunde. El mismo bloque, con cada función:

```kotlin
fun main() {
    val texto = "Kotlin"

    val a = texto.let { it.length }     // devuelve el resultado del bloque
    val b = texto.run { length }        // devuelve el resultado del bloque
    val c = texto.also { it.length }    // devuelve el propio objeto
    val d = texto.apply { length }      // devuelve el propio objeto

    println("$a $b $c $d")
}
```

Salida:

```text
6 6 Kotlin Kotlin
```

## 8.4 Las extensiones no son polimórficas

Una extensión se elige según el **tipo declarado** de la variable, no según el objeto real. Con los métodos normales (unidad 07) ocurre al revés.

```kotlin
open class Figura
class Circulo : Figura()

fun Figura.nombre() = "figura"
fun Circulo.nombre() = "círculo"

fun main() {
    val f: Figura = Circulo()          // tipo declarado: Figura; objeto real: Circulo
    println(f.nombre())
    println(Circulo().nombre())
}
```

Salida:

```text
figura
círculo
```

**Consecuencia práctica:** si una operación debe comportarse distinto en cada subclase, tiene que ser un método `abstract` u `open` de la clase, no una extensión.

## 8.5 Errores típicos

| Error | Qué ocurre | Solución |
|---|---|---|
| Esperar polimorfismo de una extensión | Se ejecuta la versión del tipo declarado, no la del objeto real | Método `abstract`/`open` en la clase |
| Extensión con el mismo nombre que un método de la clase | La extensión nunca se ejecuta: gana el método de la clase | Elegir otro nombre |
| Querer usar un miembro `private` desde una extensión | No compila | La extensión solo ve lo público, o el método va dentro de la clase |
| Usar `apply` o `also` esperando el resultado del bloque | Se obtiene el objeto original | `let` o `run` |
| Usar `let` o `run` esperando el objeto | Se obtiene el valor de la última línea del bloque | `apply` o `also` |
| `let` sin `?.` sobre un anulable | El bloque se ejecuta también con `null` y `it` es anulable | `valor?.let { }` |
| Anidar varias funciones de alcance | `it` y `this` pasan a referirse a objetos distintos; el código es ilegible | Extraer una función con nombre |
| Dar por hecho que `"%.2f".format(...)` usa punto decimal | Depende de la configuración regional | Mostrar con `format` y calcular con `Double` |

## Ejercicios

1. Extensión `String.capitalizarPalabras()` que ponga en mayúscula la inicial de cada palabra.
2. Extensión `Int.esPrimo(): Boolean`.
3. Extensión `List<Alumno>.aprobados(): List<Alumno>`.
4. Reescribe usando `apply` un bloque que cree un objeto y le asigne cinco propiedades.
5. Explica con tus palabras la diferencia entre `let` y `also` mediante un ejemplo propio.
6. **Predice la salida.** Sin ejecutarlo, escribe qué imprimirá cada bloque. Después compruébalo.
   ```kotlin
   // A
   fun Int.doble() = this * 2
   println(5.doble().doble())

   // B
   val b = "hola".also { it.uppercase() }
   println(b)

   // C
   val c = "hola".let { it.uppercase() }
   println(c)

   // D
   val lista = mutableListOf(1, 2).apply { add(3) }
   println(lista)

   // E
   val n: String? = null
   println(n?.let { "Tiene ${it.length} letras" } ?: "Sin texto")
   ```

**Para profundizar:** investiga `takeIf` y `takeUnless`, y busca ejemplos de extensiones en la propia librería estándar (`kotlin.text`).
