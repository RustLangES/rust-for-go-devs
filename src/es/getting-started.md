# Empezando

Rust es un lenguaje que permite ser escrito en diversos lugares.

Si bien nosotros recomendamos escribir código Rust en un editor de texto y de
forma local, también es posible escribir código Rust en un navegador web.

## Rust Playground

La forma más sencilla de comenzar con Rust sin necesidad de ninguna instalación 
local es utilizar el [Playground de Rust][playground]. Es una interfaz de 
desarrollo mínima que se ejecuta en el navegador web y permite escribir y 
ejecutar código Rust.

## Rust Explorer

Otra opción es utilizar el [Rust Explorer][explorer], que es una herramienta
que permite escribir código Rust, similar al Playground, pero este no es 
oficial y no es mantenido por el equipo de Rust. Sin embargo, es una buena
alternativa si quieres un editor más completo, debido a que permite
obtener autocompletado, resaltado de sintaxis y otras características que
pueden ser útiles.

## CodeSandbox

También puedes utilizar [CodeSandbox][codesandbox], que es una plataforma
que permite crear aplicaciones web pero hace unos años 
[lanzó soporte para Rust][codesandbox-support].
Tiene la ventaja de que que permite instalar dependencias y crear proyectos más
complejos. Recomendamos leer el blog post para entender cómo funciona 
integrado con Rust, además provee varias plantillas que puedes usar para 
proyectos más complejos, como levantar un servidor web directamente 
desde el navegador. (Maravilloso, ¿no?)

## Dev Container

Los entornos de ejecución web tiene algunas limitaciones, 
como el tiempo de compilación/ejecución, la memoria y la red. Otra opción que no
requiere instalar Rust sería utilizar un _dev container_, como el proporcionado 
en el repositorio <https://github.com/microsoft/vscode-remote-try-rust>. Al 
igual que el Playground de Rust, el contenedor de desarrollo se puede ejecutar
directamente en un navegador web utilizando [GitHub Codespaces] o 
[localmente con Visual Studio Code][vscode-dev-containers].

## Instalación local

La forma más común de comenzar a escribir código Rust es instalarlo
localmente en tu máquina. 

Para realizar una instalación local completa del compilador Rust y sus 
herramientas de desarrollo, consulta la sección [Instalación][installation] del 
capítulo [Empezando][getting-started] en el libro 
[El Lenguaje de Programación Rust][rs-book], o 
[visita la página de instalación][installation-website] en [rust-lang.org].


[playground]: https://play.rust-lang.org/
[explorer]: https://rustexplorer.com/
[codesandbox]: https://codesandbox.io/
[codesandbox-support]: https://codesandbox.io/blog/announcing-rust-support-in-codesandbox
[GitHub Codespaces]: https://github.com/features/codespaces
[vscode-dev-containers]: https://code.visualstudio.com/docs/devcontainers/containers
[installation]: https://book.rustlang-es.org/ch01-01-installation
[getting-started]: https://book.rustlang-es.org/ch01-00-getting-started
[rs-book]: https://book.rustlang-es.org
[installation-website]: https://www.rust-lang.org/tools/install
[rust-lang.org]: https://www.rust-lang.org/