# Punto de entrada

Tanto en Rust como en Go, el punto de entrada de un programa es la función 
`main`.
En Go, la función `main` se define dentro del paquete `main`, que es el
paquete por defecto para la ejecución de programas. Aquí hay un ejemplo simple:

```go
package main

import "fmt"

func main() {
    fmt.Println("¡Hola, mundo!")
}
```

En Rust, la función `main` también es el punto de entrada del programa, y se 
define de manera similar a Go, pero con una sintaxis diferente:

```rust
fn main() {
    println!("¡Hola, mundo!");
}
```

La función `main` es donde comienza la ejecución del programa. En Go, se utiliza 
el paquete `fmt` para imprimir en la consola, mientras que en Rust se utiliza la 
macro `println!` para lograr lo mismo.<sup>1</sup>

Ambos ejemplos imprimen "¡Hola, mundo!" en la consola, lo que demuestra cómo
se define el punto de entrada en ambos lenguajes de programación.

---

<sup>1</sup> En Rust, las macros se definen con un signo de exclamación `!` al final,
lo que indica que se trata de una macro y no de una función normal. Esto es
una característica distintiva de Rust, ya que las macros permiten una mayor
flexibilidad y metaprogramación en comparación con las funciones normales.
Veremos más sobre macros en Rust en secciones posteriores. Esta macro
a diferencia de otras que veremos más adelante, es una macro built-in, lo que 
significa que está disponible por defecto en el lenguaje y no requiere
importar ningún crate adicional. Esto la hace muy conveniente para tareas
comunes como imprimir en la consola, ya que no es necesario importar ningún
módulo o biblioteca adicional para utilizarla.