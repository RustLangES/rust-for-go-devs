# Funciones

Las funciones en Rust y Go tienen algunas similitudes pero también diferencias 
importantes debido a la filosofía de cada lenguaje.

|                             | Go                                                | Rust                            |
| --------------------------- | ------------------------------------------------- | ------------------------------- |
| Palabra clave               | `func`                                            | `fn`                            |
| Tipado del parámetro        | Después del nombre del parámetro                  | Después de los dos puntos (`:`) |
| Retorno                     | Después de paréntesis, antes de `{`               | Después de `->`                 |


Veamos un ejemplo de esto, en Go haríamos algo como esto:

```go,no_run
func suma(a int, b int) int {
    return a + b
}
```

Mientras que lo mismo en Rust sería:

```rust,ignore
fn suma(a: i32, b: i32) -> i32 {
    return a + b;
}
```

Bastante parecido ¿verdad? Sin embargo esa no es la forma más idiomatica de 
hacerlo en Rust, sino que lo más idiomatico seria utilizar una característica de 
Rust, en la que la ultima expresión de la función tiene un return implícito si
es que no termina en `;`:

```rust,ignore
fn suma(a: i32, b: i32) -> i32 {
    a + b // retorna la ultima expresión si no termina en ;
}
```

Esto funcionara con todo lo que sea una expresión como por ejemplo lo son los 
controles de flujos, que veremos más adelante.

## Closures

También podemos trabajar como Closures las funciones, en Go esto se vería de 
esta forma:

```go
#package main
# 
#import (
#	"fmt"
#)
# 
#func main() {
    x := 5
    sumar := func(y int) int {
        return x + y
    }
    fmt.Println(sumar(3)) // Imprime 8
#}
```

Mientras que en Rust seria:

```rust
    let x = 5;
    let sumar = |y| x + y;  // closure que "captura" x

    println!("{}", sumar(3)); // Imprime 8
```

Algo curioso es que en la closure de Rust no es necesario definir el tipo del 
parámetro si es que no es ambiguo, en este caso como solo tiene un caso y el
valor que recibe es `3` logra entender que `y` es de tipo `i32`, sin embargo
podríamos tiparlo nosotros de ser necesario:


```rust
    let x = 5;
    let sumar = |y:i32| x + y;  // closure que "captura" x

    println!("{}", sumar(3)); // Imprime 8
```

Veremos más acerca del uso avanzado de funciones en algunos capítulos más 
adelante.

