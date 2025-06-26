# Estructuras (`struct`)

Las estructuras en Rust y Go comparten algunas similitudes superficiales:

- Ambas se definen con la palabra clave `struct`.
- Ambas se usan para modelar tipos compuestos.
- Ambas pueden tener métodos asociados (aunque se definen de forma diferente).
- Pueden implementar múltiples `traits` en Rust de la misma manera que pueden 
  implementar múltiples interfaces en Go.
- Se almacenan en la pila (stack) por defecto.

Más adelante ahondaremos en algunas de las cosas mencionadas.

Sin embargo, las diferencias semánticas y de uso son profundas y reflejan los 
distintos modelos de diseño de cada lenguaje.

## Estructura y comportamiento separados (Métodos/Funciones Asociadas)

En Rust, struct define solo los datos. El comportamiento se define por separado 
en bloques `impl`.

En Go, los métodos se asocian mediante receivers, y no están encapsulados dentro 
de un bloque como en Rust.

Veamos este ejemplo hecho en Go:

```go
package main

import (
    "fmt"
    "math"
)

type Punto struct {
    X float64
    Y float64
}

func CalcularDistanciaEntreDosPuntos(p1 Punto, p2 Punto) float64 {
    dx := p2.X - p1.X
    dy := p2.Y - p1.Y
    return math.Sqrt(dx*dx + dy*dy)
}

func main() {
    puntoA := Punto{X: 1.0, Y: 2.0}
    puntoB := Punto{X: 4.0, Y: 6.0}
    distancia := CalcularDistanciaEntreDosPuntos(puntoA, puntoB)
    fmt.Printf("La distancia entre los puntos es: %f\n", distancia)
}

```

Definimos una estructura `Punto` con dos campos `X` e `Y`, y una función 
`CalcularDistanciaEntreDosPuntos` que toma dos puntos y calcula la distancia 
entre ellos utilizando la fórmula de distancia euclidiana.

En Rust, el mismo concepto se implementaría de la siguiente manera:

```rust
struct Punto {
    x: f64,
    y: f64,
}

fn calcular_distancia_entre_dos_puntos(p1: &Punto, p2: &Punto) -> f64 {
    let dx = p2.x - p1.x;
    let dy = p2.y - p1.y;
    (dx.powi(2) + dy.powi(2)).sqrt()
}

fn main() {
    let punto_a = Punto { x: 1.0, y: 2.0 };
    let punto_b = Punto { x: 4.0, y: 6.0 };
    let distancia = calcular_distancia_entre_dos_puntos(&punto_a, &punto_b);
    println!("La distancia entre los puntos es: {distancia}");
}
```

Para ejemplo practico ocultaremos la declaración de la estructura
en ambos casos pero igualmente el ejemplo seguirá siendo ejecutable y 
podrás ver el código oculto.

Este ejemplo si lo pensamos tendría sentido que estuviera contenido como un
método asociado a la estructura `Punto`, ya que está directamente relacionado
con los datos que contiene.

Nosotros si deseamos definir un método o función asociada a la estructura
podríamos hacerlo de la siguiente manera:

```rust
#struct Punto {
#    x: f64,
#    y: f64,
#}
#
impl Punto {
    fn calcular_distancia_a_(&self, otro: &Punto) -> f64 {
        let dx = otro.x - self.x;
        let dy = otro.y - self.y;
        (dx.powi(2) + dy.powi(2)).sqrt()
    }
}

fn main() {
    let punto_a = Punto { x: 1.0, y: 2.0 };
    let punto_b = Punto { x: 4.0, y: 6.0 };
    let distancia = punto_a.calcular_distancia_a_(&punto_b);
    println!("La distancia entre los puntos es: {distancia}");
}
```

En donde los métodos asociados se definen dentro de un bloque `impl`
que se refiere al tipo de dato `Punto`. Aquí, `&self` es una
referencia al objeto actual, similar a `this` en otros lenguajes orientados a
objetos.

En Go, podríamos definir un método asociado a la estructura de la siguiente manera:

```go
#package main
#
#import (
#    "fmt"
#    "math"
#)
#
#type Punto struct {
#    X float64
#    Y float64
#}
#
func (p *Punto) CalcularDistanciaA(otro *Punto) float64 {
    dx := otro.X - p.X
    dy := otro.Y - p.Y
    return math.Sqrt(dx*dx + dy*dy)
}

func main() {
    puntoA := Punto{X: 1.0, Y: 2.0}
    puntoB := Punto{X: 4.0, Y: 6.0}
    distancia := puntoA.CalcularDistanciaA(&puntoB)
    fmt.Printf("La distancia entre los puntos es: %f\n", distancia)
}
```

Además debemos hacer algunas aclaraciones quizás un poco obvias, tanto en Rust 
como en Go:

- No hay herencia de structs.
- No hay clases.
- El polimorfismo se logra por composición y traits/interfaces.

## Métodos y Mutabilidad

Profundizando en el ejemplo anterior, en Rust las variables son 
inmutables por defecto, por lo que si queremos modificar un campo de la
estructura debemos declararla como mutable, lo mismo ocurre en métodos:

```rust
#struct Punto {
#    x: f64,
#    y: f64,
#}
#
impl Punto {
    fn mover(&mut self, dx: f64, dy: f64) {
        self.x += dx;
        self.y += dy;
    }

    fn calcular_distancia_a_(&self, otro: &Punto) -> f64 {
        let dx = otro.x - self.x;
        let dy = otro.y - self.y;
        (dx.powi(2) + dy.powi(2)).sqrt()
    }
}

fn main() {
    let mut punto_a = Punto { x: 1.0, y: 2.0 }; // Declaramos punto_a como mutable
    let punto_b = Punto { x: 4.0, y: 6.0 };

    punto_a.mover(1.0, 0.0); // Modificamos punto_a

    let distancia = punto_a.calcular_distancia_a_(&punto_b);
    println!("La distancia entre los puntos es: {distancia}");
}
```

En Go no percibiremos muchos cambios, ya que las variables son mutables por
defecto:

```go
#type Punto struct {
#    X float64
#    Y float64
#}
#
func (p *Punto) Mover(dx, dy float64) {
    p.X += dx
    p.Y += dy
}

func (p *Punto) CalcularDistanciaA(otro *Punto) float64 {
    dx := otro.X - p.X
    dy := otro.Y - p.Y
    return math.Sqrt(dx*dx + dy*dy)
}

func main() {
    puntoA := Punto{X: 1.0, Y: 2.0}
    puntoB := Punto{X: 4.0, Y: 6.0}

    puntoA.Mover(3.0, 4.0) // No necesitamos declarar puntoA como mutable

    distancia := puntoA.CalcularDistanciaA(&puntoB)
    fmt.Printf("La distancia entre los puntos es: %f\n", distancia)
}
```

Ahora que tenemos dos metodos asociados a la estructura `Punto`, podemos ver
algunas de las ventajas de Rust, las cuales son:
- Los métodos deben ser definidos en un scope especifico (`impl`), lo que evita
  la contaminación del espacio de nombres global.
- Los métodos nos exigen distinguir entre instancias mutables e inmutables,
  lo que nos ayuda a evitar errores o detectar fácilmente donde ocurren 
  modificaciones no deseadas.

## Constructores

En Go y en Rust no tenemos constructores integrados en el lenguaje,
pero podemos definir funciones que actúen como constructores.

En el caso de Rust, veremos que se respeta muchisimo la convención de 
un método `new` que actúa como constructor:

```rust
#struct Punto {
#    x: f64,
#    y: f64,
#}
#
impl Punto {
    fn new(x: f64, y: f64) -> Self { // 👈 método clave
        Punto { x, y }
    }

    // Resto de métodos ...
#    fn mover(&mut self, dx: f64, dy: f64) {
#        self.x += dx;
#        self.y += dy;
#    }
#
#    fn calcular_distancia_a_(&self, otro: &Punto) -> f64 {
#        let dx = otro.x - self.x;
#        let dy = otro.y - self.y;
#        (dx.powi(2) + dy.powi(2)).sqrt()
#    }
}

fn main() {
    let mut punto_a = Punto::new(1.0, 2.0);
    let punto_b = Punto::new(4.0, 6.0);
    
    punto_a.mover(1.0, 0.0);

    let distancia = punto_a.calcular_distancia_a_(&punto_b);
    println!("La distancia entre los puntos es: {distancia}");
}
```

En Rust utilizamos el tipo de dato `Self` para referirnos al tipo actual
dentro del bloque `impl`. Esto es una convención que se utiliza para
indicar que el método `new` devuelve una instancia del tipo `Punto`.
Puedes considerarlo como un alias para el tipo actual.

En Go, no hay una convención estricta para los constructores, pero podemos
definir una función que actúe como constructor de la siguiente manera:

```go
#type Punto struct {
#    X float64
#    Y float64
#}
#
func NewPunto(x, y float64) *Punto { // 👈 función clave
    return &Punto{X: x, Y: y}
}

// Resto de métodos ...

#func (p *Punto) Mover(dx, dy float64) {
#    p.X += dx
#    p.Y += dy
#}
#
#func (p *Punto) CalcularDistanciaA(otro *Punto) float64 {
#    dx := otro.X - p.X
#    dy := otro.Y - p.Y
#    return math.Sqrt(dx*dx + dy*dy)
#}
#
func main() {
    puntoA := Punto{X: 1.0, Y: 2.0}
    puntoB := Punto{X: 4.0, Y: 6.0}

    puntoA.Mover(3.0, 4.0)

    distancia := puntoA.CalcularDistanciaA(&puntoB)
    fmt.Printf("La distancia entre los puntos es: %f\n", distancia)
}
```

## ¿Cómo mostrar una estructura?

- En Rust, se implementa el trait Display o Debug para imprimir structs.

- En Go, se puede definir el método String() (de fmt.Stringer).

Generalmente lo más recomendable es que utilices el trait `Debug` en Rust,
ya que es más fácil de implementar y te permite imprimir la estructura de forma
rápida, sencilla y además inspeccionar de forma recursiva los campos.

Como ejemplo, si quisiéramos imprimir la estructura `Punto` en Rust:

```rust
#[derive(Debug)] // 👈 El trait Debug se derivara e implementara automáticamente
struct Punto {
    x: f64,
    y: f64,
}

fn main() {
    let punto = Punto { x: 1.0, y: 2.0 };
    println!("{punto:?}", ); // 👈 Imprime la estructura usando Debug
}
```

En Rust usamos `#[derive(Debug)]` para indicar que queremos que Rust implemente
el trait `Debug` para nuestra estructura `Punto`. Esto nos habilitara imprimir 
la estructura usando el formato `?` (el cual significa depuración) en el 
`println!`, nosotros podemos especificarle a Rust que tipo de formato utilizar 
en las macros `println!` y `format!`, entre otras, utilizando `:`.

De esta manera `{punto:?}` imprimirá la estructura con formato de depuración.

En Go no existe un equivalente directo a `#[derive(Debug)]` como en Rust.

```go
package main

import (
    "fmt"
)

type Punto struct {
    X float64
    Y float64
}

func main() {
    p := Punto{X: 1.0, Y: 2.0}
    fmt.Printf("%+v\n", p) // Imprime: {X:1 Y:2}
}
```


Veremos más acerca de los `traits` y los derivables más adelante.