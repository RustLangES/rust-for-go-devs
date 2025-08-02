# Genéricos

Tanto en Go como en Rust, los genéricos resuelven el mismo problema: evitar la 
repetición de código cuando solo cambia el tipo de dato. Nos permiten escribir 
funciones, estructuras y tipos que funcionan con múltiples tipos sin duplicar 
la lógica.

Antes de los genéricos, en Go era común ver duplicación de funciones para 
distintos tipos o declaraciones de tipos utilizando interfaces.

Pero también podríamos crear un único tipo de dato, posteriormente a la llegada 
de Go 1.8 podemos empezar a encontrarnos código así:

{{#tabs }}
{{#tab name="Go antes de v1.8" }}

```go
#package main
#
#import "fmt"
#
type Caja struct {
    valor interface{}
}
#
#func main() {
#    c := Caja{valor: 42}
#    fmt.Println(c.valor) // 42
#
#    c2 := Caja{valor: "hola"}
#    fmt.Println(c2.valor) // hola
#
#    // Si querés operar con el valor, tenés que hacer type assertion
#    if v, ok := c.valor.(int); ok {
#        fmt.Println(v + 10) // 52
#    } else {
#        fmt.Println("No es un int")
#    }
#}
```

{{#endtab }}
{{#tab name="Go despues de v1.8" }}

```go
#package main
#
#import "fmt"
#
type Caja[T any] struct {
    valor T
}
#
#func main() {
#    c := Caja[int]{valor: 42}
#    fmt.Println(c.valor)
#
#    c2 := Caja[string]{valor: "hola"}
#    fmt.Println(c2.valor)
#}
```
{{#endtab }}
{{#endtabs }}

Rust tiene desde la primera versión genéricos por lo que no existe tal división
y el mismo código de ejemplo se podría ver de la siguiente forma:

```rust
struct Caja<T> {
    valor: T,
}
#
#fn main() {
#    let c = Caja { valor: 42 };
#    println!("{}", c.valor);
#
#    let c2 = Caja { valor: String::from("hola") };
#    println!("{}", c2.valor);
#}
```


En el caso de las funciones genéricas con Go obtendríamos un código similar a 
este:

{{#tabs }}
{{#tab name="Go antes de v1.8" }}
```go
func SumarInts(a, b int) int {
    return a + b
}

func SumarFloats(a, b float64) float64 {
    return a + b
}
```

{{#endtab }}
{{#tab name="Go despues de v1.8" }}
```go
func Sumar[T int | float64](a, b T) T {
    return a + b
}
```
{{#endtab }}
{{#endtabs }}

A diferencia de Go, en Rust no se suele especificar el tipo de dato en el 
genérico, lo que se especifica es el comportamiento del tipo de dato.

Rust evita en multiples aspectos las concreciones, lo cual nos permite código
más abstraído.

<div class="info">

Rust esta discutiendo a fecha de publicación de este capitulo acerca de tener
tipos de datos genéricos para enteros y para flotantes, es decir un tipo de dato
que pueda ser cualquier variante de entero o flotante.

Sin embargo si lo que queremos hacer es una comparación fiel a lo que vemos 
en el caso de Go es posible que no necesitemos en genéricos.

<details>
    <summary>Ejemplo hecho en Rust</summary>

En Rust aprovecharíamos las capacidades de los enums para resolver esto.

```rust
enum Numero {
    Entero(isize),
    Flotante(f64),
}

fn sumar(a: &Numero, b: &Numero) -> Numero {
    match (a, b) {
        (Numero::Entero(x), Numero::Entero(y)) => Numero::Entero(x + y),
        (Numero::Flotante(x), Numero::Flotante(y)) => Numero::Flotante(x + y),
        _ => panic!("Tipos incompatibles"), 
        // ☝️ no podemos sumar un entero y un flotante sin conversión explicita
    }
}
```

No se suele discriminar por `struct` o variante de `enum` sino por el `trait`, 
el comportamiento como tal porque si sabemos los tipos concretos que debemos
utilizar entonces no necesitamos genéricos.

También **es posible que en el futuro** esta feature de Go y muchos otros 
lenguajes de `union types`/`Anonymous sum types` también este disponible en 
Rust. Y remarco posible porque es algo que esta en discusión. Hablaremos
acerca de como se evoluciona el lenguaje, como se discute y se toman decisiones
más adelante para que si les interesa esta feature puedan comentarlo en las 
discusiones oficiales.

</details>

Si hablamos de concreciones es posible que con crear las funciones a mano
también sea una buena idea en lugar de usar un genérico una lista de 
tipos de datos específicos.

</div>

Debido a esta idea de buscar abstracciones es por eso que esto tiene relación 
con los `traits` como veremos un poco más abajo con los `trait bounds`.

## Polimorfismo con `static dispatch`

Supongamos que queremos escribir funciones que trabajen con múltiples tipos 
diferentes, siempre que estos compartan un comportamiento común.

Ya vimos qué es un `trait` y posiblemente en Go harías algo como esto:

```go,no_run
type Dibujable interface {
	Dibujar() string
}

func render(d Dibujable) {
	fmt.Println(d.Dibujar())
}
```

El parámetro `d` puede ser un `Circulo`, `Rectangulo`, etc. Lo importante es que 
cumpla con la interfaz.

La función render acepta cualquier tipo que implemente el método `Dibujar`. La 
función no sabe con lo que esta trabajando, solo necesita saber que puede 
invocar `Dibujar()`.

Y en parte en esto consiste en Polimorfismo, compartir comportamiento entre
distintos tipos de datos, da igual como lo implementen pero sabes que 
todos tienen un punto en común.

En Rust lo hacemos con un concepto, `trait bound`:

```rust
trait Dibujable {
    fn dibujar(&self) -> String;
}

fn render<T: Dibujable>(d: T) {
    println!("{}", d.dibujar());
}
```

Este es un ejemplo sencillo de trait bound, en donde `T: Dibujable` indica que
la función acepta cualquier parametro que implemente el trait `Dibujable`.

Nuevamente esto es más estricto que en Go, Go asume que cumple con la interfaz
si implementa un método con el mismo nombre, en el caso de Rust no es así, 
requiere una implementación explicita.

Rust ofrece también una sintaxis alternativa, más breve quizás:

```rust
trait Dibujable {
    fn dibujar(&self) -> String;
}

fn render(d: impl Dibujable) {
    println!("{}", d.dibujar());
}
```

Ambas formas generan código monomorfizado, es decir el compilador generara una
versión especifica de la función para cada tipo concreto que uses, esto se 
suele llamar `static dispatch`.

Es decir, si tenemos:

```rust
#trait Dibujable {
#    fn dibujar(&self) -> String;
#}
struct Cuadrado;
struct Circulo;

impl Dibujable for Circulo {
    // ...
#    fn dibujar(&self) -> String {
#        "Circulo dibujado".to_string()
#    }
}
impl Dibujable for Cuadrado {
    // ...
#    fn dibujar(&self) -> String {
#        "Cuadrado dibujado".to_string()
#    }
}

#fn render(d: impl Dibujable) {
#    println!("{}", d.dibujar());
#}
#
fn main() {
    let circulo = Circulo;
    let cuadrado = Cuadrado;
    render(cuadrado);
    render(circulo);
}
```

El resultado una vez compilado nos genera dos funciones `render` que en un caso
aceptara usar `Cuadrado` y en otro caso `Circulo`.

<div class="info">

Esto en el binario final puede representar un pequeño incremento en tamaño,
no algo muy significativo pero si estamos usando `static dispatch` para una 
función que recibe multiples tipos de datos de esta manera es posible que lo 
notemos.


</div>

Más adelante veremos un poco acerca de `Dynamic Dispatch`, otra forma de hacer 
esto con algunas ventajas considerables.

## Composición de traits multiples

A medida que nuestros programas crecen en complejidad, queremos que ciertas 
funciones trabajen únicamente con tipos que cumplan múltiples comportamientos. 
En Rust, eso se logra combinando varios trait bounds, es decir, especificando 
que un tipo genérico debe implementar más de un trait a la vez. Esto es 
equivalente a lo que en Go se hace cuando una interfaz embebe múltiples 
interfaces.

```go
type LeerEscribir interface {
	io.Reader
	io.Writer
}

func procesar(l LeerEscribir) {
	// ...
}
```

En Rust en lugar de hacer esto haríamos algo como esto:

```rust
fn procesar<T: Read + Write>(io: T) {
    // ...
}
```

<div class="info important">

Esto significa que si el trait `Read` tiene un método `read` y `Write` tiene un
método `write`, el parámetro `io` tendrá ambos métodos para utilizar.

</div>

Hay otra forma de representar esto que es con el `where`:

```rust
fn procesar<T>(io: T) where T: Read + Write {
    // ...
}
```

Los trait bounds se pueden componer de distintas formas. Esto permite crear 
funciones que trabajen con tipos muy específicos sin perder generalidad.

A medida que avancemos en nuestro expertiz en Rust podremos notar que esto
nos ayuda a generar APIs de librerias y experiencia de desarrollo realmente 
buena, simplificando muchas cosas.

