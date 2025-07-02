# Manejo de errores

El manejo de errores en Rust es nuevamente algo distinto al de Go, comparten 
algunas similitudes pero el scope de Rust es mucho más amplio.

En esta sección veremos algunos de los contrastes pero no todos, más adelante
veremos manejo de errores de forma avanzada.

## Errores Irrecuperables

Ambos lenguajes poseen algo llamado `panic` pero sin embargo se tratan de manera
distinta.

### 🐹 En Go, panic

Detiene la ejecución del hilo actual (puede ser goroutine) y puede ser 
recuperado desde un `defer` usando `recover()`.

```go
#package main
#
#import "fmt"
#
func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("¡Recuperado del panic!", r)
        }
    }()

    panic("¡Error fatal!")
}
```

Sin embargo este modelo de errores puede resultar relativamente costoso,
No está optimizado para flujos normales de control, está pensado para 
situaciones excepcionales, no para control de errores.

Cuando ocurre un panic, el runtime de Go captura una pila de llamadas completa 
(stack trace), ejecuta los `defer` en orden inverso.

Además si hay un `recover`, detiene la propagación, si no, finaliza la goroutine.

La propagación del estado interno del panic también puede ser relativamente 
costosa.

Lanzar un `panic` y atraparlo con `recover` puede costar entre 10x y 100x más 
que retornar un error normalmente.

En bucles su uso es bastante desaconsejado porque puede generar problemas 
serios.

Hay algunos casos validos donde en Go se sugiere usar `panic`:

- Librerías que deben abortar en casos imposibles.
- En tests o mocks donde querés simular una falla brutal.

### 🦀 RUST: `panic!` ligeros

En Rust también tenemos una forma de usar un `panic` como dijimos antes pero 
suele ser más ligero en tema de costos principalmente por razones de diseño, 
control del programador, y modelo de ejecución.

Profundizaremos más acerca de esto adelante pero para que se lleven una idea 
ahora:
- No genera traceback por defecto para ser "recuperable", en parte porque no hay
  `defer` y no usas algo como `recover()` y es más lineal la ejecución, entonces 
  Rust no necesita mantener estructuras para recuperar el control como Go pero 
  sí imprime backtrace si se establece `RUST_BACKTRACE=1`, esta variable de 
  entorno hace que se calcule sólo cuando se va a imprimir, no al hacer `panic!`
- Rust sigue RAII mientras que Go usa un poor man's RAII, lo que simplifica el 
  como destruir las instancias, y con el trait `Drop` (el cual se implementa por
  defecto en todo) garantizan limpieza sin `defer`
- Rust tiene configuraciones en tiempo de compilación para entender cual sera el
  comportamiento por defecto, si sera `panic = "abort"` o `panic = "unwind"`,
  con el `unwind` es más sutil y libera la memoria progresivamente, mientras
  que el `abort` puede ser más violento, directamente termina el programa, cero 
  overhead.

En Rust, la macro `panic!` no es parte del flujo normal del programa, así 
que el compilador puede optimizar mejor todo el código alrededor.

Por último si, generalmente se considera al `panic!` como un tipo de error 
irrecuperable en Rust, sin embargo tenemos una forma de hacerlo si queremos,
aunque no es la forma más idiomática, que es mediante la función `catch_unwind`
la cual nos permitirá manejar un `panic!` como un tipo de dato que veremos más 
adelante, el `Result`, y con ello podremos administrar el error como si fuese
otro error típico de nuestra código, bastante útil para algunos casos 
específicos.

Veamos esta comparativa para que quede más claro:

| Característica           | Rust `panic!`                     | Go `panic` + `recover`           |
| ------------------------ | --------------------------------- | -------------------------------- |
| ¿Backtrace automático?   | ❌ Solo si `RUST_BACKTRACE=1`      | ✅ Siempre capturado internamente |
| ¿Recuperación integrada? | ❌ No por defecto (`catch_unwind`) | ✅ Con `recover()` desde `defer`  |
| ¿Limpieza de recursos?   | ✅ Con `Drop` (RAII)               | ✅ Con `defer`                    |
| ¿Costo relativo?         | ⚙️ Bajo (abort) / Medio (unwind)  | 🚨 Alto (defer + recover + pila) |
| ¿Se puede desactivar?    | ✅ Sí (`panic = "abort"`)          | ❌ No                             |

El caso más simple si queremos que algo falle es:

```rust
fn main() {
    panic!("¡Esto explotó!");
}
```

Esto al ejecutarlo nos dara el error `¡Esto explotó!` y nos dice que si 
agregamos el `RUST_BACKTRACE=1` obtendremos más información.

Solo para ejemplificar un poco más en el siguiente ejemplo si se desea ejecutar
mostrara el backtrace de forma resumida porque para este ejemplo hemos
establecido la variable de entorno `RUST_BACKTRACE` en `1`:

```rust
#use std::env;
#
fn main() {
#    env::set_var("RUST_BACKTRACE", "1");
    funcion_peligrosa();
    println!("¡Este mensaje no sera mostrado porque falla antes!")
}

fn funcion_peligrosa() {
    panic!("¡Esto explotó!");
}
```

#### Diseccionando el backtrace

Vamos a revisar un poco el mensaje que nos da la ejecución del ejemplo anterior 
solo para darte algunas herramientas en caso de ver este mensaje en alguna 
ocasión:

```sh
¡Esto explotó!   # 👈 Nuestro mensaje de error
stack backtrace:
   0: std::panicking::begin_panic   # 👈 Aquí comienza el panico
             at /rustc/6b00bc3880198600130e1cf62b8f8a93494488cc/library/std/src/panicking.rs:769:5
   1: playground::funcion_peligrosa # 👈 Ocurrio dentro de funcion_peligrosa
             at ./src/main.rs:9:5
   2: playground::main              # 👈 funcion_peligrosa fue llamada en el main
             at ./src/main.rs:5:5   # 👈 y especificamente en la linea 5 
                                    # nosotros ocultamos código solo para que puedas
                                    # ver el error pero si, se llama en la linea 5
   3: core::ops::function::FnOnce::call_once
             at ./.rustup/toolchains/stable-x86_64-unknown-linux-gnu/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

Por ultimo nos dice que podemos usar `RUST_BACKTRACE=full` para mostrar el 
backtrace completo, el cual seria mucho más largo e indicaria más detalles pero
al mismo tiempo seria mucho más complicado de leer para alguien que comienza.

#### Recuperando un `panic!`

```rust
#use std::panic;
#
fn main() {
    let resultado = panic::catch_unwind(|| {
        println!("¡Va a ejecutar algo que falla!");

        panic!("¡Ups fallo algo!");
    });

    match resultado {
        Ok(_) => println!("No hubo pánico."),
        Err(_) => println!("¡Se capturó un panic!"), // Podríamos recuperar el mensaje
    }

    println!("¡Pero la ejecución continuo!")
}
```

De esta forma estamos manejando el posible panico.

---

## Errores recuperables

Cuando hablamos de "Errores recuperables" nos referimos a errores son errores 
esperables como pueden ser errores del tipo archivo no encontrado, conexión 
rechazada, división por cero, etc.
No son bugs, sino situaciones previstas que el programa puede manejar.

### 🐹 En Go

Go se usa una convención: las funciones devuelven `(T, error)` donde `T` es el 
tipo de retorno y `error` es un tipo de error que representa el error
ocurrido, si no hay error, el valor de `error` es `nil`.
Esto es una convención que se sigue en todo el lenguaje, no es obligatorio pero
es la forma más común de manejar errores.
Si la función falla, retorna un error, si no falla, retorna el valor esperado
y `nil` como error.
Esto es una forma de manejar errores que es bastante simple y directa, pero
puede resultar un poco verbosa y repetitiva.
Por ejemplo, si tenemos una función que divide dos números, podría verse así:

```go
#package main
#
#import "fmt"
#
func dividir(a, b int) (int, error) {
    if b == 0 {
        return 0, fmt.Errorf("división por cero")
    }
    return a / b, nil
}
#
#func main() {
#    resultado, err := dividir(10, 0)
#    if err != nil {
#        fmt.Println("Error:", err)
#        return
#    }
#    fmt.Println("Resultado:", resultado)
#}
```

Donde si `b` es cero, retornamos un error, si no, retornamos el resultado
y `nil` como error.

### 🦀 En Rust

Rust usa el tipo `Result<T, E>` para manejar errores recuperables, donde `T` es 
el tipo de retorno y `E` es el tipo de error.
El tipo `Result` es un enum que puede ser `Ok(T)` o `Err(E)`:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Lo que permite manejar errores de forma más explícita y segura, nos garantiza 
seguridad, o falla o nos devuelve el resultado esperado pero imposibilita el
obtener ambos valores.
Por ejemplo, si tenemos una función que divide dos números, podría verse así:

```rust
fn dividir(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err("División por cero".to_string())
    } else {
        Ok(a / b)
    }
}
#
#fn main() {
#    match dividir(10, 0) {
#        Ok(resultado) => println!("Resultado: {}", resultado),
#        Err(e) => println!("Error: {}", e),
#    }
#}
```

En este caso, si `b` es cero, retornamos un `Err` con un mensaje de error,
si no, retornamos un `Ok` con el resultado de la división.

Es más explicito el path al decir que algo es un error, y no es necesario
retornar un valor adicional como en Go, ya que el `Result` ya nos indica si
hubo un error o no.

En Go funciona como una tupla en Rust como un enum.

Sin embargo en Rust los errores no se pueden ignorar, si no se maneja el 
`Result`, el compilador nos dará un error de compilación, lo que nos obliga
a manejar los errores de forma explícita.

Este es el manejo de errores más simple en Rust, y se encuentra en esta sección
porque es facilmente comparable con el manejo de errores en Go.

Sin embargo, Rust tiene un sistema de manejo de errores mucho más avanzado y
flexible que permite manejar errores de forma más eficiente, permitiendo 
propagación (una de las funcionalidades más queridas en Go), conversiones de 
tipo y dando seguridad en todos los casos, todo esto lo veremos en la sección
de Manejo de Errores Avanzados.

### Comparativa de manejo de errores

Aquí una tabla comparativa entre Rust y Go para el manejo de errores

|                          | **Rust**                          | **Go**                            |
| ------------------------ | --------------------------------- | --------------------------------- |
| ¿Cómo se manejan?        | `Result<T, E>`                    | `(T, error)`                      |
| ¿Qué tipo de error?      | `enum Result<T, E>`               | `error` (interfaz)                |
| ¿Obliga a manejarlo?     | ✅ Sí (el compilador lo exige)    | ❌ No (convención `if err != nil`)|
| ¿Qué pasa si lo ignorás? | ⚠️ Warning o error de compilación | 🚫 Nada, se puede ignorar         |
| ¿Propagación de errores? | ✅                                | ✅ Con `return` o `defer`         |


