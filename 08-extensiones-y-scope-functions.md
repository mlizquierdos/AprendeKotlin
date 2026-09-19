# 08 · Funciones de extensión y funciones de alcance

## 8.1 Funciones de extensión

Permiten **añadir métodos a clases que no podemos modificar** (de la librería estándar, de terceros o de Java).

```kotlin
// Añadimos un método a String sin heredar ni modificar su código
fun String.esEmailValido(): Boolean =
    this.contains("@") && this.contains(".")

println("celia@centro.es".esEmailValido())   // true
```

```kotlin
fun Double.aEuros(): String = "%.2f €".format(this)
println(19.5.aEuros())        // 19,50 €

fun List<Double>.mediaRedondeada(): Double =
    if (isEmpty()) 0.0 else "%.2f".format(average()).replace(",", ".").toDouble()
```

**Cómo funciona por dentro:** no se modifica la clase. El compilador genera una función
estática que recibe el objeto como primer parámetro. Por tanto:

- No pueden acceder a miembros `private` de la clase.
- Se resuelven **estáticamente**: no hay polimorfismo en extensiones.

En Android se usan muchísimo para simplificar código repetitivo:

```kotlin
fun Context.toast(mensaje: String) =
    Toast.makeText(this, mensaje, Toast.LENGTH_SHORT).show()

// dentro de una Activity:  toast("Guardado")
```

## 8.2 Propiedades de extensión

```kotlin
val String.primeraLetra: Char?
    get() = this.firstOrNull()
```

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

### `let`

```kotlin
val email: String? = leerEmail()
val dominio = email?.let { it.substringAfter("@") } ?: "sin dominio"
```

### `apply` — configuración

```kotlin
val alumno = Alumno("Ana").apply {
    curso = 2          // "this" es el alumno: acceso directo a sus propiedades
    activo = true
}
// devuelve el alumno ya configurado
```

### `also` — efectos secundarios sin romper la cadena

```kotlin
val notas = listOf(7.0, 5.5, 9.0)
    .also { println("Procesando ${it.size} notas") }
    .filter { it >= 6 }
```

### `run` y `with`

```kotlin
val descripcion = alumno.run {
    "$nombre está en $curso º"       // acceso directo, devuelve la cadena
}

with(alumno) {
    println(nombre)
    println(curso)
}
```

### Cómo elegir

1. ¿Quiero **devolver el mismo objeto**? → `apply` (configurar) o `also` (efecto secundario).
2. ¿Quiero **devolver otra cosa**? → `let` (con `it`) o `run` (con `this`).
3. ¿Estoy trabajando con un **anulable**? → `?.let { }`.

> No anidéis funciones de alcance: el código se vuelve ilegible enseguida.
> Si necesitáis tres niveles, extraed una función con nombre.

## Ejercicios

1. Extensión `String.capitalizarPalabras()` que ponga en mayúscula la inicial de cada palabra.
2. Extensión `Int.esPrimo(): Boolean`.
3. Extensión `List<Alumno>.aprobados(): List<Alumno>`.
4. Reescribe usando `apply` un bloque que cree un objeto y le asigne cinco propiedades.
5. Explica con tus palabras la diferencia entre `let` y `also` mediante un ejemplo propio.

**Para profundizar:** investiga `takeIf` y `takeUnless`, y busca ejemplos de extensiones en la propia librería estándar (`kotlin.text`).
