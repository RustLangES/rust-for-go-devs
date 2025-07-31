# Organización de código

A la hora de estructurar proyectos, los lenguajes modernos deben ofrecer 
mecanismos claros para divdir el código en componentes reutilizables y 
mantenibles. Tanto Go como Rust ofrecen soluciones para este problema, pero lo
hacen de maneras bastante distintas, reflejando sus filosofías.

## Importando archivos

### La forma de Go

En Go, la organización del código se basa en una convención de directorios y
paquetes:

- Cada carpeta es un paquete (`package nombre`)
- Todos los archivos dentro de una carpeta deben pertenecer al mismo paquete.
- La visibilidad: cualquier identificador que comience con mayúscula es 
  exportado (o es público), y el resto es privado en relación al paquete.

Este enfoque si bien funciona puede ser considerado un enfoque debil o simple,
esto es porque es basado en escritura, se basa en una convención a la hora de 
escribir código, y es una sutileza que muchos pueden pasar por alto en algunas
ocaciones, sobre pone la convención sobre la configuración.

{{#tabs }}
{{#tab name="Estructura de proyecto" }}
```
.
└── src
    ├── main.go
    └── util.go
```

{{#endtab }}
{{#tab name="util.go" }}
```go,no_run
package main

func Sumar(a, b int) int {
    return a + b
}
```

{{#endtab }}
{{#tab name="main.go" }}
```go,no_run
package main

import "fmt"

func main() {
    fmt.Println(util.Sumar(2, 3))
}
```
{{#endtab }}
{{#endtabs }}

### La forma de Rust

Rust, por el contrario, favorece la modularidad explícita mediante una palabra 
reservada. Cada módulo debe declararse mediante la palabra clave `mod`, y lo que 
se desea que tenga una visibilidad pública, es decir que pueda ser importado en
otros lados, debe marcarse con `pub`. Los módulos pueden estar definidos en el
mismo archivo, en archivos separados o incluso en subdirectorios.

Para este caso seguiremos con una estructura similar:

{{#tabs }}
{{#tab name="Estructura de proyecto" }}
```
.
└── src
    ├── main.rs
    └── util.rs
```

{{#endtab }}
{{#tab name="util.rs" }}
```rust,noplayground
pub fn sumar(a: i32, b: i32) -> i32 {
    a + b
}
```

{{#endtab }}
{{#tab name="main.rs" }}
```rust,noplayground
mod util;

fn main() {
    println!("{}", util::sumar(1,2));
}
```
{{#endtab }}
{{#endtabs }}

Ya con este ejemplo notaremos que Rust funciona con `namespaces` es decir con 
los `::` estaremos accediendo al modulo `util` y a la función `saludar` y 
ejecutandola con los parentesis.

Veremos un poco más de namespaces en la sección 
[Un poco acerca de los namespaces][about-namespaces]

No es necesario escribir en mayusculas para exportar, solo utilizamos la 
palabra reservada `pub`, siendo esto más notorio que un case sensitive.

Por otro lado estamos usando `mod` para declarar el modulo, en Rust, cada 
archivo que utilicemos para escribir código sera considerado un modulo y
debe ser declarado.

Es por eso que se declara en el archivo `main.rs` el modulo/archivo `util.rs`.

Esto tiene como ventaja que podemos de manera artificil, simular separaciones
de código en un mismo archivo, por ejemplo a diferencia de Go, podria tener algo
como esto:

{{#tabs }}
{{#tab name="main.rs" }}
```rust
fn main() {
    println!("{}", util::sumar(1,2));
}

mod util {
    pub fn sumar(a: i32, b: i32) -> i32 {
        a + b
    }

    fn saludar() {
        println!("hola");
    }
}
```
{{#endtab }}
{{#endtabs }}

De forma en que estamos simulando visibilidad, este mismo archivo contiene
todo el modulo `util` y la función `main` además de esto agregamos una función 
`saludar` dentro del modulo `util`.

Esta función `saludar` al no ser declarada con la palabra reservada `pub`
devolvera un error al intetar ser accedida.

{{#tabs }}
{{#tab name="main.rs" }}
```rust
fn main() {
    println!("{}", util::sumar(1,2));
    util::saludar();
}

mod util {
    pub fn sumar(a: i32, b: i32) -> i32 {
        a + b
    }

    fn saludar() {
        println!("hola");
    }
}
```
{{#endtab }}
{{#endtabs }}

Recibiremos el error:

```
function `saludar` is private
```

Explicandonos justamente esto, aunque este en el mismo archivo, si algo esta
contenido dentro de un modulo y no es declarado como publico no permitira su
acceso desde fuera del modulo, sin embargo internamente podremos continuar 
usandolo:

{{#tabs }}
{{#tab name="main.rs" }}
```rust
fn main() {
    println!("{}", util::sumar(1,2));
}

mod util {
    pub fn sumar(a: i32, b: i32) -> i32 {
        saludar();
        a + b
    }

    fn saludar() {
        println!("hola");
    }
}
```
{{#endtab }}
{{#endtabs }}

#### Un poco acerca de los namespaces

Hay varios motivos detras de esta decisión de Rust, si bien al comienzo puede
resultar algo extraño tiene buenos puntos por los cuales el lenguaje funciona 
de esta manera.

El caso más basico es porque permite desambiguar de manera sencilla, un caso 
de ejemplo podría ser el siguiente:

```rust,no_run
mod logica {
    pub fn procesar() {}
}

mod io {
    pub fn procesar() {}
}

fn main() {
    logica::procesar();
    io::procesar();
}
```

Donde tenemos varias funciones con el mismo nombre (`procesar`) pero los 
`namespaces` al ser distintos permiten desambiguar con claridad.

En una sección más adelante hablaremos de la visibilidad granular que puedes 
tener en Rust basados en este sistema.

Además de lo ya mencionado modulos logicos que pueden convivir varios dentro 
un mismo archivo dependiendo de distintas decisiones que tomemos como 
desarrolladores, por ejemplo un caso muy tipico es crear un modulo `test`
que unicamente se compilara con una flag del compilador, esa flag sera activada
cuando ejecutemos, `cargo test`.

Veremos más de testing en futuros capitulos.

#### Removiendo namespaces

Volviendo al ejemplo original, es posible que no nos guste acceder al los 
namespaces para ejecutar una función o importar algo, es por eso que en Rust
existe la palabra `use` permitiendo hacer justamente esto.


{{#tabs }}
{{#tab name="Estructura de proyecto" }}
```
.
└── src
    ├── main.rs
    └── util.rs
```

{{#endtab }}
{{#tab name="util.rs" }}
```rust,noplayground
pub fn sumar(a: i32, b: i32) -> i32 {
    a + b
}
```

{{#endtab }}
{{#tab name="main.rs" }}
```rust,no_run
mod util;

use util::sumar;

fn main() {
    println!("{}", sumar(1,2));
}
```
{{#endtab }}
{{#endtabs }}


## Manejando carpetas

Cuando terminamos trabajando con multiples carpetas y archivos es posible que
intentemos generalizar de alguna forma, darles un sentido, un nombre basado
en distintos aspectos, quizás es la funcionalidad que cumple el archivo o lo
que contiene.

También es posible que tengamos que trabajar con multiples visibilidades
dependiendo de muchas cosas y es ahí donde Rust empieza a tener cobrar mucho 
sentido, veamos los siguientes ejemplos:

### Go

{{#tabs }}
{{#tab name="Estructura de proyecto" }}
```
.
└── src
    ├── main.go
    └── services
        ├── auth.go
        └── publish.go
```

{{#endtab }}
{{#tab name="services/auth.go" }}
```go,no_run
package services

import "fmt"

func PublicLogin() {
    fmt.Println("login público")
}

func privateToken() {
    fmt.Println("token privado")
}

func validateUser() {
    fmt.Println("Validando usuario")
}
```

{{#endtab }}
{{#tab name="services/publish.go" }}
```go,no_run
package services

import "fmt"

func DoPublish() {
    validateUser() // ✅ OK: estamos en el mismo paquete
    fmt.Println("Publicando contenido...")
}

```

{{#endtab }}
{{#tab name="main.go" }}
```go,no_run
package main

import "miproyecto/services"

func main() {
    services.PublicLogin()       // ✅ OK: Comienza con mayuscula
    // services.privateToken()   // ❌ no compila: no está exportada
    // services.validateUser()   // ❌ no compila: no está exportada
}
```
{{#endtab }}
{{#endtabs }}

### Rust

{{#tabs }}
{{#tab name="Estructura de proyecto" }}
```
.
└── src
    ├── main.rs
    └── services
        ├── auth.rs
        ├── mod.rs
        └── publish.rs
```

{{#endtab }}
{{#tab name="services/auth.rs" }}
```rust,noplayground
pub fn public_login() { // 👈 esto es importable como libreria de forma externa
                        // Es decir publico en todos los niveles
    println!("login público");
}

fn private_token() {  // 👈 esto es accesible solo en este archivo
    println!("token privado");
}

pub(crate) fn internal_check() { // 👈 esto es accesible en este proyecto
    println!("solo visible internamente");
}

pub(super) fn validate_user() { // 👈 esto es accesible solo a nivel carpeta
    println!("visible solo para el módulo padre");
}

```

{{#endtab }}
{{#tab name="services/publish.rs" }}
```rust,noplayground
use super::auth; // Importamos el módulo `auth` que no es público

pub fn do_publish() {
    auth::validate_user(); // ✅ accesible porque `validate_user` es `pub(super)`
    println!("Publicando contenido...");
}
```
{{#endtab }}
{{#tab name="services/mod.rs" }}
```rust,noplayground
pub mod auth;
pub mod publish;
```
{{#endtab }}
{{#tab name="main.rs" }}
```rust,noplayground
mod services;

fn main() {
    services::auth::public_login();     // ✅ OK: es pub
    // services::auth::private_token(); // ❌ Error: no es pub
    services::auth::internal_check();   // ✅ OK: es pub(crate)
    // services::auth::validate_user(); // ❌ Error: no es pub
}
```
{{#endtab }}
{{#endtabs }}

Este código es solo a manera de ejemplificación pero teniendo en cuenta las 
comparaciones veremos que en Rust estamos usando un archivo que en Go no 
tenemos, el archivo `mod.rs`

### ¿Qué es `mod.rs`?

Es un archivo necesario en estos contextos donde queremos agrupar archivos dentro de un modulo/carpeta, en este caso `services`, `mod.rs` actua como un
punto de entrada del módulo que a su vez involucra otros modulos/archivos.

El archivo mod.rs define el módulo/carpeta y sirve como punto de entrada para 
sus submódulos.

De esta forma en proyectos más grandes se puede generar una organización 
jerárquicamás clara. Se podría tomar que `mod.rs` actúa como un índice o 
catálogo del modulo.

<div class="info">

Si poseen conocimiento en JavaScript o TypeScript, sabran que es una practica 
común aunque no siempre aconsejada, eso debido a los problemas que tienen
algunos transpiladores a la hora de hacer tree shaking, pero en el caso de Rust
esto no es así, Rust logra eliminar correctamente el código no deseado.

</div>

Esto además del tema de organización permite tener un encapsulamiento mucho más
controlado y atomico.

A medida de que el proyecto crece suele ser un gran beneficio.

### Visibilidad avanzada

Dependiendo nuestro caso de uso podremos utilizar más o menos especificidad a
la hora de trabajar con la visibilidad de nuestro código en Rust, en el ejemplo
anterior vimos que en el archivo `services/auth.rs` hay varias firmas distintas.

Por defecto en Rust todo el código que creemos es privado.

Dentro de `services/auth.rs` al declarar algo como `pub` estamos permitiendo
que se pueda utilizar a nivel carpeta/módulo `services`.

Al hacer que `services/mod.rs` también exporte el modulo de `auth` estamos 
diciendo que un nivel superior puede acceder a `services/auth` y todo lo `pub`
dentro.

A si mismo si estuviéramos haciendo una libreria (adelantandonos a algo que
veremos más adelante) y la libreria en el punto de entrada también exporta ese
modulo, en este caso `services`, lo que sucederia es que desde donde se utilice
la libreria podremos acceder a todo lo `pub` dentro de `nuestra_lib/services/auth`.

También podemos especificar un poco más declarando si es publico en distintos
niveles, por ejemplo, `pub(super)` representa que solo sera accesible o publico
dentro del modulo padre, es decir todo lo que este dentro de `services` tendra
acceso a lo declarado como `pub(super)`.

Otra visibilidad que puede resultar útil es declarar algo como `pub(crate)` de
esta forma restringiremos la visibilidad de lo publico al nivel de nuestro 
proyecto, si diseñamos una libreria y usamos `pub(crate)`, lo que hayamos 
declarado no podra ser utilizado de forma externa a diferencia de un `pub`
convencional.

#### Visibilidades muy poco frecuentes

Veamos otras visibilidades menos frecuentes:

1. Cambiando de perspectiva existe la posibilidad de encontrarnos con la 
visibilidad `pub(self)`, es muy poco frecuente, es una visilidad muy restrictiva
se usa para explicitar la intención de que no debe ser accesible desde fuera
del modulo/archivo actual.

2. Tenemos una forma para nada frecuente pero puede resultar útil en
algún contexto especifico que es `pub(in path)`, donde `path` puede ser el 
nombre de un modulo o una ruta hacia un modulo, de esta forma estamos 
declarando que sera visible especificamente ese `path`.


# ¿Entonces?

Podríamos resumirlo en que el enfoque de Rust implica ser más explicitos, y nos
da multiples herramientas para trabajar nuestro código. Si bien se abordo mucho
las practicas más comunes suelen ser menos, es decir, se suele abordar el 
código:
- Usando carpetas y archivos `mod.rs`
- Se suelen remover los namespaces eliminar
    - Excepto cuando hay ambiguedad y no estamos usando alias
- Las visibilidades más frecuentes son privado (es decir sin visibilidad 
  definida), `pub` y `pub(crate)`
    - En algunos casos puede llegar a ocurrir el tener que usar `pub(super)`
    - Otros tipos de visibilidades son extremadamente frecuentes.

Todo esto es extremadamente positivio en proyectos medianos y grandes, nos
permite tener un control muy detallado de lo que estamos haciendo, como 
modelamos y como crear APIs de libreria que aporten a la experiencia de 
desarrollo.

[about-namespaces]: #un-poco-acerca-de-los-namespaces
