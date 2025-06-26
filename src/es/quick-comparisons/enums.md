# Enums

## 🐹 Enums en GO

En muchos lenguajes tenemos este mecanismo para definir un tipo de dato, en Go
no tenemos un tipo de dato `enum` como tal, pero podemos simularlo con
constantes y tipos personalizados como iota, lo cual puede estar bien para
enums simples como por ejemplo:

```go
#package main
#
#import "fmt"
#
type Estado int

const (
    Inactivo Estado = iota // 0
    Activo                 // 1
    Suspendido             // 2
    Eliminado              // 3
)

func main() {
    estado := Activo

    switch estado {
    case Inactivo:
        fmt.Println("El estado es: Inactivo")
    case Activo:
        fmt.Println("El estado es: Activo")
    case Suspendido:
        fmt.Println("El estado es: Suspendido")
    case Eliminado:
        fmt.Println("El estado es: Eliminado")
    default:
        fmt.Println("Estado desconocido")
    }
}
```

Esto funcionara pero es un caso muy básico con varios problemas, es posible que 
en varias oportunidades necesitemos algo más complejo, por ejemplo, si queremos
no podemos simular tagged unions/sum types/tipos algebraicos, no podemos tener 
métodos asociados a los enums, no podemos tener un enum con un valor asociado, 
etc.

Si desearamos algo más complejo, como por ejemplo un enum con un valor asociado
podríamos hacerlo de la siguiente manera:

```go
type Mensaje interface{}

type Saludar struct {
    Nombre string
}

type Mover struct {
    X, Y int
}

type Salir struct{}

func procesar(m Mensaje) {
    switch v := m.(type) {
        case Saludar:
            fmt.Println("Hola,", v.Nombre)
        case Mover:
            fmt.Printf("Mover a (%d, %d)\n", v.X, v.Y)
        case Salir:
            fmt.Println("Adiós")
        default:
            fmt.Println("Mensaje desconocido")
    }
}
```

Sin embargo recurrimos a otra manera, en la que perdemos las variantes,
no podemos tener un enum con un valor asociado.

Es por eso que en Go debemos decidir si queremos utilizar variantes simples
o estructuras con interfaces para manejar casos más complejos. 

Además como perdemos la funcionalidad de las variantes podemos generar 
código con errores, perdemos exhaustividad.

## 🦀 RUST: enum con tipos algebraicos

Por otro lado, los enums de Rust son reconocidos por ser especialmente potentes
y versátiles, permitiendo definir variantes con datos asociados, métodos y
exhaustividad en el manejo de casos.

En Rust definiriamos un enum de la siguiente manera:

```rust
enum Mensaje {
    Saludar(String),
    Mover { x: i32, y: i32 },
    Salir,
}

fn procesar(m: Mensaje) {
    match m {
        Mensaje::Saludar(nombre) => println!("Hola, {nombre}!"),
        Mensaje::Mover { x, y } => println!("Mover a ({x}, {y})"),
        Mensaje::Salir => println!("Adiós"),
    }
}
```

En este ejemplo breve estamos haciendo muchas cosas que en Go posiblemente se 
nos complicaria:

- Definimos un tipo de dato `Mensaje` el cual tiene varias variantes
    - La variante `Saludar` que tiene un `String` contenido
    - `Mover` que tiene dos campos `x` e `y` como si fuese una estructura
    - Y `Salir` que no tiene datos asociados
- Luego tenemos una función `procesar` que recibe un `Mensaje` y utiliza
  `match` para manejar cada variante de forma exhaustiva.
- Si no cubrimos todas las variantes lo sabremos, Rust nos pide cubrir todas las
  posibilidades en tiempo de compilación, lo que garantiza que no se nos olvide 
  manejar un caso.

Además, podemos agregar métodos asociados a los enums, lo que nos permite
encapsular la lógica relacionada con cada variante dentro del enum mismo.

### Representación numérica de los enums

En Rust, los enums pueden ser representados numéricamente, similar a como se 
hace en Go con `iota`, nosotros podemos definir un enum asociando un valor
numérico a cada variante, incluso podemos saltar valores o definir
valores específicos para cada variante.

```rust
enum Estado {
    Inactivo = 0,
    Activo = 1,
    Suspendido = 2,
    Eliminado = 54,
    Desconocido = 100,
}

fn main() {
    let estado = Estado::Activo;
    println!("El estado numérico es: {}", estado as u8); // -> 1
}
```

Sin embargo, es importante aclarar que no podemos transformar un numero a
una variante de enum directamente. Esto debido a que podría resultar ambiguo.
No todos los valores numéricos son válidos para los enums, es por eso que si
debemos hacer esto lo ideal seria implementar un `trait` en Rust, que
permita convertir un número a una variante de enum de forma segura, por supuesto
esto es un poco más avanzado, lo veremos más adelante en el libro, pero para
no irse con las manos vacías, aquí hay un ejemplo de cómo podríamos
implementar esto:

```rust
#[derive(Debug)]
enum Estado {
    Inactivo = 0,
    Activo = 1,
    Suspendido = 2,
    Eliminado = 54,
    Desconocido = 100,
}

impl From<u8> for Estado {
    fn from(valor: u8) -> Self {
        match valor {
            0 => Estado::Inactivo,
            1 => Estado::Activo,
            2 => Estado::Suspendido,
            54 => Estado::Eliminado,
            _ => Estado::Desconocido, // Para cualquier otro valor, usamos Desconocido
        }
    }
}

fn main() {
    let estado: Estado = 1.into(); // Convertimos el número 1 a Estado::Activo
    println!("El estado es: {estado:?}", );
}
```

De esta manera convertimos un número a una variante de enum de forma segura,
evitando ambigüedades y garantizando que solo se utilicen valores válidos.

### Métodos/Funciones asociadas a enums

En Rust, los enums pueden tener métodos asociados, lo que permite encapsular
la lógica relacionada con cada variante dentro del enum mismo. Esto es útil
para mantener el código organizado y evitar la repetición.

```rust
#[derive(Debug)]
enum Direccion {
    Norte,
    Sur,
    Este,
    Oeste,
}

impl Direccion {
    fn girar_a_la_izquierda(&self) -> Self {
        match self {
            Direccion::Norte => Direccion::Oeste,
            Direccion::Oeste => Direccion::Sur,
            Direccion::Sur => Direccion::Este,
            Direccion::Este => Direccion::Norte,
        }
    }
}

fn main() {
    let direccion_actual = Direccion::Norte;
    let nueva_direccion = direccion_actual.girar_a_la_izquierda();
    println!("Nueva dirección: {nueva_direccion:?}");
}
```

En este ejemplo, hemos definido un enum `Direccion` con cuatro variantes
y un método `girar_a_la_izquierda` que devuelve la dirección resultante al
girar a la izquierda desde la dirección actual. Esto permite que cada variante
tenga su propia lógica asociada, lo que mejora la legibilidad y mantenibilidad
del código.

