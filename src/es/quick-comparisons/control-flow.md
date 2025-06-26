# Control de flujo

Este capitulo sera separado en [Ejemplos rápidos](#ejemplos-rápidos) y en 
[Lectura](#lectura) veremos más detalladamente en que se diferencian Rust y Go
con respecto a este tema.

## Ejemplos rápidos

Todos los controles de flujo son considerados expresiones en Rust, por tanto,
podemos asignarlos a variables o retornarlos de forma directa.

### If & Else

| Go                          | Rust                         |
| --------------------------- | ---------------------------- |
| `if` no necesita paréntesis | Igual, pero **es expresión** |
| Bloques obligatorios        | Igual                        |
| Sin valor de retorno        | Puede devolver valor         |

En Go suele representarse el control de flujo de esta forma:

<table>
    <thead>
        <tr>
            <td>X = 0</td>
            <td>X = 1</td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>

```go
# package main
# 
# import (
# 	"fmt"
# )
# 
# func main() {
#   x := 0
    if x > 0 {
        fmt.Println("positivo")        
    } else {
        fmt.Println("negativo")        
    }
# }
```

</td>
                <td>

```go
# package main
# 
# import (
# 	"fmt"
# )
# 
# func main() {
#   x := 1
    if x > 0 {
        fmt.Println("positivo")        
    } else {
        fmt.Println("negativo")        
    }
# }
```

</td>
</tr>
</tbody>
</table>



Rust lo representaría de forma similar:

<table>
    <thead>
        <tr>
            <td>X = 0</td>
            <td>X = 1</td>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>

```rust
    #let x = 0;
    # 
    if x > 0 {
        println!("positivo");        
    } else {
        println!("negativo");        
    }
```

</td>
                <td>

```rust
    # let x = 1;
    #
    if x > 0 {
        println!("positivo");        
    } else {
        println!("negativo");        
    }
```

</td>
</tr>
</tbody>
</table>

Pero en Rust podemos además asignar el resultado de la expresión a una variable:

```rust
    #let x = 0;
    let mensaje = if x > 0 { "positivo" } else { "negativo" };
    #println!("{mensaje}")
```

### Switch vs Match

| Go                                                 | Rust                        |
| -------------------------------------------------- | --------------------------- |
| `switch` es flexible                               | `match` es **exhaustivo**   |
| `fallthrough` explícito <sup>1</sup> <sup>2</sup>  | No existe `fallthrough`     |
| No error si faltan casos                           | Error si no se cubren todos |

- <sup>1</sup> Con fallthrough nos referimos a que el flujo de ejecución 
  continúa al siguiente caso sin necesidad de un `break` explícito.
- <sup>2</sup> En Go, `switch` no requiere `break` para evitar el `fallthrough`, 
    pero si se desea, se puede usar `fallthrough` como keyword
    para obtener ese comportamiento explícitamente.


Ejemplo en Go:

```go
#package main
#
#import (
#	"fmt"
#)
# 
#func main() {
#    day := "lunes"
    switch day {
        case "lunes":
            fmt.Println("Inicio de semana")
        case "viernes":
            fmt.Println("Casi fin")
        default:
            fmt.Println("Otro día")
    }
#}
```

En Rust, el equivalente sería:

```rust
#     let day = "lunes";
    match day {
        "lunes" => println!("Inicio de semana"),
        "viernes" => println!("Casi fin"),
        _ => println!("Otro día"),
    }
```

### Bucles

| Go                    | Rust                             |
| --------------------- | -------------------------------- |
| `for` único tipo      | `for`, `while`, `loop` separados |
| `for {}` = while true | `loop {}` explícito              |

#### Bucles for

En Go podríamos hacer lo siguiente:

```go
#package main
# 
#import (
#	"fmt"
#)
# 
#func main() {
    for i := 0; i < 5; i++ {
        fmt.Println(i)
    }
#}
```

En Rust de forma similar podríamos conseguir el mismo efecto:

```rust
    for i in 0..5 {
        println!("{}", i);
    }
```

#### Bucle Infinito

En Go un bucle infinito podríamos hacerlo de la siguiente forma:

```go,no_run
    for {
        fmt.Println("infinito")
    }
``` 

Mientras que en Rust usaríamos:

```rust,no_run
    loop {
        println!("infinito");
    }
```

#### Bucles con While

Go no tiene bucle `while` pero en Rust si tenemos.

```rust
    let mut i = 0;
    while i < 5 {
        println!("{}", i);
        i += 1;
    }
```

El bucle `while` puede resultar útil cuando buscamos una condición de corte
quizás no tan clara, por ejemplo ejecutar un bucle de código hasta que se 
termine de leer un stream de datos, condiciones en que no tengamos que usar 
contadores.

#### `break` y `continue`

Ambos lenguajes los tienen, pero Rust permite `break` con valor en `loop`:

```rust
    let mut x = 0;

    let resultado = loop {
        if x > 10 {
            break x * 2; // aquí
        }
        x += 1;
    };

    println!("{resultado}");
```

De esta forma se asigna el valor del `break` dentro de `resultado`.

Eso no existe en Go.

---

### Comparación rápida entre los bucles de Rust

| Caso                                         | Mejor usar       | ¿Por qué?                                |
| -------------------------------------------- | ---------------- | ---------------------------------------- |
| Iterar de 0 a N                              | `for`            | Más claro y compacto                     |
| Esperar condición que no depende de contador | `while`          | Más expresivo para condiciones dinámicas |
| Leer hasta terminar input o streams de datos | `while`          | El número de iteraciones es desconocido  |
| Ejecutar al menos una vez siempre            | `loop` + `break` | O en Go: `for {}` con `break` manual     |

</br>

---

### `if let` y `match` como pattern matching (solo Rust)

Rust permite patrones destructurantes muy potentes que no existen en Go.

```rust
    let punto = (0, 5);

    if let (0, y) = punto {
        println!("Está sobre el eje Y en {y}");
    }
```

En este caso si `opcion` es equivalente a la variante `Some`, se extrae el valor
que contiene y se lo usa con el nombre de `x`. Además podemos agregarle un 
control `else` si queremos cubrir el caso en que no se cumpla la condición.

Si el primer valor de la tupla fuera `1`, no entraría en el bloque.

Con `match` podríamos hacer algo similar, pero en este caso debemos cubrir ambas 
variantes, el `match` es exhaustivo, sino tenemos en cuenta ambos casos
obtendremos un error en compilación:

```rust,editable
    let color = "azul";

    match color {
        "rojo" => println!("El color es rojo"),
        "verde" => println!("El color es verde"),
        otro => println!("Otro color, el cual es {otro}"), // <-- Eliminar
    }
```

Si has eliminado la linea, veras el error, esto es porque las variantes del tipo 
slice String son infinitas y solo estamos contemplando el caso en que sea
`"rojo"` o `"verde"`.

Rust nos sugiere agregar el caso por default que puede ser usando `_` o 
asignando el posible valor a una variable, como tenias previamente en la 
variable `otro` que posiblemente hayas eliminado.

Un caso donde tengamos un scope más pequeño podría ser usar pattern matching
para determinar rangos numéricos:

```rust,editable
    let mi_numero_favorito = 2;

    match mi_numero_favorito {
        0 | 1 => println!("Te gusta el binario!"),
        2 => println!("Simple"),
        3..=6 => println!("Un número entre 3 y 6 inclusive"),
        caso_raro if caso_raro % 2 == 0 => {
            println!("Un número par mayor a 6: {caso_raro}");
        },
        42 => println!("El sentido de la vida, el universo y todo lo demás"),
        ..0 => println!("Un número negativo 😭"),
        6.. => println!("De seis al infinito!")
    }
```

De esta forma nosotros sabemos que las posibilidades son desde el valor minimo 
de `i32` hasta el máximo, y podemos cubrir todos los casos, de forma en que 
no es necesario usar el valor por defecto para cubrir casos no contemplados.

Si removemos el caso del `0 | 1`, `2` o el rango del `3..=6`, Rust nos avisará
que no estamos cubriendo todos los casos posibles.

La exhaustiva en su máximo esplendor.