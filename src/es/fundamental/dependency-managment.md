# Gestión de Dependencias

Si vienes del ecosistema de Go, estás acostumbrado a organizar tus proyectos en
paquetes (`package`) y usar `go.mod` para definir dependencias. En Rust, la 
organización y gestión de dependencias sigue un enfoque similar pero con 
herramientas y convenciones.

## ¿Qué es un crate?

En Rust, el termino `crate` lo podriamos traducir como caja, cajon. 
Los crates puedes imaginarlo de forma similar a los modulos o paquetes de Go,
pero tienen algunas pequeñas diferencias, por ejemplo:
- Todo proyecto en Rust es, al menos, un crate
- Un crate puede ser un proyecto binario (ejecutable) o puede ser una 
  biblioteca/libreria, lo cual permite que tu código sea reutilizable
- Crates pueden depender de otros Crates, algunos oficiales o de terceros

Hasta ahora venimos hablando de los proyectos mayoritariamente como proyectos 
binarios, los proyectos binarios son los que poseen un archivo con la función
`main`.

Un crate binario generalmente tiene un archivo `main.rs`, mientras que las 
librerias en Rust generalmente tienen como punto de entrada para compilar el 
archivo `lib.rs`.

## Cargo la navaja suiza

Ya hemos hablado en el sección de [Tooling](./../quick-comparisons/tooling.md)
un poco acerca de Cargo.

En Go usamos `go build`, `go get`, `go test`, etc., como comandos separados, 
Rust unifica todas estas funcionalidades bajo Cargo.

## Cargo.toml

En Go podemos encontrar el archivo `go.mod` donde generalmente declaramos las 
dependencias, mientras que en Rust podemos encontrar el archivo `Cargo.toml`.
De forma similar tenemos `go.sum` que en Rust su equivalente seria `Cargo.lock`
el archivo donde quedan guardadas las resoluciones de paquetes que toma Cargo. 

En este archivo declaramos la gran mayoría de configuraciones de nuestro 
proyecto.


Declaramos las dependencias, nombre del crate, la versión, la edición de Rust 
que debemos usar, etc.

Veamos un ejemplo de como se ve:

```toml
[package]
name = "mi_crate"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = "1.0"
tokio = { version = "1.35", features = ["full"] }
```

En dependencies declararemos las dependencias, tenemos estas dos maneras de 
declarar la dependencia, de forma simplificada o deforma más selectiva.

Estos crates seran buscados en [crates.io][crates.io], el repositorio oficial
de dependencias, sin embargo podemos configurar un registry privado para 
utilizar un repositorio corporativo de dependencias al igual que también podemos
configurar la descarga de paquetes directamente desde un link de git.

La forma en que Cargo las dependencias terminaran siendo compiladas y cacheadas
dentro del directorio `target`.

### Diferencias

#### Features

En Rust las dependencias pueden contener features flags, estas features 
habilitan o deshabilitan secciones del codigo en las librerias.

Esto permite generar en el ecosistema librerias que son compatibles con 
multiples implementaciones de otras librerias, es decir.

Convencionalmente cuando elegimos una dependencia u otra una de las cosas que 
tenemos que tener en consideración es si se puede o no utilizar con alguna de 
las librerias que tenemos, con Rust eso no suele ser necesario porque esta
funcionalidad nos permite tener codigo plugeable.

### Opcionabilidad

Con el mismo mecanismo de features flags algo que podemos hacer es tener 
dependencias circunstanciales, es decir si estamos haciendo una libreria quizás
se de la situación de querer agregar dos dependencias que hacen exactamente lo 
mismo, sin embargo no queremos agregar tamaño adicional a nuestro binario final
o por lo menos no agregar tamaño durante la descarga del paquete.

Rust nos permite que mediante las feature flags señalar que dependendencias 
nuestra propia libreria deberia descargar.

### Ventajas con la resolución

Previamente hablamos de que las dependencias se pueden descargar desde un
repositorio centralizado como [crates.io], puede ser con un link de un 
repositorio de git, de forma local o con un registry privado.

Esto ayuda a la experiencia de desarrollo en algunos casos, han habido casos en 
Go con respecto a cambios de nombre en paquetes, repositorios o cambios de 
dueños, cuentas eliminadas y han ocasionado problemas.

Rust soluciona esto de esta forma, intentando que la mayor cantidad de 
dependencias puedan ser descargadas desde un repositorio centralizado.

En Go generalmente si dos módulos necesitan versiones distintas de una misma 
dependencia, el build falla, al igual que puede ocasionar que haya conflicto
entre versiones de dependencias.

Rust al estar asignando una versión para cada paquete y permitir compilaciones
entre ediciones, permitir que puedas utilizar dos versiones de la misma 
libreria, en definitiva tiene una resolución mucho más flexible.

### Integración con el resto del ecosistema

Cada dependencia que descargues se descarga con su correspondiente 
documentación, por lo que si ejecutas `cargo doc` puedes acceder a la 
documentación de tu proyecto y a si mismo a las dependencias que estes usando.

Si tu deseas publicar una dependencia con `cargo publish`, Cargo se encargara 
de validar la información del `Cargo.toml`, pero además ejecutara los tests y
deployara la documentación en [docs.rs].

Si tienes dudas de tus dependencias podras ejecutar `cargo audit` para auditar
fácilmente todo tu proyecto, además de esto Cargo ofrece extensibilidad por lo
que puedes con `cargo vet`, una extensión de Cargo, importar reglas y medidas
de seguridad que ofrecen empresas para sus propios proyectos, por ejemplo 
[Google publica sus auditorias][google-audits], estas auditorias las podrias
importar y con la misma rigurosidad con la que ellos validan su software tu 
estarias validando el tuyo, mismos estandares y reglas de seguridad extendidas
por diversas companias solo agregando unas propiedades en tu `Cargo.toml`.

## Agregar dependencias

Si bien podemos agregar las dependencias manualmente agregandolas a nuestro
`Cargo.toml` también podemos ejecutar el comando `cargo add` el cual intentara
descargar la dependencia y la agregara al `Cargo.toml` con todas las 
configuraciones que hayamos pasado por CLI, si tienes registries privados 
configurados priorizada estos registries por encima de [crates.io]

## Linkeo local

Algo que en multiples lenguajes se suele complicar es el linkear dependencias 
locales a nuestro proyecto, Rust nos permite esto simplemente configurando el 
path por ejemplo, nuestro `Cargo.toml` se veria algo así:

```toml
[dependencies]
mi_dependencia = { path = "../mi_dependencia" }
```

De forma en que retrocedera un directorio linkeara nuestra dependencia a la 
carpeta `mi_dependencia` la cual debe ser un crate.

La forma de enlazar dependencias localmente en Rust resulta muy comodo pero 
además otra practica común puede ser usar monorepositorios usando `workspaces`.
Pero esto ultimo lo veremos más adelante.



[crates.io]: https://crates.io
[docs.rs]: https://docs.rs
[google-audits]: https://github.com/google/rust-crate-audits

