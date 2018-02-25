---
layout: default
title: html5/css3/ts/js
description: Arquitectura de referencia para la capa de presentación de aplicaciónes empresariales en Colombia
---
# [](#header-1)Objetivo

Definir la arquitectura de referencia para el desarrollo de una aplicación web basada en el diseño SPA (Single Page Application) y dirigida a la tecnología Angular 4. Este documento solo se enfocará en la capa de presentación. La meta final es proveer a los equipos de desarrollo lineamientos sencillos y claros, a través de módulos, que permitan avanzar progresivamente el desarrollo de acuerdo a las necesidades sin tener que hacer reprocesos (refactor).

# [](#header-1)Metodología

Entendiendo que nuestro interés va dirigido a soluciones empresariales en Colombia, se listarán los requerimientos funcionales y no funcionales más comunes a los que se enfrenta el equipo de desarrollo de las aplicaciones web. Como resultado se podrá definir el alcance a través de la enumeración de los componentes con los que la aplicación web interactuará para finalmente definir los lineamientos y módulos.


# [](#header-1)Desarrollo

## [](#header-2)Bibliotecas:

Se definen como el conjunto de datos estructurados que tienen un sentido de negocio sin lógica de programación, por ejemplo expresiones regulares, definición de máximos y mínimos, etiquetas de campos, ayudas, definiciones de botones (colores, iconos, etiquetas).

### [](#header-3)¿Por qué es necesario?

Las entidades del estado y empresas colombianas se caracterizan por tener gran cantidad de validaciones, reglas complejas y repetitivas. Al incluirlas en desarrollos web, si dicha lógica de negocio no está centralizada, se comenzará a replicar código, tarde o temprano saldrán errores porque no hay homogeneidad en la aplicación; todo se comportará de formas diferentes, será evidente y difícil de mantener.

### [](#header-3)¿Qué se debe incluir en las bibliotecas?

En la capa de presentación hay dos aspectos clave:

1. <strong>Estático visual</strong>: En general las aplicaciones web incluyen textos, botones y ventanas emergentes que deben estandarizarse. Para los textos, se requiere un módulo multilenguaje que defina títulos, mensajes de alerta, ayudas y etiquetas de campos entre otros. Los botones, a su vez tienen reglas controladas por la usabilidad. Se debe acotar la libertad que tienen los desarrolladores para usar los botones porque no todas las combinaciones de color, etiquetas, ayudas e iconos son correctas. Una biblioteca de botones debe entonces permitir elegir de un conjunto limitado el botón más adecuado para la ocasión. Finalmente, las ventanas emergentes, son el conjunto de los dos aspectos anteriores más algunos propios: tiene título, ícono, mensaje y botones. Algunas veces contienen controles y tablas pero eso se abordará más adelante.

2. <strong>Comportamientos por configuración</strong>: 

### [](#header-3)Lineamientos

#### [](#header-4)Fragmentación y/o jerarquía:

Para tener éxito en la implantación de las bibliotecas, se deben fragmentar y/o jerarquizar. Si no está divido y/o agrupada la información, usualmente los desarrolladores perderán el orden repitiendo definiciones o dejando de usar las bibliotecas centralizadas remplazándolas por código quemado.

Entonces, la primera pregunta que deben responder es ¿en qué dirección va a aumentar progresivamente la aplicación? una respuesta válida, pero no la única, puede ser "historias de usuario". Esto ayudará a definir cómo partir las bibliotecas que por lo general tienden a ser muy extensas.

Fragmento común: Siempre debe haber una parte que es común, allí se deben registrar aquellas definiciones transversales que se reutilizarán a lo largo de todo el desarrollo, este fragmento evolucionará rápidamente al principio y cesará en la mitad del proyecto. Será conocido por todos los colaboradores que poco a poco lo isrán construyendo.

Luego, teniendo en cuenta de la dirección en que aumenta la aplicación, se debe tomar una segunda decisión, que 

Cada proyecto puede tener sus propias bibliotecas, pero 


# [](#header-1)Requerimientos:

Módulos:

- Interfáz con Gestor documental (subir, bajar, actualizar)
- Visor de documentos (pdf/txt/imágenes) Con capacidad de imprimir.
- Biblioteca de expresiones regulares
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
- Generador de documentos / certificados PDF. Con encabezado, pie, marcas de agua, contenido dinámico como tablas y texto a través de plantillas y claves. Integrado con campo de texto enriquecido.
- Seguridad, interfáz con componente de SSO (p.e. Keycloak).
- Indicador de actividad, con opción de barra de progreso, mensaje y opción de cancelar.
- Cargue de listas tipo (llave/valor) con metadata + caché. p.e. Departamento/Municipio/Indicativo tel.
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
- Administrador de fechas.
	- Sincronizar fecha con el servidor; no usar la local del navegador.
	- Cálculo epoch de: (Fin del (día/mes/año), Inicio del (día/mes/año), sumar (días, meses, años))
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