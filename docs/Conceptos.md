# DEFINICIÓN DE CONCEPTOS PARA TECHDOCS:

## Preparador de TechDocs:
El Preparador de TechDocs obtiene los archivos del código fuente del proveedor de alojamiento del código fuente (github) y los pasa al generador para los próximos pasos.

- Hay 2 tipos disponibles:
    - Preparador común de git (git clone)
    - Lector de URL (utiliza la API del proveedor para descargar los archivos)
        - Recomendado
        - Más rápido

## Generador de TechDocs:
Este paso ejecuta el contenedor de TechDocs o ejecuta mkdocs CLI para generar archivos HTML estáticos y sus archivos.

## Editor de TextDocs:
El techdocs-backend vive con 2 editores: Google Cloud y Local Filesystem.

- Un editor de techdocs es el responsable de:
    - Comunicación bidireccional entre techdocs-backend y el almacenamiento
    - Publicar archivos estáticos generados en un almacenamiento (definido en techdocs.builder)
    - Leer archivos del almacenamiento

## Contenedor de TechDocs:
Contenedor docker que crea páginas HTML estáticas, incluidas hojas de estilo y scripts de Markdown, a través de mkdocs.

## CLI de TechDocs:
Facilita la escritura, generación y vista previa de la documentación.

- Actúa como contenedor alrededor del contenedor techdocs.

## Lector de TechDocs:
La documentación generada por TechDocs se genera como sitios HTML estáticos. Por lo tanto, TechDocs Reader se creó para poder integrar sitios HTML pregenerados con la interfaz de usuario de Backstage.

## Transformadores de TechDocs:
Proporcionan una manera de transformar el contenido HTML en la renderización previa y posterior.

## Complementos de TechDocs:
Los complementos pueden cargar y mostrar información dinámicamente en cualquier lugar de TechDocs Reader.
