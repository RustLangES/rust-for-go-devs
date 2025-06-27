# Nulabilidad y Opcionalidad

Este capitulo sera separado en [Ejemplos rápidos](#ejemplos-rápidos) y en 
[Lectura](#lectura) veremos más detalladamente en que se diferencian Rust y Go
con respecto a este tema.


## Ejemplos rápidos

En Go suele representarse la nubilidad de esta forma:
```go,no_run
some := 2 // short declaration
var some int = 2 
var none *int = nil
```

Hay varias formas de declarar la variable.
Esto en Rust se representaria como:

```rust,no_run
let some = 1; // Short
let some: i32 = 1; // declaración con tipo
let some = Some(1); // Con inferencia
let some: Option<i32> = Some(1); // declaración con tipo
let none: Option<i32> = None; // única forma de declarar un valor nulleable
```

Sin embargo el valor en el caso de Rust reside en el interior, esta envuelto.
Por lo que una vez que logremos saber si el contenido es o no un valor valido,
podemos usar diversos metodos para utilizar el contenido.

```go
#package main
#
#import "fmt"
#
func main() {
    var username *string = nil
    if username == nil {
        fmt.Println("Not found")
    } else {
        fmt.Println(*username)
    }
}
```

En el caso de Go usamos directamente la referencia, en el caso de Rust:

```rust
fn main() {
    let username: Option<String> = None;
    if username.is_none() {
        println!("Not found")
    } else {
        println!("{}", username.unwrap())
    }
}
```

De esta forma podemos acceder con el método `unwrap()` al valor que reside 
dentro. Sin embargo esta forma de hacerlo es bastante peligrosa, hay que 
asegurarnos de que va a tener algo dentro previamente sino obtendremos un 
`panic`. Más adelante veremos multiples maneras de hacer lo mismo pero mucho 
mejor y sin correr el riesgo de que recibir un `panic`.

---

## Lectura

En diversos lenguajes tenemos el valor `null`, en Go representamos el valor 
utilizando `nil`, lo usamos para representar un valor que es faltante, ausente
o lógicamente no inicializado. Por ejemplo:

```go
#package main
#
#import "fmt"
#
func search(id int) *string {
    if id == 1 {
        username := "Admin"
        return &username
    }

    return nil
}

func main() {
    username := search(2)
    if username == nil {
        fmt.Println("Not found")
    } else {
        fmt.Println(*username)
    }
}
```

En este caso estamos devolviendo un puntero a `string`, ese `string` puede ser 
un valor nulo, por lo cual intentamos manejarlo correctamente, pero esto es 
porque nosotros sabemos que retornara `nil`, es posible que nosotros no sepamos
cuando nos devuelve un valor `nil` y si ocurre eso tendremos algunos problemas.

El famoso [error del millón de dolares][billion-dolar-mistake], este error se 
debe a que muchas veces no sabemos cuando una referencia es nulleable y 
cometemos el error de que bajo una casuistica determinada asumir que sera así 
en todas los casos. Como la gran mayoría de lenguajes, Go no puede determinar 
la solución a este error, en parte porque no es exhaustivo, cosa que Rust si es.

Uno de los grandes motivos de adopción de Rust es su garantía de seguridad,
una de ellas es justamente esta, un lenguaje null safety.

¿Como consigue Rust solucionar este problema?

Utiliza el concepto de los `Option`. Los `Option` en Rust no son más que un tipo
de dato que representa si hay un valor o no lo hay. Similar a la idea de null, 
¿verdad?.
Rust sin embargo gracias a estas dos variantes, logra asegurar que estes 
teniendo en cuenta ambos casos, internamente este tipo de dato solamente se 
representa como un enum.

```rust,no_run
enum Option<T> {
    Some(T),
    None
}
```

Es apenas un poco más complejo que esto, pero internamente funciona así.
Es debido a esto que Rust no asume, no tiene certeza de cual de las dos 
variantes hay, es por eso que hay que tener en cuenta ambas.

El mismo ejemplo anterior podríamos representarlo como:

```rust
fn search(id: i32) -> Option<String> {
    if id == 1 {
        let username = "Admin".to_string();
        return Some(username);
    }
    None
}

fn main() {
    let username = search(2);
    if username.is_none() {
        println!("Not found");
    } else {
        println!("{}", username.unwrap());
    }
}
```

Sin embargo estamos usando el metodo `unwrap()` de `Option`, porque en la 
definición del enum `Option` el contenido puede estar en vuelto en la variante
`Some(T)`. Nosotros queremos lo que hay dentro de `Some`, es por eso que 
literalmente debemos desenvolverlo y obtener el valor interno.
Hacer esto no es del todo seguro, hay que previamente verificar cual variante es
ya que un `unwrap()` sobre la variante `None` implicaria obtener un `panic`.

Rust al utilizar un envoltorio en lugar de un valor crudo como Go puedes tener
metodos y utilidades que te permiten manejar de diversas formas las casuísticas.

Veremos más adelante formas más avanzadas de solucionar este problema de 
maneras mejores.


[billion-dolar-mistake]: https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/
