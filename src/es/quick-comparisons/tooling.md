# Tooling

Cuando se trata de herramientas en el ecosistema de Go provee un tooling más 
minimalista, mientras que el enfoque de Rust es más modular, mucho más 
completo y extensible. 

En Go, el comando `go` es la herramienta principal para compilar, ejecutar y 
administrar dependencias. Por ejemplo, para compilar un programa Go, simplemente 
se usa el comando `go build`, y para ejecutarlo, se usa `go run`. Además, Go 
tiene un sistema de gestión de dependencias integrado que permite descargar y
administrar bibliotecas de terceros de manera sencilla. Por ejemplo, para 
agregar una dependencia, se puede usar el comando `go get`, que descarga la 
biblioteca y la agrega al archivo `go.mod`.

En cambio, Rust utiliza `cargo` como su herramienta principal. `Cargo` es un
sistema de gestión de paquetes y compilación que permite compilar, ejecutar y
administrar dependencias de manera más avanzada. Por ejemplo, para compilar un
proyecto Rust, se usa el comando `cargo build`, y para ejecutarlo, se usa
`cargo run`. Además, `Cargo` permite crear y administrar proyectos de manera
estructurada, con un archivo `Cargo.toml` que define las dependencias y la
configuración del proyecto. También permite crear bibliotecas y paquetes
reutilizables, lo que facilita la compartición de código entre proyectos.
Si deseas agregar una dependencia en Rust, se puede hacer editando el archivo
`Cargo.toml` y agregando la dependencia en la sección `[dependencies]`. Por
ejemplo, para agregar la biblioteca `serde`, se puede agregar la siguiente línea:

```toml
[dependencies]
serde = "1.0"
```

Fácil ¿no? Mientras que Go al no tener un sistema centralizado de paquetes y 
debes ingresar la URL del repositorio de la biblioteca que deseas
utilizar, Rust tiene un registro centralizado de paquetes llamado
[crates.io](https://crates.io/), donde puedes buscar y encontrar
bibliotecas de terceros. Esto facilita la búsqueda y el uso de bibliotecas
en proyectos Rust, ya que puedes encontrar fácilmente las dependencias que
necesitas y agregarlas a tu proyecto de manera sencilla.

Crates.io además proporciona una interfaz web para explorar y buscar
bibliotecas, lo que facilita la navegación y la búsqueda de paquetes
disponibles y da más visibilidad a las bibliotecas, sin embargo, hay
otras interfaces web que puedes utilizar para buscar
bibliotecas de Rust, como [Lib.rs](https://lib.rs/), que es una
alternativa a crates.io y ofrece una interfaz con más características, como
un sistema de etiquetas, una lista de dependencias, features y da una muchas 
[estadisticas sobre la comunidad](https://lib.rs/stats) de Rust, como el número 
de paquetes publicados mes a mes desde la creación de crates.io.

Esto ayuda a la mantenibilidad del proyecto, ya que el sistema de URLs de Go 
puede ser explicito pero propenso a errores si la URL cambia o si el
repositorio no está disponible. Ha ocurrido que algunas bibliotecas cambian
de dueño en Github o son eliminadas, lo que puede causar problemas en
proyectos que dependen de ellas. En cambio, Rust al tener un registro 
centralizado de paquetes, entre las políticas de crates.io, se asegura de que
las bibliotecas no se eliminen o cambien de nombre sin previo aviso, lo que
ayuda a mantener la estabilidad y la confiabilidad de las dependencias.

También se proporciona información acerca de crates que son dependientes de 
otros crates por ejemplo, lo que nos permite descubrir nuevas
bibliotecas que pueden ser útiles para nuestro proyecto que están planteadas de
base para ser compatibles con la biblioteca que estamos utilizando.

Otra maravilla de Rust es la funcionalidad de `cargo doc`, que genera
documentación para tu proyecto y sus dependencias de manera automática.
Esto es especialmente útil para mantener la documentación actualizada y
asegurar que esté disponible para otros desarrolladores que puedan
utilizar tu código. La documentación generada incluye ejemplos de uso,
descripciones de funciones y estructuras, y cualquier otra información
relevante que hayas incluido en tu código. Además, la documentación esta
se autogenerada a partir de los comentarios en el código, la disponibilidad de
ejemplos en tu repositorio y la posibilidad de navegar por la
documentación de las dependencias, de esta forma podemos navegar por esta 
documentación desde una libreria backend web que nosotros queramos e ir 
profundizando más y más hasta que lleguemos a la documentación oficial de Rust
con los tipos más básicos del lenguaje, esto es porque la documentación
está diseñada para ser navegable de punta a punta sin salir del sitio, de forma
que puedes interconectar múltiples documentaciones. Si la librería que usas
tiene comentarios de documentación (hay diferencias entre comentarios
documentación y comentarios normales) se mostrarán directamente en ella y al 
tener una dependencia de la misma librería podrás navegar por esta documentación
y revisar la documentación de la dependencia y a su vez la documentación de esa
otra dependencia, y así sucesivamente hasta llegar al core de Rust fue dicho.

Esta documentación como dijimos será generada de forma automática en base a tu
código, si no incluyes ningún comentario, ni ejemplos de código, la
generación de documentación no será muy útil pero si mostrará por ejemplo
los tipos de datos que estés utilizando, las funciones públicas, los módulos
y demás, pero si incluyes comentarios de documentación verás que es muy fácil
enriquecerla sin mucho esfuerzo.

También Rust provee un linter integrado llamado `clippy`, que ayuda a
identificar problemas comunes en el código y sugiere mejoras. `Clippy` se
ejecuta automáticamente al compilar el proyecto y proporciona advertencias
y sugerencias para mejorar la calidad del código. Por ejemplo, puede
sugerir el uso de una función más eficiente o advertir sobre posibles
errores de lógica. El que esto sea algo estandarizado permite que la amplia 
mayoría de los proyectos de Rust tengan una calidad de código más alta y
consistente, `clippy` tiene varios modos y puede resultar siendo extremadamente
inteligente para detectar algunos problemas.

Como hemos visto al comienzo del libro también hay herramientas online como
los playgrounds de Rust, similares a los de Go pero en estos se incluyen
las funcionalidades de agregar dependencias, utilizar `clippy`, cambiar de 
`toolchain`, se permiten multiples versiones de Rust y como vemos se puede 
extender muchísimo ya que hay distintos playgrounds de Rust que permiten por 
ejemplo 








