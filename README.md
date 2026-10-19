# Tutorial de Kotlin para DAM · 2º curso

Material de apoyo del módulo **Programación Multimedia y Dispositivos Móviles**.
El objetivo es llegar a Android Studio con una base sólida de Kotlin, partiendo de lo que ya sabéis de Java.

## Requisitos previos

- Haber cursado Programación y Programación Orientada a Objetos (Java) en 1º.
- **JDK 17 o 21 (LTS)** instalado.
- **IntelliJ IDEA Community Edition** para las unidades 01–09.
- **Android Studio** a partir de la unidad 10.

## Índice

| Unidad | Contenido | Tipo |
|---|---|---|
| [01](01-entorno-y-primer-programa.md) | Entorno, primer programa, estructura de un proyecto Gradle | concepto |
| [02](02-sintaxis-tipos-y-control-de-flujo.md) | Variables, tipos, `if`/`when`, bucles, rangos | concepto |
| [03](03-null-safety.md) | Seguridad frente a nulos: `?`, `?.`, `?:`, `!!`, `let` | concepto |
| [04](04-funciones-y-lambdas.md) | Funciones, parámetros por defecto, lambdas, funciones de orden superior | concepto |
| [05](05-colecciones.md) | Listas, mapas, conjuntos y operadores funcionales: `filter`, `map`, `sumOf`, `count`, ordenar sin modificar, resúmenes con `data class` | práctica |
| [06](06-poo-clases-y-data-class.md) | Clases, parámetros y propiedades del constructor, valores por defecto, datos opcionales, `toString()`, `data class` | concepto |
| ↳ [val vs var](val-vs-var.md) | Ampliación: `val`/`var`, mutabilidad de objetos, arrays, comparación con Java | complementario |
| ↳ [Excepciones](excepciones.md) | Ampliación: `throw`, `try/catch`, sin excepciones comprobadas, `require`/`check`, excepciones propias, `use` | complementario |
| [07](07-herencia-interfaces-sealed.md) | Clases abstractas, constructores en una jerarquía, `toString()` con `super`, herencia `open`, interfaces, `object`, `enum`, `sealed class` | concepto |
| [08](08-extensiones-y-scope-functions.md) | Funciones de extensión y `let/run/with/apply/also` | concepto |
| [09](09-corrutinas.md) | Concurrencia: `suspend`, `launch`, `async`, `Dispatchers` | concepto |
| [10](10-puente-a-android.md) | De Kotlin a Android: primer proyecto y Jetpack Compose | Android |
| [11](11-proyecto-final.md) | Proyecto integrador por fases | proyecto |

## Cómo trabajar con este repositorio

1. Clonad el repositorio: `git clone <url>`.
2. Leed una unidad completa antes de escribir código.
3. Escribid **vosotros** los ejemplos: copiar y pegar no enseña nada.
4. **Ejecutad cada ejemplo y comparad lo que sale con el bloque «Salida»** que lo acompaña. Si no coincide, buscad la diferencia antes de seguir: es la mejor forma de detectar lo que no habéis entendido.
5. Resolved los ejercicios del final de cada unidad en vuestro propio proyecto. En los ejercicios de **«Predice la salida»**, escribid primero vuestra respuesta en papel y solo después ejecutad el código.
6. Leed la tabla de **errores típicos** de cada unidad: recoge fallos reales de prácticas y pruebas de cursos anteriores.
7. Los bloques marcados con **Para profundizar** son ampliación voluntaria.

## Convención de los ejemplos

Hay dos tipos de fragmentos de código:

- **Ejemplos completos**, que incluyen su propia función `main()`. Se pueden copiar en un fichero `.kt` y ejecutar tal cual. Van seguidos de un bloque **«Salida»** con lo que imprimen.
- **Fragmentos breves**, sin `main()`, que ilustran una sola idea. Para probarlos, colocad las líneas de código dentro de una función `main()`.

En los dos casos, **las clases, interfaces y `object` se declaran fuera de `main()`**, a nivel de fichero o en ficheros propios. Dentro de `main()` solo se crean los objetos y se usan:

```kotlin
// Fichero Main.kt
class Libro(val titulo: String)          // fuera de main

fun main() {
    val libro = Libro("Platero y yo")    // dentro de main: crear y usar
    println(libro.titulo)
}
```

Los comentarios explican **por qué** se toma cada decisión de diseño, no sólo qué hace la línea.

> **Sobre los decimales:** cuando un ejemplo formatea números con `"%.2f".format(...)`, el separador depende de la configuración regional de vuestro equipo. Podéis ver `3,14` donde la salida dice `3.14`, o al revés: es el mismo resultado.

## Antes de entregar una práctica

Revisad esta lista. Cada punto corresponde a un error que se repite en las entregas:

- [ ] El proyecto **compila** sin errores. Un proyecto que no compila no se puede ejecutar ni comprobar.
- [ ] Lo he **ejecutado** con datos cuyo resultado conozco de antemano (por ejemplo, 10 € + 10 % = 11 €).
- [ ] Si no tiene sentido crear objetos de la clase base, es `abstract`, y los métodos que toda subclase debe tener también son `abstract`.
- [ ] Cada valor por defecto está en el constructor que se llama (el de la subclase), no solo en la clase base.
- [ ] Cada dato que recibe una subclase llega a la clase base o se guarda con `val`.
- [ ] Los datos opcionales son anulables con `= null`, y se muestran con `?:`.
- [ ] Las clases sobrescriben `toString()`, y las subclases lo amplían con `super.toString()`.
- [ ] Las funciones que calculan algo **devuelven** el valor en lugar de imprimirlo.
- [ ] Uso el resultado de cada `map`, `filter` y `sortedBy`.
- [ ] Los números que escribe el usuario se leen con `toIntOrNull()` o `toDoubleOrNull()`, y una entrada incorrecta no cierra el programa.
- [ ] Importes y porcentajes son `Double`, nunca `Int`.
- [ ] He borrado el código comentado, los `import` sin usar y el comentario de plantilla del IDE.

---
*Material docente. Uso libre dentro del ciclo.*
