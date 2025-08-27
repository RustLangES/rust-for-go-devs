# Mutabilidad Interna

Cuando vimos `Box`, entendimos cómo Rust controla la propiedad de un valor en el 
heap.
El siguiente paso es entender cómo Rust nos permite mutar valores incluso cuando 
no tenemos acceso mutable directo: eso es lo que llamamos mutabilidad interna.

En Rust, el sistema de tipos y el sistema de préstamos (borrowing) son tan 
poderosos que nos permiten tener mutabilidad incluso cuando el valor no es 
mutable en sí mismo.

Puede parecer contradictorio, pero es una característica intencional del 
lenguaje que nos permite escribir código seguro y concurrente sin sacrificar la
flexibilidad.

En Go seria imposible mutar un valor a través de una referencia inmutable, pero en Rust

En Go esto no existe como concepto formal. En Go uno simplemente pasa un puntero 
y puede mutar el valor, porque el lenguaje permite `aliasing libre` y no impone 
restricciones de préstamo o propiedad, no tenemos `trait bounds` que nos ayuden 
a controlar el acceso a los datos, ni delimitar las posibilidades de un objeto.

## Aliasing

```go
package main
import "fmt"
func main() {
    x := 10
    p := &x   // p apunta a x
    q := &x   // q también apunta a x
    *p = 20
    fmt.Println(x, *q) // imprime 20 20
}
```

Aquí `x` tiene aliasing, porque `p` y `q` son dos referencias que pueden mutar 
el mismo valor.

Es decir el termino `aliasing` se refiere a que el lenguaje permite tener varias 
referencias mutables o inmutables al mismo tiempo, sin restricciones.

En Go, puedes tener tantos punteros como quieras apuntando al mismo valor y 
mutarlos a voluntad.

Esto es flexible, pero también arriesgado, porque puedes crear condiciones de 
carrera si accedes desde múltiples threads sin locks u otros tipos de problemas
con respecto a la consistencia.

### ¿Cómo maneja esto Rust?

Rust no permite aliasing libre para referencias mutables

Solo una referencia mutable (`&mut T`) puede existir a la vez, o varias 
referencias inmutables (`&T`) pueden coexistir, pero no puedes combinarlas con 
una mutable como hemos visto en capítulos anteriores.

Pero hay casos en donde quizás podamos necesitar un mecanismo similar al 
aliasing, no exactamente lo mismo porque no queremos que sea tan libre, pero
sí que nos permita cierta flexibilidad para que algunos casos, como caches,
contadores, o estructuras de datos complejas, en esos casos deberíamos poder 
mutar su estado interno incluso cuando no tenemos acceso mutable directo.

Quizás queremos implementar un comportamiento en que el usuario de nuestra
estructura no necesite preocuparse por si es mutable o no, y que internamente
podamos cambiar su estado.

Es aquí donde conceptos previamente vistos como `Smart Pointers` y 
`trait bounds` nos ayudan a implementar este patrón de `mutabilidad interna`.

Esto es algo que Go resuelve "liberando" la mutabilidad, mientras que Rust lo 
resuelve "restringiendo" y pidiendo al compilador que controle la seguridad.

## Dos nuevos Smart Pointers

Cuando exploramos `Box`, entendimos que Rust cuida celosamente quién es el dueño 
de un valor en el `heap`. Esa propiedad nos da seguridad, nadie puede tocar lo 
que no le pertenece. Pero la vida real, y nuestros programas, no siempre son tan
rígidos. A veces necesitamos un poco de flexibilidad, queremos cambiar cosas 
aunque no tengamos permiso explícito, al menos en apariencia.

### Cell

Rust nos ofrece esta ventana de libertad cuidadosamente medida. Primero, con 
`Cell`, un contenedor pequeño y sencillo. Nos permite alterar valores simples, 
incluso cuando la variable que los contiene parece inmutable. Es como si Rust 
nos dijera: "Sí, puedes tocar esto, pero solo si sabes lo que haces y no 
compartes el juguete con otros al mismo tiempo".

Rust no nos deja hacer esto con cualquier cosa. `Cell` es para tipos que son
`Copy`, es decir, tipos simples como números o booleanos. Si intentamos usarlo
con algo más complejo, Rust nos detendrá en seco, también nos prohibe de usarlo
en contextos concurrentes, porque ahí la seguridad es aún más crítica.

Rust logra esto nuevamente utilizando su sistema de tipos y `trait bounds`. 
`Cell` implementa el `!Sync` trait, lo que significa que no puede ser compartido 
entre threads. Esto nos protege de condiciones de carrera y otros problemas que 
podrían surgir si varios threads intentaran mutar el mismo valor al mismo 
tiempo.

Cuando veamos más adelante el capítulo de concurrencia, entenderemos mejor esto.

Veamos un ejemplo simple:

```rust
use std::cell::Cell;

fn main() {
    let x = Cell::new(5);
    x.set(10);
    println!("{}", x.get()); // imprime 10
}
```

Como veremos de forma similar que con `Box` debemos de importarlo desde el 
módulo y luego podemos crear una `Cell` con un valor inicial, nuevamente 
`Cell` es un `Smart Pointer`, el valor se almacena en el heap y `Cell` mismo 
controlara la propiedad y el acceso a ese valor, es por eso que una vez 
inicializado `x` es inmutable, pero internamente podemos cambiar su valor con
el método `set`, y obtener su valor con `get`, el que validara si es posible 
modificar el valor o no sera el propio compilador.

Esta estructura es considera de una abstracción de coste cero 
(zero-cost abstraction), porque el compilador optimiza el código para que no 
haya sobrecarga en tiempo de ejecución, es decir, el código generado es tan
eficiente como si hubiéramos manipulado el valor directamente.

### RefCell

La realidad nos demuestra que rara vez la mutabilidad que buscamos se reduce a 
tipos de datos simples como enteros o booleanos. Nosotros muchas veces 
buscaremos mutar estructuras más complejas, para estructuras más complejas, 
surge `RefCell`, que nos da un poco más de libertad: podemos pedir prestadas 
referencias mutables a datos incluso si el contenedor parece inmutable. Rust ya 
no puede verificar todo en compile time, así que hace un chequeo en runtime: si 
intentamos ser demasiado ambiciosos y pedir mutaciones conflictivas, Rust nos 
detiene antes de que causemos estragos.

```rust 
use std::cell::RefCell;

fn main() {
    let data = RefCell::new(vec![1, 2, 3]);
    match data.try_borrow_mut() {
        Ok(mut_ref) => {
            mut_ref.push(4);
        }
        Err(_) => {
            println!("No se pudo obtener una referencia mutable");
        }
    }
    let borrowed = data.try_borrow();
    match borrowed {
        Ok(ref_ref) => {
            println!("{:?}", ref_ref); // imprime [1, 2, 3, 4]
        }
        Err(_) => {
            println!("No se pudo obtener una referencia inmutable");
        }
    }
}
```

Aquí, `RefCell` nos permite mutar un vector incluso cuando `data` es
inmutable. Usamos `try_borrow_mut` para obtener una posible referencia mutable 
al vector y si nos lo permite el lenguaje podemos modificarlo. Si intentamos 
pedir prestadas múltiples referencias mutables al mismo tiempo, obtendremos un 
error.

El comportamiento de `RefCell` se basa en el conteo de referencias en tiempo de
ejecución. Mantiene un registro de cuántas referencias inmutables y mutables
están activas. Si intentamos pedir prestada una referencia mutable mientras hay
referencias inmutables activas, o viceversa, `RefCell` nos impedirá hacerlo, 
lanzando un pánico en tiempo de ejecución en el caso de usar `borrow_mut` o
`borrow` o retornando un `Err` en el caso de usar `try_borrow_mut` o
`try_borrow`.

Rust permite ambos manejos de errores pero no nos permite ignorar el error, 
porque eso rompería la seguridad que Rust nos ofrece, es preferible mostrar
un error en tiempo de ejecución que permitir un comportamiento indefinido los 
cuales suelen ser la causa de bugs muy difíciles de rastrear.

El caso de arriba es solo una ejemplificación simple, pero veamos una 
abstracción más realista:

{{#tabs }}
{{#tab name="main.rs" }}

```rust
mod client;
use client::Client;
use std::thread::sleep;

fn main() {
    let limiter = Client::new();

    for i in 0..=5 {
        match limiter.execute_request() {
            Ok(_) => println!("Request {i} ejecutada con éxito"),
            Err(e) => println!("Error en request {i}: {e}"),
        }
        sleep(Duration::from_secs(1));
    }

    println!("Uso actual: {}", limiter.current_usage());
}
```

{{#endtab }}
{{#tab name="client.rs" }}

```rust
use std::cell::RefCell;
use std::time::{Duration, Instant};

struct Client {
    limit: i32,
    usage: RefCell<i32>, // mutabilidad interna
    last_request: RefCell<Option<Instant>>, // cooldown interno
}

impl Client {
    fn new() -> Self {
        Client {
            limit: 5,
            usage: RefCell::new(0),
            last_request: RefCell::new(None),
        }
    }

    fn execute_request(&self) -> Result<(), String> {
        // Intentamos actualizar el cooldown
        let Ok(mut last) = self.last_request.try_borrow_mut() else {
            return Err("No se pudo acceder al cooldown".to_string());
        };

        let now = Instant::now();

        if let Some(prev) = *last {
            if now.duration_since(prev) < Duration::from_secs(1) {
                return Err("Cooldown activo, espera antes de enviar otra request".to_string());
            }
        }
    
        // actualizamos el cooldown
        *last = Some(now);

        let Ok(mut usage) = self.usage.try_borrow_mut() else {
            return Err("No se pudo ejecutar la request".to_string());
        };

        if *usage < self.limit {
            *usage += 1; // incremento secreto
            // Se ejecuto
            Ok(())
        } else {
            Err("Límite de requests alcanzado".to_string())
        }
    }

    fn current_usage(&self) -> i32 {
        match self.usage.try_borrow() {
            Ok(u) => *u,
            Err(_) => {
                println!("No se puede leer usage ahora");
                0
            }
        }
    }
}
```

{{#endtab }}
{{#endtabs }}

<div class="info play ferris-scientific">
<p>Link de ejemplo completo en <a href="https://www.rustexplorer.com/b/z00dc1">Rust Explorer</a></p>
</div>

En este ejemplo podemos ver cómo `Client` usa `RefCell` para manejar su estado
interno de manera segura. Aunque `Client` es inmutable desde el exterior, puede
mutar los atributos `usage` y `last_request` internamente gracias a `RefCell`.

Es una forma elegante de encapsular la mutabilidad, permitiendo que el usuario 
de `Client` no tenga que preocuparse por si es mutable o no, mientras que
internamente `Client` puede gestionar su estado de manera segura y controlada.

Esta es una de las ventajas de usar Smart Pointers, podemos crear abstracciones
más complejas y seguras, aprovechando el sistema de tipos y el control de
préstamos de Rust.

Así es como podemos mantener un estado interno mutable, sin requerir al usuario
de nuestra estructura que tenga que lidiar con la mutabilidad directamente.

## ¿Por qué es útil esto?

Va a ser muy útil en varios escenarios como los mencionados antes, pero 
es realmente útil a la hora de hacer abstracciones más complejas, porque 
nos permite que el usuario no se preocupe que se modifica de la variable,
si tuvieramos que declarar todo como mutable, el usuario final tendría que
tener un mayor conocimiento de cómo funciona nuestra estructura, y eso
rompería la encapsulación.

En este caso nosotros no le pedimos nada al usuario, el usuario simplemente
crea un `Client` y ejecutara las request y el `Client` internamente se encargara
de gestionar su estado.

Son abstracciones inteligentes que nos permiten escribir código más limpio y 
seguro.

---

En el proximo capítulo veremos cómo Rust maneja la concurrencia, y veremos que
estas abstracciones son aún más valiosas en ese contexto, tendremos otra 
abstracción que nos permitirá mutar datos de forma segura incluso en presencia 
de múltiples threads.

Asegurándonos de que no haya condiciones de carrera y nuevamente manteniendo la
consistencia de los datos.

