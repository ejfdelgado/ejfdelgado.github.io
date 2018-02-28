---
layout: default
titulo: html5/css3/ts/js
descripcion: Arquitectura de referencia para la capa de presentación de aplicaciónes empresariales en Colombia
---

* Do not remove this line (it will not be displayed)
{:toc}

# [](#header-1)Objetivo

Definir la arquitectura de referencia para el desarrollo de una aplicación web basada en el diseño SPA (Single Page Application) y dirigida a la tecnología Angular 5. Este documento solo se enfocará en la capa de presentación. La meta final es proveer a los equipos de desarrollo lineamientos sencillos y claros, a través de los módulos mínimos, que permitan avanzar progresivamente el desarrollo de acuerdo a las necesidades sin tener que hacer reprocesos.

# [](#header-1)Metodología

Dado que nuestro interés va dirigido a soluciones empresariales en Colombia, se listarán los requerimientos funcionales y no funcionales más comunes a los que se enfrenta el equipo de desarrollo de las aplicaciones web en este contexto. Así mismo se definirá el alcance a través del entendimiento de los actores con los que la aplicación web se debe relacionar. Finalmente se definirán los lineamientos y módulos mínimos para cumplir con los retos planteados.


# [](#header-1)Módulos y Lineamientos

## [](#header-2)Módulo de invocación a servicios
***
Se establece como la única puerta de comunicación con los servicios del back end. Generalmente hay varios aspectos técnicos, de negocio y de desarrollo que son recurrentes al momento de implementar la comunicación con los servicios. Por lo tanto es buena práctica encapsular esta lógica en un único módulo.
	
### [](#header-3)¿Por qué es necesario?

* <strong>Seguridad</strong>: La invocación a servicios puede incluir manejo de cabeceras http para soportar los aspectos de seguridad. El módulo puede esconder esta lógica repetitiva en cada invocación. Si eso dependiera del desarrollador en cada invocación, podría olvidarlo y quedarían servicios incompletos.
* <strong>Complejidad de desarrollo</strong>: Generalmente hay lógica repetitiva en la capa de presentación asociada a las invocaciones http. Por ejemplo, 1. Cuando hay error de servidor se debe interpretar y mostrar una ventana emergente, 2. Cuando todo va bien, se muestra una ventana de éxito genérica, 3. Mientras el servicio está en proceso de responder, se debe indicar que el sistema está ocupado, entre otros. Si toda esta lógica se debe implementar cada vez que se hace una invocación, será demasiado compleja y confusa para leer y corregir errores.

### [](#header-3)Lineamientos

#### [](#header-4)Se debe usar una arquitectura de tuberías

Se plantea esta arquitectura porque usualmente en la comunicación desde y hacia los servicios, hay procesos genéricos de transformación de datos repetitivos. Dependiendo de la dirección en que viajan los datos se incluyen:

* <strong>Invocación</strong>: Este módulo plantea factorizar atributos repetitivos en la invocación. Por ejemplo, la IP y puerto de la url objeto de la invocación; el atributo "Accept", puede ser siempre application/json; encabezados de seguridad como el "Bearer" se debe agregar automáticamente; finalmente pueden necesitarce encabezados propios de la solución. Todo esto permite agregar modificaciones sin afectar los desarrollos previos.

* <strong>Respuesta</strong>: El "Status Code" de la respuesta puede implicar transformación de datos. Por ejemplo, el estándar http define el 204 cuando no hay resultados y los servicios no están obligados a retornar ningún dato, entonces se hace necesario y útil recrear una respuesta virtual acorde al dato esperado. Por ejemplo, si se esperaba una lista, se debe responder con una lista vacia [], si se esperaba un conteo, se debe responder con un cero, si se esperaba un texto o un objeto, se debe recrear un nulo.

* <strong>Respuesta con caché</strong>: Se puede manejar un caché en el front-end, con políticas de actualización periódica. En este caso la respuesta será generada automáticamente a partir de datos en memoria sin hacer la invocación realmente.

#### [](#header-4)Las tuberías deben transformar datos en comportamientos

Hay unos comportamientos que dependen del progreso de la invocación http y otros del "Status Code" de la respuesta.

- <strong>Progreso de la Invocación</strong>: Al iniciar la invocación se debe retroalimentar al usuario que el sistema está ocupado hasta que la invocación finalice. Un caso más sofisticado puede incluir la notificación de progreso, por ejemplo cuando se trata de enviar archivos grandes al servidor. Finalmente se puede agregan un botón cancelar que permita abortar la invocación a juicio del usuario.

- <strong>"Status Code" de la Respuesta</strong>: Los códigos estándares http permiten implementar comportamientos que se deben factorizar. Algunos ejemplos se listan a continuación:

	* <strong>Errores generales</strong>: El código 500 debe por lo general mostrar una ventana emergente notificando que es un error inesperado en el servidor.
	* <strong>Errores de negocio</strong>: Por otro lado, el código de error de negocio 409, a diferencia del anterior pueden dar algún detalle o mensaje que permita entender más fácilmente lo que sucede, en estos casos la ventana emergente podría remplazar el mensaje genérico por el detalle que provea el back end.
	* <strong>Trazabilidad</strong>: Se puede incluir que en caso de error, invoque otro servicio de log de errores que incluya detalles de la invocación y la respuesta.
	* <strong>Verbosidad</strong>: En modo desarrollo, se puede habilitar un modo verboso visualmente que permita al desarrollador ver la traza completa de error sin tener que ver la consola de errores.

#### [](#header-4)Tuberías configurables

Se debe permitir habilitar o desabilitar los comportamientos predefinidos porque siempre hay excepciones a estas reglas. Por ejemplo, habrán casos donde, por negocio, no importa si un servicio falla, solo debe continuar.

## [](#header-2)Módulo indicador de actividad
***
El indicador de actividad es un módulo sencillo pero muy útil que se materializa en esa pantalla trasparente u oscura. Su función principal es indicar que el sistema está ocupado.

### [](#header-3)¿Por qué es necesario?

* <strong>Usabilidad</strong>: El usuario podrá saber que el sistema está trabajando porque tendrá una retroalimentación visual inmediata después que él realiza una acción.
* <strong>Rendimiento</strong>: El sistemá limitará las acciones del usuario para por ejemplo evitar presionar dos veces el mismo botón por equivocación. Este comportamiento evita sobrecargar el servidor inutilmente.

### [](#header-3)Lineamientos

Este módulo debe ser singleton. Se debe plantear como un módulo dentro de la arquitectura de tuberías del "módulo de invocación a servicios".



## [](#header-2)Módulo de filtros de datos
***

Este módulo incluye las funciones de transformación que se aplican a los datos con el fin de presentarlos al usuario final sin perjudicar el tipo de dato que se maneja en código. Por ejemplo: el dinero debe ser decimal; pero el usuario vera $2'000.000.00, por otro lado las fechas deben ser de tipo entero (epoch); pero al usuario se le mostrará el formato dd/mm/aaaa (o el formato que defina el cliente).

### [](#header-3)¿Por qué es necesario?

* <strong>Rendimiento</strong>: si internamente se usan datos formateados, cada vez que toque hacer operaciones como sumas o comparaciones, será necesario primero transformar los datos, incurriendo en más operaciones. Este punto se agrava cuando las funciones de transformación son complejas.

* <strong>Propenso a errores</strong>: un caso ocurre en las comparaciones de datos y como consecuenca también en los ordenamientos. Por ejemplo el formato de fecha dd/mm/aaaa no se ordena correctamente porque el texto "02/02/2017" es mayor que "01/02/2018" lo cual es falso.

* <strong>Homogeneidad</strong>: Si no se centralizan las funciones de transformación de datos, puede pasar que: 1. Los desarrolladores presentarán las variables al usuario final en bruto, es decir sin el formato deseado. 2. Los desarrolladores replicarán las funciones de transformación exibiendo comportamientos diferentes incluyendo posibles errores.

### [](#header-3)Lineamientos

#### [](#header-4)¿Qué filtros crear?

En este punto es mejor pecar por exceso. Hay unos básicos que saltan a la vista como lo que se han presentado previamente; fechas o dinero. Pero también pueden existir otros más complejos, por ejemplo el que recibe un objeto que representa una empresa y genera el formato "Nit 897456-8 / Heinsohn", otro común es el que convierte un dato boleano y genera Sí/No, finalmente incluyo el que formatea decimales rellenando con ceros después de la coma. En cada proyecto se deben detectar todos los tipos de datos que se requieren formatear.

#### [](#header-4)Las enumeraciones deben tener su propio filtro

Las enumeraciones son definiciones de tipos de datos que tienen sentido en código pero no se deben presentar al usuario final sin formatear. Por ejemplo, mientras a lo largo del código se maneja la enumeración "CEDULA_CIUDADANIA" el usuario deberá ver "Cédula de ciudadanía". Se debe crear entonces un filtro especial que transforma enumeraciones a su representación formateada. Este punto se debe complementar con el módulo de administración de listas de datos.

#### [](#header-4)Deben ser bidireccionales

Significa que debe existir la función que formatea el dato pero también la función que regenera el dato a partir de una representación formateada. Aplica para todos los tipos de datos involucrados en campos de entrada. Esto es prerrequisito porque los campos de entrada recibirán el dato en la representación del usuario y estos deben bajar al código como datos básicos. Una implicación de este punto es que las funciones deben ser 1:1, es decir que una misma representación formateada debe tener un único valor posible.

## [](#header-2)Módulo de administración de listas de datos
***

En este módulo se plantea administrar todos los tipos de datos de negocio que <strong>no</strong> son básicos (enteros, textos). Por ejemplo: 1. Tipos de documento, 2. Departamentos. 3. Estado civil entre otros. Pueden tener su origen en enumeraciones o tablas, en ambos casos para la capa de presentación el origen debe ser transparente.

### [](#header-3)¿Por qué es necesario?

* <strong>Facilita el desarrollo</strong>: Las listas de datos se van a usar con la misma intensidad que se usarán los tipos de datos básicos. Por lo tanto si este tema está solucionado de manera transversal, facilitará su uso y administración.

* <strong>Rendimiento</strong>: Por un lado, las listas de datos pueden ser muchas, por otro lado, cada lista de datos puede tener muchos elementos. Por lo tanto, la manera como se administren va tener un impacto fuerte en rendimiento sea positivo o negativo.

### [](#header-3)Lineamientos

En su implementación debe ser singleton.

#### [](#header-4)Debe tener cache

Teniendo en cuenta que las listas de datos rara vez cambian y que tienen un uso intensivo a lo largo de la aplicación. Este módulo debe soportar caché en la capa de presentación para que sirva de proxy antes de invocar el servicio. Esto tiene un impacto fuerte porque usualmente todas las pantallas tienen algun tipo de dato no básico sumado a los argumentos mencionados previamente.

#### [](#header-4)Debe permitir metadatos

Las listas de datos deben contener una representación tipo máquina, por ejemplo el id, que puede ser numérico o enumerado entre otros. Por otro lado deben tener la representación de usuario final que será usada para filtrar el dato. Opcionalmente debe permitir incluir metadatos. Los metadatos son información que complementa el tipo de dato y que está estrechamente relacionada a cada valor dentro de una lista de datos. Por ejemplo, los departamentos o municípios tienen asociado un indicativo de teléfono, el cuál debe estar disponible al tiempo que lo está.

#### [](#header-4)Deben ser filtrables

Las listas de datos deben permitir ser filtrables. Esto es muy importante para permitir que las listas de datos tengan jerarquía. Un ejemplo básico son las listas de departamentos y municipios, en este caso los municipios deben permitir ser filtrados por un departamento específico.

## [](#header-2)Módulo de administración de fechas
***

Las fechas son un tipo de dato especial que tiene comportamientos complejos que no son evidentes. 

### [](#header-3)¿Por qué es necesario?

* <strong>Vulnerabilidad</strong>: En las empresas colombianas hay muchos requerimientos fuertes que involucran las fechas, por ejemplo restricciones de máximos y mínimos. En los navegadores, bastará con cambiar la fecha del computador para hacer creer a la capa de presentación que la fecha es otra.
* <strong>Complejidad alta</strong>: Las fechas incluyen muchos cálculos complejos que son repetitivos. Por ejemplo: 1. El último día del mes, que debe tener en cuenta que los meses tienen días irregulares incluyendo los años biciestos. 2. Conteo de días hábiles, que debe tener en cuenta la parametrización de base de datos con los días feriados para colombia. Si no se centralizan estos cáculos, se  replicará código y será muy probable que estos contengan errores.

### [](#header-3)Lineamientos

#### [](#header-4)Deben tener representación epoch

El tipo de dato más sencillo, preciso y estándar es el que plantea unix; el epoch. Es ampliamente soportado en todos los lenguajes: javascript, java, python, php, etc. Por lo tanto se debe utilizar este tipo de dato, tanto en el código de la capa de presentación como en la definición de los servicios del back end.

#### [](#header-4)La fecha debe estar sincronizada

Si la fecha no está sincronizada con la del servidor, bastará con cambiar la fecha del computador para poder esquivar las reglas. Entonces el módulo deberá mantener una versión actualizada de la fecha actual y permitir ser accedida desde el principio de la aplicación.

#### [](#header-4)Debe centralizar las operaciones

Este módulo debe centralizar todas las operaciones que involucren fechas, sean sumas, restas, comparaciones, etc. Todas estas funciones deben recibir representaciones epoch y deben igualmente entregar epoch.

## [](#header-2)Módulos de bibliotecas
***

Se definen como el conjunto de datos estructurados que tienen un sentido de negocio sin lógica de programación, por ejemplo textos como etiquetas de campos, ayudas, definiciones de botones, definición de máximos y mínimos, expresiones regulares entre otros.

### [](#header-3)¿Por qué es necesario?

Las entidades del estado y empresas colombianas se caracterizan por tener gran cantidad de entidades de negocio, validaciones, reglas complejas y repetitivas. Al llevarlas a un desarrollos web si dicha lógica de negocio no está centralizada, tarde o temprano saldrán errores porque no hay homogeneidad en la aplicación, será evidente porque todo se comportará de formas diferentes y será difícil de mantener porque hay código replicado.

### [](#header-3)Lineamientos

En su implementación debe ser singleton.

### [](#header-4)¿Qué se debe incluir en las bibliotecas?

En la capa de presentación tenemos los siguientes aspectos clave:

1. <strong>Estático visual</strong>: En este aspecto se incluyen textos, botones y ventanas emergentes. Si estos elementos se estandarizan, los desarrolladores se desgastarán menos tiempo pensando en cómo mezclar los diferentes elementos para que las pantallas luzcan lo mejor posible.
* <strong>Los textos</strong>: son bastamente usados, sobre todo en títulos, mensajes emergentes, ayudas y etiquetas de campos entre otros. Una ventaja de tener una biblioteca de textos es que permite administrarla por una persona no desarrolladora y además abre la posibilidad de soportar múltiples lenguajes.
* <strong>Los botones</strong>: estos tienen reglas controladas por la usabilidad. Se debe acotar la libertad que tienen los desarrolladores para usar los botones porque no todas las combinaciones de color, etiquetas, ayudas e iconos son correctas. Una biblioteca de botones debe entonces permitir elegir de un conjunto limitado el botón más adecuado para la ocasión.
* <strong>Las ventanas emergentes</strong>: son el conjunto de los dos aspectos anteriores, más algunos propios: ellas tienen título, ícono, mensaje y botones. Algunas veces contienen campos y tablas pero eso se abordará más adelante. Una vez más, acá se deben acotar los aspectos mencionados porque no todas las combinaciones tienen sentido. Además hay algunos que se repetirán demasiado como los mensajes de éxito, error, los que piden confirmar antes de continuar, entre otros.

2. <strong>Comportamientos por configuración</strong>: 
* Validaciones: Cada entidad de negocio tendrá una representación visual que usualmente es un conjunto de campos. Cada campo tiene sus propias validaciones, gran parte de las validaciones se pueden resolver con expresiones regulares, máximos y mínimos.

#### [](#header-4)Deben permitir fragmentarse y/o tener jerarquía

Para tener éxito en la implantación de las bibliotecas, se deben fragmentar y/o jerarquizar. Si no está divido y/o agrupada la información, usualmente los desarrolladores perderán el orden repitiendo definiciones o dejando de usar las bibliotecas centralizadas remplazándolas por código quemado.

Entonces, la primera pregunta que deben responder es ¿en qué dirección va a aumentar progresivamente la aplicación? una respuesta válida, pero no la única, puede ser "historias de usuario". Esto ayudará a definir cómo partir las bibliotecas que por lo general tienden a ser muy extensas.

Fragmento común: Siempre debe haber una parte que es común, allí se deben registrar aquellas definiciones transversales que se reutilizarán a lo largo de todo el desarrollo, este fragmento evolucionará rápidamente al principio y cesará en la mitad del proyecto. Será conocido por todos los colaboradores que poco a poco lo isrán construyendo.

Luego, teniendo en cuenta de la dirección en que aumenta la aplicación, se debe tomar una segunda decisión, que...

Cada proyecto puede tener sus propias bibliotecas, pero...

- Biblioteca de representación visual de entidades de negocio que contiene: etiquetas/ayudas/validaciones(regex/min/max(fechas basadas en fecha del servidor))/mensajes de error/



## [](#header-2)Visualizador de documentos
***

Visor de documentos (pdf/txt/imágenes) Con capacidad de imprimir.

### [](#header-3)¿Por qué es necesario?

* <strong></strong>: 

### [](#header-3)Lineamientos

## [](#header-2)Módulo cliente de gestor documental
***

Interfáz con Gestor documental (subir, bajar, actualizar)

### [](#header-3)¿Por qué es necesario?

* <strong></strong>: 

### [](#header-3)Lineamientos


## [](#header-2)Fabrica de campos
***

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

### [](#header-3)¿Por qué es necesario?

* <strong></strong>: 

### [](#header-3)Lineamientos

## [](#header-2)Módulo de seguridad
***
Interfáz con componente de SSO (p.e. Keycloak).

### [](#header-3)¿Por qué es necesario?

* <strong></strong>: 

### [](#header-3)Lineamientos


## [](#header-2)Módulo de tablas
***
Debe incluir:

- Paginado del lado del servidor (ordenamiento, filtrado por columnas)
- Contenido dinámico e interactivo. (botones, campos editables, validaciones cruzadas).
- Generador de archivos CSV/XLSX.
- Check box de selección unica/todos
- Control de resaltado de filas dependiendo de condiciones.
	
### [](#header-3)¿Por qué es necesario?

* <strong></strong>: 

### [](#header-3)Lineamientos


## [](#header-2)Módulo de parámetros
***
	
### [](#header-3)¿Por qué es necesario?

* <strong></strong>: 

### [](#header-3)Lineamientos



# [](#header-1)Texto sin procesar
***

- Generador de documentos (json a html) / certificados PDF. Con encabezado, pie, marcas de agua, contenido dinámico como tablas y texto a través de plantillas y claves. Integrado con campo de texto enriquecido. Permitir firma digital.

- Manejo de url's con estado embebido + encripción.

- Manejo de Formularios:
	- Usar jerarquía de Sección/Grupo de campos/etiqueta de campo. p.e. con fines de auditoría.
	- Retroalimentación visual de errores con color rojo.
	- Asegurar atributos name únicos.
- Modelo
	- Detección de cambios en el modelo. Para por ejemplo solicitar guardar antes de abandonar pantalla.
	- Granularidad de campo para dejar registro de auditoría.
	- Almacenamiento de estados del modelo para permitir "deshacer".
- SEO y facilidad de acceso con herramientas que leen el código y generan voz.
	- tag <description> ligado al estado (url)
	- Maquetación estructurada usando H1, H2, H3.
	- Inputs con name=""
	- Imágenes con alt=""
- Control de inicialización mínima para que la aplicación funcione correctamente (parámetros/fecha sincronizada)


- Tener claro el manejo para:
	- Historias nuevas
	- Campos nuevos
	- Modales nuevos, que incluyan campos/tablas/botones.
	- Integración con BPM.

- Administrador de tabs.
	- Abandono de tab condicionado.
	- Navegación controlada.
