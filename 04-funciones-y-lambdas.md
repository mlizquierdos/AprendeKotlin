# 04 · Funciones y lambdas

> **Convención de los ejemplos:** los ejemplos completos incluyen su propia función `main()` y van seguidos de su **Salida**. Las funciones y clases se declaran **fuera** de `main()`.

## 4.1 Declaración

```kotlin
fun sumar(a: Int, b: Int): Int {
    return a + b
}

// Cuerpo de expresión: el tipo de retorno se infiere
fun sumar2(a: Int, b: Int) = a + b

// Sin retorno: el tipo es Unit (equivalente a void), y puede omitirse
fun saludar(nombre: String) {
    println("Hola, $nombre")
}

fun main() {
    println(sumar(2, 3))
    println(sumar2(4, 5))
    saludar("Ana")
}
```

Salida:

```text
5
9
Hola, Ana
```

## 4.2 Devolver o imprimir: no es lo mismo

Una función puede hacer dos cosas muy distintas con un resultado:

- **Devolverlo** (`return` o cuerpo de expresión): quien la llama recibe el valor y decide qué hacer con él (imprimirlo, guardarlo, usarlo en otro cálculo, mostrarlo en una pantalla de Android…).
- **Imprimirlo** (`println`): el valor aparece en la consola y se pierde. La función no devuelve nada útil: su tipo de retorno es `Unit`.

```kotlin
fun precioFinal(precio: Double): Double = precio * 1.21     // DEVUELVE

fun mostrarPrecio(precio: Double) {                          // IMPRIME (devuelve Unit)
    println(precio * 1.21)
}

fun main() {
    val total = precioFinal(10.0) + precioFinal(20.0)       // se puede seguir calculando
    println("Total: %.2f €".format(total))

    mostrarPrecio(10.0)                                       // solo imprime
    // val total2 = mostrarPrecio(10.0) + mostrarPrecio(20.0)   // ERROR: Unit no se puede sumar
}
```

Salida:

```text
Total: 36,30 €
12.1
```

> El separador decimal de `format` depende de la configuración regional del equipo: puede salir `36,30` o `36.30`.

### El caso `kotlin.Unit`

Si se imprime el resultado de una función que ya imprime, aparece un texto extraño:

```kotlin
fun mostrarPrecio(precio: Double) {
    println(precio * 1.21)
}

fun main() {
    println(mostrarPrecio(10.0))
}
```

Salida:

```text
12.1
kotlin.Unit
```

Hay dos impresiones: la de **dentro** de la función (el precio) y la de **fuera**, que imprime lo que devuelve la función. Como no devuelve nada, ese «nada» es `Unit` y se imprime como `kotlin.Unit`.

### Regla del módulo

| La función… | Debe… |
|---|---|
| Calcula algo (un precio, una media, un texto que describe un objeto) | **Devolver** el valor, sin `println` dentro |
| Muestra información al usuario | Estar en `main` o en una función de interfaz, y **llamar** a las funciones que calculan |

En Android esto es obligatorio: en una aplicación no hay consola, y un `println` dentro de una clase del modelo no se ve nunca. Lo que se muestra en pantalla es el valor que **devuelven** las funciones.

> Para describir un objeto como texto existe un método pensado justo para eso: `toString()`, que **devuelve** el texto (unidad 06).

### Llamar a una función y descartar su resultado

El error contrario también es frecuente: llamar a una función que devuelve algo y no usar el resultado. Kotlin no avisa; el valor simplemente se pierde.

```kotlin
fun aplicarDescuento(precio: Double) = precio * 0.9

fun main() {
    val precio = 50.0
    aplicarDescuento(precio)                    // el resultado se calcula y se tira
    println(precio)                             // precio sigue valiendo 50.0

    val rebajado = aplicarDescuento(precio)     // ahora sí se guarda
    println(rebajado)
}
```

Salida:

```text
50.0
45.0
```

`aplicarDescuento` no modifica `precio`: **calcula** un valor nuevo y lo devuelve. Lo mismo ocurre con los operadores de colecciones de la unidad 05 (`map`, `filter`, `sortedBy`).

## 4.3 Cuidado con `= { }`

En una función de cuerpo de expresión, lo que va después del `=` **es** el valor devuelto. Si ese valor va entre llaves, no es un bloque de código: es una **lambda** (apartado 4.5), y la función devuelve la lambda en lugar del resultado.

```kotlin
fun conIvaMal(precio: Double) = { precio * 1.21 }      // devuelve una FUNCIÓN
fun conIvaBien(precio: Double) = precio * 1.21          // devuelve un Double

fun main() {
    println(conIvaMal(10.0))
    println(conIvaBien(10.0))
}
```

Salida (el texto de la primera línea varía):

```text
MainKt$$Lambda/0x00007f8104001000@4b67cf4d
12.1
```

Si se indica el tipo de retorno, el compilador detecta el error:

```kotlin
fun doble(n: Int): Int = { n * 2 }
```

```text
error: return type mismatch: expected 'kotlin.Int', actual 'kotlin.Function0<kotlin.Int>'.
```

| Forma | Qué devuelve |
|---|---|
| `fun f(): Double { return x * 2 }` | El valor (cuerpo de bloque, con `return`) |
| `fun f() = x * 2` | El valor (cuerpo de expresión) |
| `fun f() = { x * 2 }` | **Una función**: casi nunca es lo que se quiere |

## 4.4 Parámetros por defecto y con nombre

```kotlin
fun crearUsuario(
    nombre: String,
    activo: Boolean = true,
    rol: String = "alumno"
) = "$nombre ($rol) activo=$activo"

fun main() {
    println(crearUsuario("Ana"))                       // usa los valores por defecto
    println(crearUsuario("Luis", rol = "profesor"))    // argumento con nombre: se salta 'activo'
}
```

Salida:

```text
Ana (alumno) activo=true
Luis (profesor) activo=true
```

Esto elimina la necesidad de la **sobrecarga de constructores** típica de Java. En la unidad 06 veréis que funciona igual en los constructores de las clases.

## 4.5 Número variable de argumentos

```kotlin
fun media(vararg notas: Double): Double =
    if (notas.isEmpty()) 0.0 else notas.average()

fun main() {
    println(media(7.0, 8.5, 6.0))
    println(media())
}
```

Salida:

```text
7.166666666666667
0.0
```

## 4.6 Lambdas

Una lambda es una función sin nombre que se puede guardar en una variable o pasar como argumento.

```kotlin
fun main() {
    val doble: (Int) -> Int = { n -> n * 2 }
    println(doble(5))

    // Con un solo parámetro se puede usar el nombre implícito "it"
    val triple: (Int) -> Int = { it * 3 }
    println(triple(5))
}
```

Salida:

```text
10
15
```

Sintaxis del tipo función: `(TiposDeEntrada) -> TipoDeSalida`.

La **última expresión** de una lambda es su valor de retorno; no se escribe `return`. Por eso, en el apartado 4.3, `{ precio * 1.21 }` es una función que, **cuando se llama**, devuelve el precio con IVA.

## 4.7 Funciones de orden superior

Son funciones que **reciben o devuelven** otras funciones.

```kotlin
fun operar(a: Int, b: Int, operacion: (Int, Int) -> Int): Int {
    return operacion(a, b)
}

fun main() {
    println(operar(6, 3) { x, y -> x + y })
    println(operar(6, 3) { x, y -> x * y })
}
```

Salida:

```text
9
18
```

Convención: si la lambda es el **último** parámetro, se saca fuera del paréntesis.

Esta convención es la base de la sintaxis de Jetpack Compose que veréis en la unidad 10 (fragmento, no se ejecuta en consola):

```kotlin
Button(onClick = { contador++ }) {
    Text("Pulsa aquí")     // esta lambda final es el contenido del botón
}
```

## 4.8 Referencias a funciones

```kotlin
fun esPar(n: Int) = n % 2 == 0

fun main() {
    val numeros = listOf(1, 2, 3, 4, 5, 6)
    println(numeros.filter(::esPar))
}
```

Salida:

```text
[2, 4, 6]
```

## 4.9 Funciones locales

```kotlin
fun validarAlumno(nombre: String, email: String) {
    // función anidada: sólo tiene sentido dentro de esta validación
    fun validar(campo: String, valor: String) {
        if (valor.isBlank()) throw IllegalArgumentException("$campo vacío")
    }
    validar("nombre", nombre)
    validar("email", email)
}

fun main() {
    validarAlumno("Ana", "ana@centro.es")
    println("Datos correctos")
}
```

Salida:

```text
Datos correctos
```

## 4.10 Una función reutilizable: leer números sin que el programa se cierre

Casi todos los programas de consola piden números al usuario. Si se convierten con `toInt()` o `toDouble()`, una letra cierra el programa con `NumberFormatException`. Es mejor escribir **una vez** una función que pregunte hasta recibir un valor válido, y usarla en todo el programa:

```kotlin
// Pide un número decimal hasta que el usuario escriba uno válido.
// Acepta coma o punto como separador decimal.
fun leerDouble(mensaje: String, minimo: Double = 0.0): Double {
    while (true) {
        print(mensaje)
        val valor = readlnOrNull()?.trim()?.replace(',', '.')?.toDoubleOrNull()
        if (valor != null && valor >= minimo) return valor        // DEVUELVE el número
        println("  Introduce un número mayor o igual que $minimo.")
    }
}

fun leerTexto(mensaje: String): String {
    while (true) {
        print(mensaje)
        val texto = readlnOrNull()?.trim().orEmpty()
        if (texto.isNotEmpty()) return texto
        println("  No puede estar vacío.")
    }
}

fun main() {
    val nombre = leerTexto("Producto: ")
    val precio = leerDouble("Precio: ")
    println("$nombre cuesta %.2f €".format(precio))
}
```

Ejemplo de ejecución (en negrita lo que escribe el usuario):

<pre>
Producto: <b>Teclado</b>
Precio: <b>abc</b>
  Introduce un número mayor o igual que 0.0.
Precio: <b>39,90</b>
Teclado cuesta 39,90 €
</pre>

Fijaos en que `leerDouble` y `leerTexto` **devuelven** el dato leído; es `main` quien decide qué hacer con él. `readlnOrNull()` y `toDoubleOrNull()` se explican en las unidades 01 y 03.

## 4.11 Errores típicos

| Error | Qué ocurre | Solución |
|---|---|---|
| Función que calcula algo y lo imprime con `println` | El valor no se puede reutilizar; en Android no se ve | Devolver el valor y que quien llama lo imprima |
| `println(objeto.mostrar())`, donde `mostrar()` ya imprime | Sale el dato y, después, `kotlin.Unit` | `mostrar()` debe devolver el texto, o mejor, sobrescribir `toString()` |
| Llamar a una función que devuelve algo sin usar el resultado | El cálculo se pierde y nada cambia | Guardarlo: `val rebajado = aplicarDescuento(precio)` |
| `fun f() = { ... }` | La función devuelve una lambda: se imprime `MainKt$$Lambda...` | Quitar las llaves: `fun f() = ...` |
| Un parámetro que la función recibe y no usa | El dato que se pasa no influye en el resultado | Usarlo o quitarlo de la firma |
| Una función de cálculo que modifica propiedades del objeto | Cada llamada cambia el estado: el resultado depende de cuántas veces se llame | Calcular y devolver sin modificar nada |
| `toInt()` o `toDouble()` sobre lo que escribe el usuario | Una letra cierra el programa | `toIntOrNull()` / `toDoubleOrNull()`, o la función `leerDouble` del apartado 4.10 |

## Ejercicios

1. Función `aplicarDescuento(precio: Double, porcentaje: Double = 10.0): Double`.
2. Función de orden superior `repetir(veces: Int, accion: (Int) -> Unit)` que ejecute `accion` pasándole el número de iteración.
3. Crea una calculadora que guarde las operaciones en un `Map<String, (Double, Double) -> Double>` y permita al usuario elegir por consola. Usa la función `leerDouble` del apartado 4.10 para leer los operandos.
4. Reescribe con lambdas un `Comparator` anónimo de Java que hayas usado en 1º.
5. **Predice la salida.** Sin ejecutarlo, escribe qué imprimirá cada bloque. Después compruébalo.
   ```kotlin
   // A
   fun saludo(nombre: String) { println("Hola, $nombre") }
   println(saludo("Eva"))

   // B
   fun mitad(n: Double) = n / 2
   val x = 10.0
   mitad(x)
   println(x)

   // C
   fun cuadrado(n: Int) = n * n
   println(cuadrado(cuadrado(2)))

   // D
   fun siete() = { 7 }
   println(siete()())
   ```
6. **Refactoriza.** Esta función mezcla calcular e imprimir. Reescríbela para que devuelva el texto y muéstralo desde `main` para tres alumnos distintos.
   ```kotlin
   fun informe(nombre: String, nota: Double) {
       val resultado = if (nota >= 5) "aprobado" else "suspenso"
       println("$nombre: $nota ($resultado)")
   }
   ```

**Para profundizar:** lee sobre `inline fun` y por qué las funciones de orden superior de la librería estándar están marcadas como `inline`.
