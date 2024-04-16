# TECHDOCS

- Solución de código abierto para tratar la documentación a través de archivos markdown.
- TechDocs Addon Framework agrega funciones además de la experiencia básica de código similar a un documento.
- El complemento mkdocs mejora la experiencia de lectura de los documentos.

## PROVEEDORES:

### De almacenamiento de código:
- Github
- GitLab
- Bitbucket
- Azure DevOps
- Gerrit
- AWS

### De almacenamiento de archivos:
- Local (sistema de archivos de Backstage)
- Nube de Google (GCS)
- AWS
- Almacenamiento de blobs de Azure
- OpenStack

## Ubicación de los plugins necesarios:
- `@backstage/plugin-techdocs`
- `@backstage/plugin-techdocs-react`
- `@backstage/plugin-techdocs-backend`
- `@techdocs/cli`
- `contenedor-techdocs`

## INTEGRACIÓN DEL PLUGIN:

- Lo más probable es que TechDocs venga ya instalado; si no, ejecutar en el directorio raíz del proyecto:
    ```bash
    yarn --cwd packages/app add @backstage/plugin-techdocs
    ```

- En `packages/app/src/App.tsx`:
    ```typescript
    import {
        DefaultTechDocsHome,
        TechDocsIndexPage,
        TechDocsReaderPage,
    } from '@backstage/plugin-techdocs';

    const AppRoutes = () => {
        <FlatRoutes>
            {/*...otras rutas de plugins...*/}
            <Route path="/docs" element={<TechDocsIndexPage/>}>
                <DefaultTechDocsHome/>
            </Route>
            <Route
                path="/docs/:namespcae/:kind/:name/*"
                element={<TechDocsReaderPage/>}
            />
        </FlatRoutes>;
    };
    ```

- Para usar TechDocs Addon:
    ```typescript
    import { TechDocsAddons } from '@backstage/plugin-techdocs-react';
    import { ReportIssue } from '@backstage/plugin-techdocs-module-addons-contrib';

    const AppRoutes = () => {
        <FlatRoutes>
            {/*...otras rutas de plugins...*/}
            <Route path="/docs" element={<TechDocsIndexPage/>}>
                <DefaultTechDocsHome/>
            </Route>
            <Route
                path="/docs/:namespace/:kind/:name/*"
                element={<TechDocsReaderPage/>}
            >
                <TechDocsAddons>
                    <ReportIssue />
                </TechDocsAddons>
            </Route>
        </FlatRoutes>;
    };
    ```

- Agregar complemento TechDocs backend (API antiguo):
    ```bash
    yarn --cwd packages/backend add @backstage/plugin-techdocs-backend
    ```

    - Creamos archivo `packages/backend/src/plugins/techdocs.ts`:

        ```typescript
        import { DockerContainerRunner } from '@backstage/backend-common';
        import {
            createRouter,
            Generators,
            Preparers,
            Publisher,
        } from '@backstage/plugin-techdocs-backend';
        import Docker from 'dockerode';
        import { Router } from 'express';
        import { PluginEnvironment } from '../types';

        export default async function createPlugin(
            env: PluginEnvironment,
        ): Promise<Router> {
            // Preparers are responsible for fetching source files for documentation.
            const preparers = await Preparers.fromConfig(env.config, {
                logger: env.logger,
                reader: env.reader,
            });

            // Docker client (conditionally) used by the generators, based on techdocs.generators config.
            const dockerClient = new Docker();
            const containerRunner = new DockerContainerRunner({ dockerClient });

            // Generators are used for generating documentation sites.
            const generators = await Generators.fromConfig(env.config, {
                logger: env.logger,
                containerRunner,
            });

            // Publisher is used for
            // 1. Publishing generated files to storage
            // 2. Fetching files from storage and passing them to TechDocs frontend.
            const publisher = await Publisher.fromConfig(env.config, {
                logger: env.logger,
                discovery: env.discovery,
            });

            // checks if the publisher is working and logs the result
            await publisher.getReadiness();

            return await createRouter({
                preparers,
                generators,
                publisher,
                logger: env.logger,
                config: env.config,
                discovery: env.discovery,
                cache: env.cache,
            });
        }
        ```

    - Es posible que se necesite el paquete `dockerode`.

    - Importamos el complemento backend de TechDocs en el backend de Backstage:
        - En `packages/backend/src/index.ts`:

            ```typescript
            import techdocs from './plugins/techdocs';

            async function main(){
                const techdocsEnv = useHotMemoize(module, ()=> createEnv('techdocs'));

                apiRouter.use('/techdocs', await techdocs(techdocsEnv));
            }
            ```

- Agregar complemento TechDocs backend (API nuevo):
    ```bash
    yarn --cwd packages/backend add @backstage/plugin-techdocs-backend
    ```

    - En `backend/src/index.ts`:

        ```typescript
        backend.add(import('@backstage/plugin-search-backend-module-catalog/alpha'));

        backend.start();
        ```

- Definiciones necesarias:
    - Si queremos que TechDocs backend sea el responsable de generar los sitios de documentación:
        
        (Fichero `app-config.yaml`)
        ```yaml
        techdocs:
            builder: 'local'
        ```

    - Para que Backstage asuma que los sitios se generan en la canalización de CI/CD de cada entidad:
        
        (Fichero `app-config.yaml`)
        ```yaml
        techdocs:
            builder: 'external'
        ```

    - Techdocs necesita saber dónde almacenar los sitios de documentación y dónde recuperarlos.
        - Esto lo gestiona un EDITOR.

        ```yaml
        techdocs:
            builder: 'local'
            publisher:
                type: 'local'
        ```

    - El contenedor TechDocs backend ejecuta un contenedor acoplable con mkdocs instalado para generar la interfaz de los documentos a partir de archivos Markdown. Si está implementando Backstage usando Docker, significa que su contenedor intentará ejecutar otro contenedor Docker para TechDocs backend.

        - SOLUCIÓN -> Cambiar que mkdocs lo ejecute 'local' en vez de Docker (que es como viene por defecto)

            ```yaml
            techdocs:
                builder: 'local'
                publisher:
                    type: 'local'
                generator:
                    runIn: local
            ```

        - Para que funcione tenemos que agregar las siguientes líneas al fichero `Dockerfile`:

            ```Dockerfile
            RUN apt-get update && apt-get install -y python3 python3-pip python3-venv

            ENV VIRTUAL_ENV=/opt/venv
            RUN python3 -m venv $VIRTUAL_ENV
            ENV PATH="$VIRTUAL_ENV/bin:$PATH"

            RUN pip3 install mkdocs-techdocs-core
            ```
``

## Vamos a añadir los primeros documentos:
    -Lo primordial es tener esta estructura en cada complemento, en mi caso como usaré un componente de ejemplo, dicho componente se encuentra en backstagejose/examples/entities.yaml 
    (concretamente el example-website). Muestro la entructura de directorios que poseo para que sea más sencilla la explicación

        ├── backstagejose
            ├── examples
                ├── template
                    ├── content
                        ├── docs
                            ├── index.md (documento a mostrar)
                        ├── catalog-info.yaml
                        ├── mkdocs.yml
                    ├── template.yaml
                ├── entities.yaml 
                ├── mkdocs.yml


    -Necesitamos esta anotación, en mi caso se la añado al componente y le defino la ruta en la cual encontramos esta estructura:

        ├── entities.yaml (según la guía aquí debería estar el archivo catalog-info.yaml, pero como queremos asignarselo a la entidad de prueba, la anotación tenemos que metersela al fichero entities.yaml)
        ├── mkdocs.yml
        └── docs
            └── index.md

        ```yaml
        backstage.io/techdocs-ref: dir:. 
        ```
            -Esta es la ruta por defecto para techdocs, en nuestro caso no es esta, la correcta sería la siguiente:

        ```yaml
        backstage.io/techdocs-ref: dir:./template/content/
        ```
            -Ya que nuestro directorio docs/ se encuentra en examples/template/content/

    -Una vez definida la ruta en la que encontrar los documentos a mostrar, necesitamos definir el archivo mkdocs.yml de la siguiente forma:

        ```yaml
        site_name: TechDocs-Jose
        site_description: An informative description
        plugins:
        - techdocs-core
        nav:
        - Getting Started: index.md (Esto nos indica que fichero tiene que mostrar primero)
        ```

        -Hemos de definir los 2 archivos llamados mkdocs.yml, he probado a eliminar uno de los 2 pero entonces no lo hace correctamente

    -De esta forma al entrar dentro del componente en su apartado "DOCS", nos buscará en la ruta definida por la anotación y mostrará el fichero que hemos definido en mkdocs.yml. Así como un índice con el resto de los documentos que guardemos en dicha carpeta. Esos documentos solo aparecerán dentro del componente, ya que es en el único sitio en el que hemos definido la ruta para llegar hasta ellos.

## VISTA PREVIA:
-Ejecutando lo siguiente en una terminal podemos obtener una vista previa de sus documentos dentro de una instancia local de Backstage y obtener recargas en vivo de los cambios. Esto es útil cuando desea obtener una vista previa de su documentación mientras escribe:

    ```bash
    cd /path/to/docs-repository/
    npx @techdocs/cli serve
    ```