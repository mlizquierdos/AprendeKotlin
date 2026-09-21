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
| [05](05-colecciones.md) | Listas, mapas, conjuntos y operadores funcionales | práctica |
| [06](06-poo-clases-y-data-class.md) | Clases, constructores, propiedades, `data class` | concepto |
| ↳ [val vs var](val-vs-var.md) | Ampliación: `val`/`var`, mutabilidad de objetos, arrays, comparación con Java | complementario |
| [07](07-herencia-interfaces-sealed.md) | Herencia, interfaces, `object`, `enum`, `sealed class` | concepto |
| [08](08-extensiones-y-scope-functions.md) | Funciones de extensión y `let/run/with/apply/also` | concepto |
| [09](09-corrutinas.md) | Concurrencia: `suspend`, `launch`, `async`, `Dispatchers` | concepto |
| [10](10-puente-a-android.md) | De Kotlin a Android: primer proyecto y Jetpack Compose | Android |
| [11](11-proyecto-final.md) | Proyecto integrador por fases | proyecto |

## Cómo trabajar con este repositorio

1. Clonad el repositorio: `git clone <url>`.
2. Leed una unidad completa antes de escribir código.
3. Escribid **vosotros** los ejemplos: copiar y pegar no enseña nada.
4. Resolved los ejercicios del final de cada unidad en vuestro propio proyecto.
5. Los bloques marcados con **Para profundizar** son ampliación voluntaria.

## Convención de los ejemplos

Todos los fragmentos son ejecutables tal cual dentro de una función `main()`,
salvo que se indique lo contrario. Los comentarios explican **por qué** se toma
cada decisión de diseño, no sólo qué hace la línea.

---
*Material docente. Uso libre dentro del ciclo.*
