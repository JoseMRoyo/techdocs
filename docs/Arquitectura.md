## ARQUITECTURA BÁSICA VS ARQUITECTURA RECOMENDADA


-Cuando abre un sitio TechDocs en Backstage, TechDocs Reader realiza una solicitud de techdocs-backendcomplemento con el ID de la entidad y la ruta de la página actual que está viendo. En respuesta, recibe los archivos estáticos (HTML, CSS, JSON, etc.) para representar en la página en TechDocs/Backstage.

-Los archivos estáticos constan de HTML, CSS e imágenes generadas por MkDocs. Eliminamos todo el JavaScript antes de agregarlo a Backstage por razones de seguridad. Y hay un techdocs_metadata.jsonarchivo adicional que TechDocs necesita para representar un sitio. Es importante que utilice techdocs-cli o techdocs-container para generar los documentos para el resultado esperado.

-Luego, TechDocs Reader aplica una lista de "Transformadores" (consulte Conceptos ) que modifican los archivos HTML estáticos generados para una serie de casos de uso, por ejemplo, eliminar ciertos encabezados, filtrar algunas etiquetas HTML, etc.

-Actualmente, utilizamos el sistema de archivos local del servidor Backstage (o techdocs-backend) para almacenar los archivos generados.

Ver --> [Diagrama de la arquitectura básica de TechDocs](https://backstage.io/assets/images/architecture-basic.drawio-00e1ca112fb1eb62a3918e5310185194.svg)

-Sin embargo, lo ideal es utilizar un sistema de almacenamiento externo (por ejemplo, AWS S3, GCS o Azure Blob Storage) y utilizar una canalización de CI/CD con el repositorio que tenga un paso/trabajo dedicado para generar documentos para TechDocs. Los archivos estáticos generados se almacenan en una solución de almacenamiento en la nube de su elección. 

Ver --> [Diagrama de la arquitectura recomendada para TechDocs](https://backstage.io/assets/images/architecture-recommended.drawio-b90a644e7ae6f63987a9e5c50efdcb40.svg)

-De manera similar a como se hace en la configuración básica, TechDocs Reader solicita techdocs-backendun complemento para el sitio de documentos. techdocs-backendluego solicita a su solución de almacenamiento configurada los archivos necesarios y los devuelve a TechDocs Reader.
