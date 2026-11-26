# JROProyectoDAW

Página de inicio del **CFGS Desarrollo de Aplicaciones Web (2º curso)** del I.E.S. Los Sauces, realizada por Juan Rivera Ordoñez.

![Versión](https://img.shields.io/badge/versión-1.1.0-blue) ![HTML5](https://img.shields.io/badge/HTML5-E34F26?logo=html5&logoColor=white) ![CSS3](https://img.shields.io/badge/CSS3-1572B6?logo=css3&logoColor=white) ![Apache](https://img.shields.io/badge/Apache-.htaccess-D22128?logo=apache&logoColor=white)

## Índice

- [Descripción](#descripción)
- [Estructura](#estructura)
- [Configuración del servidor (.htaccess)](#configuración-del-servidor-htaccess)
- [Páginas de error](#páginas-de-error)
- [Despliegue](#despliegue)
- [Tecnologías](#tecnologías)
- [Ramas](#ramas)
- [Versiones](#versiones)
- [Autor](#autor)

## Descripción

El proyecto es un índice web que sirve como punto de entrada a los proyectos de cada asignatura del curso. Desde `index.html` se accede, mediante tarjetas con icono, a:

| Siglas | Asignatura |
|--------|------------|
| DWES | Desarrollo web en entorno servidor |
| CIB  | Ciberseguridad |
| DWEC | Desarrollo web en entorno cliente |
| DASP | Digitalización aplicada a los sectores productivos |
| DAW  | Despliegue de aplicaciones web |
| DIW  | Diseño de interfaces web |
| SASP | Sostenibilidad aplicada al sistema productivo |

## Estructura

```
JROProyectoDAW/
├── .htaccess             # Configuración de Apache (listados y páginas de error)
├── index.html            # Página principal con el índice de asignaturas
└── webroot/
    ├── css/              # Hojas de estilo
    ├── error/
    │   ├── error403.html # Página para el error 403 (acceso prohibido)
    │   └── error404.html # Página para el error 404 (no encontrado / en construcción)
    ├── font/Roboto/      # Fuente Roboto
    └── images/           # Logos, iconos e imágenes de error
```

## Configuración del servidor (.htaccess)

El archivo [`.htaccess`](.htaccess) de la raíz configura Apache para este proyecto:

```apache
# Impide mostrar el listado de archivos de una carpeta sin índice (devuelve 403)
Options -Indexes

# Páginas de error personalizadas
ErrorDocument 404 /webroot/error/error404.html
ErrorDocument 403 /webroot/error/error403.html
```

- **`Options -Indexes`**: si se entra en una carpeta sin `index.html`, en lugar de listar sus archivos el servidor responde con un error **403**.
- **`ErrorDocument`**: redirige los errores 403 y 404 a las páginas propias del proyecto, de modo que funcionan aunque el proyecto se despliegue de forma independiente.

> **Requisito:** el servidor Apache debe permitir `.htaccess` en el directorio del proyecto (`AllowOverride All`, o al menos `AllowOverride Options FileInfo`).

## Páginas de error

| Código | Página | Cuándo aparece |
|--------|--------|----------------|
| 403 | [`webroot/error/error403.html`](webroot/error/error403.html) | Acceso a una carpeta sin índice o a un recurso prohibido |
| 404 | [`webroot/error/error404.html`](webroot/error/error404.html) | Recurso inexistente o sección aún en construcción |

## Despliegue

1. Copiar el contenido del repositorio en el directorio del servidor web (Apache).
2. Comprobar que `AllowOverride` permite usar el `.htaccess`.
3. `ErrorDocument` necesita la ruta **desde la raíz del servidor**. Si el proyecto no se despliega en la raíz, ajustar las rutas del `.htaccess`, por ejemplo:

   ```apache
   ErrorDocument 404 /JROProyectoDAW/webroot/error/error404.html
   ErrorDocument 403 /JROProyectoDAW/webroot/error/error403.html
   ```

## Tecnologías

- HTML5
- CSS3 (Flexbox y Grid)
- Apache HTTP Server (`.htaccess`)

## Ramas

- `master`: rama principal.
- `developerJRO`: rama de desarrollo.

## Versiones

| Versión | Cambios |
|---------|---------|
| **v1.1.0** | Añadido `.htaccess` (sin listado de directorios) y páginas de error 403 y 404 personalizadas. |
| v1.0.0 | Primera versión: índice de asignaturas y README. |

## Autor

Juan Rivera Ordoñez — I.E.S. Los Sauces, 2026-2027
