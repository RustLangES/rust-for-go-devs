# Traits

Los `traits` en Rust son interfaces de comportamiento, permiten definir métodos
que deben implementar los tipos que quieran comportarse de cierta manera. Esto
puede resultar familiar de cierta forma, debido a que en Go tenemos las 
`interface` que puede llegar a sonar similar pero los `traits` van más lejos.

Un `trait` básico en Rust puede verse de la siguiente forma:

```rust,no_run
trait Describible {
    fn describir(&self) -> String;
}
```

Aquí estamos definiendo un trait llamado `Describible`, cualquier tipo que 
implemente `Describible` deberá de definir el método `describir`.

```go,no_run
type Describible interface {
    Describir() string
}
```

Seria quizás la contraparte en Go.

<div class="info">

Sin embargo cuando se implementan son bastante distintos en el sentido de 
diseño de lenguaje, Go parte de la idea de un `subtyping` o `structural typing`, 
mientras que Rust implementa traits de forma nominal. 

Esto quiere decir que Go asume que implementaste una `interface` solamente porque
el tipo de dato tiene un método/función asociada, no revisara que efectivamente
la interfaz sea exactamente la misma.

Rust por su contraparte si, en Rust puedes tener varios métodos/funciones 
asociadas con el mismo nombre y no representara necesariamente que estés
implementando el `trait`.

</div>

## Implementando `trait` en una `struct`

En Rust para implementar un `trait` en un `struct` de forma similar a cuando
implementamos un método en el capitulo de 
[Estructuras](./../quick-comparisons/structures.md#métodos-y-mutabilidad)
podemos hacer lo mismo con `trait`s:

```rust
struct Usuario {
    nombre: String
}

impl Describible for Usuario {
    fn describir(&self) -> String {
        format!("Usuario: {}", self.nombre)
    }
}
```

De esta forma se podría leer como implementar `Describible` para la estructura
`Usuario` y dentro escribimos la implementación.

Lo mismo en Go podría verse de la siguiente forma:

```go
type Usuario struct {
    Nombre string
}

func (u Usuario) Describir() string {
    return "Usuario: " + u.Nombre
}
```

## Métodos por defecto

En Rust algo que podemos hacer son las conocidas `Blanket implementations`, 
termino que se emplea para decir que podemos tener implementaciones por defecto,
para conseguir esto normalmente en Go debemos usar composición manual o 
interfaces vacías lo cual puede resultar menos cómodo.

En Rust eso se podría hacer de la siguiente manera en la declaración del 
`trait`:

```rust
trait Describible {
    fn describir(&self) -> String {
        "Sin descripción".to_string()
    }
}
```

De esta forma si implementamos el `trait` y no sobre escribimos el método 
tendremos un fallback, una implementación por default que podremos usar, si no
estamos interesados en sobre escribir el método podemos simplemente 
implementarlo de la siguiente forma:

```rust
#trait Describible {
#    fn describir(&self) -> String {
#        "Sin descripción".to_string()
#    }
#}
#
struct Telefono {
    modelo: String
}

impl Describible for Telefono {}
#

fn main() {
    let telefono = Telefono { modelo: "Nokia 1100".to_string() };   
    
    println!("{}", telefono.describir());
}
```

Puede llegar a ser útil en algunos casos en los que no nos interese 
reimplementar cosas.


## Orphan Rules

En Rust se puede implementar un trait sobre tipos de otros módulos, con algunas 
restricciones pero este mecanismo permite un sistema de tipos elástico, muy 
flexible, un ejemplo clásico es el siguiente:

```rust
impl Describible for i32 {
    fn describir(&self) -> String {
        format!("Entero: {}", self)
    }
}

fn main() {
    println!("{}", 2.describir());
}
```

De esta forma ahora todos los `i32` en este modulo tienen implementado 
`Describible` y por lo tanto tienen el método `describir`.

Es decir, estamos implementando un `trait` en un tipo de dato externo, un tipo
de dato perteneciente al propio lenguaje. 

Estamos modificando una parte del lenguaje.

Y es por estas reglas que el lenguaje permite ser tan extensible, sin embargo
a pesar de que digamos que es un lenguaje flexible, las orphan rules tienen
como principal limitación que requiere que alguna de las dos cosas pertenezcan
en nuestro dominio, en nuestra propiedad y sean definidos en nuestro crate.

Es decir:
- Si el `trait` esta definido en nuestro crate podemos implementarlo para 
  las estructuras o enums que sean de nuestro crate o de terceros
- Podemos implementar un `trait` externo si la estructura o enum nos pertenece, 
  es decir, está en nuestro crate

Otra forma de verlo es de una manera más formal:

```rust,noplayground
impl Trait for Type
```
Esto es posible solo si:
- `Trait` esta definido en nuestro crate
- `Type` esta definido en nuestro crate

> Si ambos (el `trait` y el tipo) son externos, no puedes implementar el trait 
para este tipo.


### ❌ Ejemplo Invalido 

```rust
use std::net::IpAddr;
use std::string::ToString;

impl ToString for IpAddr {
    fn to_string(&self) -> String {
        format!("Dirección IP personalizada: {}", self)
    }
}
```

**❌ Invalido**: Tanto `ToString` como `IpAddr` son externos, esto viola las 
orphan rules

### Eludiendo las reglas

Puede darse el caso en donde tu haciendo código más avanzado necesites 
implementar un `trait` externo para un tipo externo, es decir, implementar
un `trait` de una libreria para una estructura o un enum de una libreria.

En tal caso veremos más adelante algunas formas comunes de como lograr esto en 
el capitulo acerca de Newtypes.