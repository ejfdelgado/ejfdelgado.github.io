---
layout: default
titulo: html5/css3/ts/js
descripcion: Arquitectura de referencia para la capa de presentación de aplicaciónes empresariales en Colombia
---

* Do not remove this line (it will not be displayed)
{:toc}

# [](#header-1)Objetivo

Definir la arquitectura de referencia para el desarrollo de una aplicación web basada en el diseño SPA (Single Page Application) y dirigida a la tecnología Angular 5. Este documento solo se enfocará en la capa de presentación. La meta final es proveer a los equipos de desarrollo lineamientos sencillos y claros, a través de módulos, que permitan avanzar progresivamente el desarrollo de acuerdo a las necesidades sin tener que hacer reprocesos (refactor).

# [](#header-1)Metodología

Entendiendo que nuestro interés va dirigido a soluciones empresariales en Colombia, se listarán los requerimientos funcionales y no funcionales más comunes a los que se enfrenta el equipo de desarrollo de las aplicaciones web. Como resultado se podrá definir el alcance a través de la enumeración de los componentes con los que la aplicación web interactuará para finalmente definir los lineamientos y módulos.


# [](#header-1)Desarrollo

***

## [](#header-2)Filtros de datos

Son las funciones de transformación que se aplican a los datos con el fin de presentarlos al usuario final sin perjudicar el tipo de dato que se maneja en código. Por ejemplo: las fechas deben ser de tipo entero (epoch); pero al usuario se le mostrará dd/mm/aaaa, por otro lado el dinero debe ser decimal; pero el usuario vera $2'000.000.00.

### [](#header-3)¿Por qué es necesario?

* <strong>Rendimiento</strong>: si internamente se usan datos formateados, cada vez que toque hacer operaciones como sumas o comparaciones, será necesario primero transformar los datos, incurriendo en más operaciones. Este punto se agrava cuando las funciones de transformación son complejas.

* <strong>Propenso a errores</strong>: un caso ocurre en las comparaciones de datos y como consecuenca también en los ordenamientos. Por ejemplo el formato de fecha dd/mm/aaaa no se ordena correctamente porque el texto "02/02/2017" es mayor que "01/02/2018" lo cual es falso.

* <strong>Homogeneidad</strong>: Si no se centralizan las funciones de transformación de datos, puede pasar que: 1. Los desarrolladores presentarán las variables al usuario final en bruto, es decir sin el formato deseado. 2. Los desarrolladores replicarán las funciones de transformación que puede que presenten comportamientos diferentes.

### [](#header-3)Lineamientos

#### [](#header-4)Bidireccional

Significa que debe existir la función que formatea el dato pero también la función que regenera el dato a partir de una representación formateada. Aplica para todos los tipos de datos involucrados en campos de entrada. Esto es prerrequisito porque los campos de entrada recibirán el dato en la representación del usuario y estos deben bajar al código como datos básicos. Una implicación de este punto es que las funciones deben ser 1:1, es decir que una misma representación formateada debe tener un único valor posible.

#### [](#header-4)¿qué filtros crear?

En este punto es mejor pecar por exceso. Hay unos básicos que saltan a la vista como lo que se han presentado previamente; fechas o dinero. Pero también pueden existir otros más complejos, por ejemplo el que recibe un objeto que representa una empresa y genera el formato "Nit 897456-8 / Heinsohn", otro común es el que convierte un dato boleano y genera Sí/No. En cada proyecto se deben detectar todos los tipos de datos que se requieren formatear.

#### [](#header-4)Enumeraciones

Las enumeraciones son literales o textos que tienen sentido en código pero no se deben presentar al usuario final sin formatear. Por ejemplo, mientras a lo largo del código se maneja la enumeración "CEDULA_CIUDADANIA" el usuario deberá ver "Cédula de ciudadanía". Se debe crear entonces un filtro especial que transforma enumeraciones a su representación formateada. Este punto se debe complementar con el módulo de administración de listas de datos.

***

## [](#header-2)Módulo de administración de listas de datos

Este módulo incluyen todos los tipos de datos de negocio que <strong>no</strong> son básicos (enteros, textos). Por ejemplo: 1. Tipos de documento, 2. Departamentos. 3. Estado civil entre otros. Pueden ser de dos tipos según su origen: enumeraciones o tablas, en ambos casos para la capa de presentación el origen debe ser transparente.

### [](#header-3)¿Por qué es necesario?

* <strong>Rendimiento</strong>: Teniendo en cuenta que las listas de datos rara vez cambian y que tienen un uso intensivo a lo largo de la aplicación. Se puede incluir un módulo de caché en la capa de presentación que sirva de proxi antes de invocar el servicio. Esto tiene un impacto fuerte porque usualmente todas las pantallas tienen algun tipo de dato no básico.

* <strong>Mantenibilidad</strong>: Código centralizado.

* <strong>Facilita el desarrollo</strong>: Las listas de datos se van a usar con la misma intensidad que se usarán los tipos de datos básicos. Por lo tanto si este tema está solucionado de manera transversal, facilitará su uso y administración.

### [](#header-3)Lineamientos

En su implementación debe ser singleton.

#### [](#header-4)Debe permitir metadatos

Las listas de datos deben contener una representación tipo máquina, por ejemplo el id, que puede ser numérico o enumerado entre otros. Por otro lado deben tener la representación de usuario final que será usada para filtrar el dato. Opcionalmente debe permitir incluir metadatos. Los metadatos son información que complementa el tipo de dato y que está estrechamente relacionada a cada valor dentro de una lista de datos. Por ejemplo, los departamentos o municípios tienen asociado un indicativo de teléfono, el cuál debe estar disponible al tiempo que lo está.

#### [](#header-4)Filtrable

Las listas de datos deben permitir ser filtrables. Hay listas de datos

***

## [](#header-2)Módulo de administración de fechas

Las fechas son un tipo de dato especial que tiene comportamientos complejos que no son evidentes. 

### [](#header-3)¿Por qué es necesario?

* <strong>Vulnerabilidad</strong>: En las empresas colombianas hay muchos requerimientos fuertes que involucran las fechas, por ejemplo restricciones de máximos y mínimos. En los navegadores, bastará con cambiar la fecha del computador para hacer creer a la capa de presentación que la fecha es otra.
* <strong>Complejidad alta</strong>: Las fechas incluyen muchos cálculos complejos que son repetitivos. Por ejemplo: 1. El último día del mes, que debe tener en cuenta que los meses tienen días irregulares incluyendo los años biciestos. 2. Conteo de días hábiles, que debe tener en cuenta la parametrización de base de datos con los días feriados para colombia. Si no se centralizan estos cáculos, se  replicará código y será muy probable que estos contengan errores.

### [](#header-3)Lineamientos

#### [](#header-4)Representación epoch

El tipo de dato más sencillo, preciso y estándar es el que plantea unix; el epoch. Es ampliamente soportado en todos los lenguajes: javascript, java, python, php, etc. Por lo tanto se debe utilizar este tipo de dato, tanto en el código de la capa de presentación como en la definición de los servicios del back end.

#### [](#header-4)Fecha sincronizada

Si la fecha no está sincronizada con la del servidor, bastará con cambiar la fecha del computador para poder esquivar las reglas. Entonces el módulo deberá mantener una versión actualizada de la fecha actual y permitir ser accedida desde el principio de la aplicación.

#### [](#header-4)Operaciones

Este módulo debe centralizar todas las operaciones que involucren fechas, sean sumas, restas, comparaciones, etc. Todas estas funciones deben recibir representaciones epoch y deben igualmente entregar epoch.

***

## [](#header-2)Módulos de bibliotecas

Se definen como el conjunto de datos estructurados que tienen un sentido de negocio sin lógica de programación, por ejemplo textos como etiquetas de campos, ayudas, definiciones de botones, definición de máximos y mínimos, expresiones regulares entre otros.

### [](#header-3)¿Por qué es necesario?

Las entidades del estado y empresas colombianas se caracterizan por tener gran cantidad de entidades de negocio, validaciones, reglas complejas y repetitivas. Al llevarlas a un desarrollos web si dicha lógica de negocio no está centralizada, tarde o temprano saldrán errores porque no hay homogeneidad en la aplicación, será evidente porque todo se comportará de formas diferentes y será difícil de mantener porque hay código replicado.

### [](#header-3)¿Qué se debe incluir en las bibliotecas?

En la capa de presentación tenemos los siguientes aspectos clave:

1. <strong>Estático visual</strong>: En este aspecto se incluyen textos, botones y ventanas emergentes. Si estos elementos se estandarizan, los desarrolladores se desgastarán menos tiempo pensando en cómo mezclar los diferentes elementos para que las pantallas luzcan lo mejor posible.
* <strong>Los textos</strong>: son bastamente usados, sobre todo en títulos, mensajes emergentes, ayudas y etiquetas de campos entre otros. Una ventaja de tener una biblioteca de textos es que permite administrarla por una persona no desarrolladora y además abre la posibilidad de soportar múltiples lenguajes.
* <strong>Los botones</strong>: estos tienen reglas controladas por la usabilidad. Se debe acotar la libertad que tienen los desarrolladores para usar los botones porque no todas las combinaciones de color, etiquetas, ayudas e iconos son correctas. Una biblioteca de botones debe entonces permitir elegir de un conjunto limitado el botón más adecuado para la ocasión.
* <strong>Las ventanas emergentes</strong>: son el conjunto de los dos aspectos anteriores, más algunos propios: ellas tienen título, ícono, mensaje y botones. Algunas veces contienen campos y tablas pero eso se abordará más adelante. Una vez más, acá se deben acotar los aspectos mencionados porque no todas las combinaciones tienen sentido. Además hay algunos que se repetirán demasiado como los mensajes de éxito, error, los que piden confirmar antes de continuar, entre otros.

2. <strong>Comportamientos por configuración</strong>: 
* Validaciones: Cada entidad de negocio tendrá una representación visual, cuando.

### [](#header-3)Lineamientos

En su implementación debe ser singleton.

#### [](#header-4)Fragmentación y/o jerarquía

Para tener éxito en la implantación de las bibliotecas, se deben fragmentar y/o jerarquizar. Si no está divido y/o agrupada la información, usualmente los desarrolladores perderán el orden repitiendo definiciones o dejando de usar las bibliotecas centralizadas remplazándolas por código quemado.

Entonces, la primera pregunta que deben responder es ¿en qué dirección va a aumentar progresivamente la aplicación? una respuesta válida, pero no la única, puede ser "historias de usuario". Esto ayudará a definir cómo partir las bibliotecas que por lo general tienden a ser muy extensas.

Fragmento común: Siempre debe haber una parte que es común, allí se deben registrar aquellas definiciones transversales que se reutilizarán a lo largo de todo el desarrollo, este fragmento evolucionará rápidamente al principio y cesará en la mitad del proyecto. Será conocido por todos los colaboradores que poco a poco lo isrán construyendo.

Luego, teniendo en cuenta de la dirección en que aumenta la aplicación, se debe tomar una segunda decisión, que 

Cada proyecto puede tener sus propias bibliotecas, pero 

***

## [](#header-2)Módulo de 

### [](#header-3)¿Por qué es necesario?

* <strong></strong>: 

### [](#header-3)Lineamientos

***

# [](#header-1)Requerimientos

Módulos:

- Cargue de listas tipo (llave/valor) con metadata + caché. p.e. Departamento/Municipio/Indicativo tel.
- Biblioteca de expresiones regulares
- Administrador de fechas.
	- Sincronizar fecha con el servidor; no usar la local del navegador.
	- Cálculo epoch de: (Fin del (día/mes/año), Inicio del (día/mes/año), sumar (días, meses, años))

- Interfáz con Gestor documental (subir, bajar, actualizar)
- Visor de documentos (pdf/txt/imágenes) Con capacidad de imprimir.
- Filtros configurables (dinero/fecha/periodo/epoch) (manejo controlado de decimales, indicador de millomes, miles, signos de $,%)
- Directivas específicas para campos:
	- Básicos (texto/número/html5(Correo, Celular, Teléfono)) que tengan (ayudas, etiqueta de comparación, validaciones (min, max), múltiples errores) con botón de copiado y etiqueta para comparación.
	- Texto enriquecido WYSWYG.
	- Fechas: (año), (año/mes), (mes), (año/mes/día), (hora/minuto), (año/mes/día/hora/minuto) 
	- Archivos (administración de tamaño máximo/tipos válidos/mostrar progreso) con metadata para indexación
	- Lista autocompletable; con o sin slección de una lista.
	- Direcciones (Serializar / Deserializar)
	- Detector de lector de código de barras o QR.
	- Control de inhabilitado/corregible/requerido(asterísco)/
	- Selección múltiple con flechas -> <-. p.e. Roles de un usuario.
	- Tener clara la forma de hacer validaciones cruzadas, por ejemplo en manejo de rangos de fechas min y max.
	- Consultar persona + validaciones propias de Colombia (NIT, dígito de verificación, Cédula, Carné diplomático)
	- Validación inversa de turing (ver google)
- Generador de documentos (json a html) / certificados PDF. Con encabezado, pie, marcas de agua, contenido dinámico como tablas y texto a través de plantillas y claves. Integrado con campo de texto enriquecido. Permitir firma digital.
- Seguridad, interfáz con componente de SSO (p.e. Keycloak).
- Indicador de actividad, con opción de barra de progreso, mensaje y opción de cancelar.
- Manejo de url's con estado embebido + encripción.
- Tablas:
	- Paginado del lado del servidor (ordenamiento, filtrado por columnas)
	- Contenido dinámico e interactivo. (botones, campos editables, validaciones cruzadas).
	- Generador de archivos CSV/XLSX.
	- Check box de selección unica/todos
	- Control de resaltado de filas dependiendo de condiciones.
- Administrador de tabs.
	- Abandono de tab condicionado.
	- Navegación controlada.
- Manejo de Formularios:
	- Usar jerarquía de Sección/Grupo de campos/etiqueta de campo. p.e. con fines de auditoría.
	- Retroalimentación visual de errores con color rojo.
	- Asegurar atributos name únicos.
	
- SEO y facilidad de acceso con herramientas que leen el código y generan voz.
	- tag <description> ligado al estado (url)
	- Maquetación estructurada usando H1, H2, H3.
	- Inputs con name=""
	- Imágenes con alt=""

Modelo:
	- Detección de cambios en el modalo. Para por ejemplo solicitar guardar antes de abandonar pantalla.
	- Granularidad de campo para dejar registro de auditoría.
	- Almacenamiento de estados del modelo para permitir "deshacer".

- Control de inicialización mínima para que la aplicación funcione correctamente (parámetros/fecha sincronizada)

- Biblioteca de representación visual de entidades de negocio que contiene: etiquetas/ayudas/validaciones(regex/min/max(fechas basadas en fecha del servidor))/mensajes de error/

- Biblioteca de botones (iconos/textos/ayudas/colores)

Tener claro el manejo para:
- Historias nuevas
- Campos nuevos
- Modales nuevos, que incluyan campos/tablas/botones.

- Integración con BPM.