## COMPLEMENTOS:

-Los complementos son un mecanismo mediante el cual puede personalizar la experiencia de TechDocs para abordar necesidades de orden superior

-Un complemento es un componente de react, y puede recuperar y representar datos usando backstage, API y componentes nativos

-Los complementos declaran "location" donde serán renderizados:
    -Header (complementos que llenan el encabezado desde la derecha, en la misma línea que el título)
    -Subheader (debajo del encabezado pero sobre todo el contenido) (Excelente para herramientas/configuración de la pantalla de techdocs)
    -Settings (se agregan a la lista del menú configuración)
    -PrimarySideBar (a la izquierda del contenido, encima de la navegación)
    -SecondarySideBar (a la derecha del contenido, encima de la tabla)
    -Content (ubicación especial para complementos que aumentan el contenido)
    -Component (ubicación virtual)

    Ver --> [Esquema de localizaciones de los complementos](https://backstage.io/assets/images/addon-locations-684d47051de7f2b16c25d8354d4f1f2f.png)

-La instalación y configuración de complementos se realiza dentro de la interfaz de una aplicación Backstage. Los complementos se importan desde complementos y se agregan debajo de un componente de registro
<TechDocsReader>. Se puede configurar tanto para TechDocs Reader como para Entity Docs.

-Los complementos se muestran en el órden que se registran

-INSTALACIÓN Y USO:

Para comenzar a usar complementos, debe agregar el paquete @backstage/plugin-techdocs-module-addons-contrib a su aplicación:

    ```bash
    yarn --cwd packages/app add @backstage/plugin-techdocs-module-addons-contrib
    ```

-En el archivo packages/app/src/App.tsx:

    ```typescript
    import { TechDocsReaderPage } from '@backstage/plugin-techdocs';
    import { TechDocsAddons } from '@backstage/plugin-techdocs-react';
    import { ReportIssue } from '@backstage/plugin-techdocs-module-addons-contrib';

    // ...

    <Route path="/docs/:namespace/:kind/:name/*" element={<TechDocsReaderPage />}>
    <TechDocsAddons>
        <ReportIssue />
        {/* Other addons can be added here. */}
    </TechDocsAddons>
    {techDocsPage} // This is your custom TechDocs reader page
    </Route>;
    ```

-Configurar complementos en la pestaña de documentación en la página de la entidad:

    ```typescript
    // packages/app/src/components/catalog/EntityPage.tsx

    import { EntityLayout } from '@backstage/plugin-catalog';
    import { EntityTechdocsContent } from '@backstage/plugin-techdocs';
    import { TechDocsAddons } from '@backstage/plugin-techdocs-react';
    import { ReportIssue } from '@backstage/plugin-techdocs-module-addons-contrib';

    // ...

    <EntityLayout.Route path="/docs" title="Docs">
    <EntityTechdocsContent>
        <TechDocsAddons>
        <ReportIssue />
        {/* Other addons can be added here. */}
        </TechDocsAddons>
    </EntityTechdocsContent>
    </EntityLayout.Route>;
    ```

-COMPLEMENTOS DISPONIBLES:
<ExpandableNavigation />	@backstage/plugin-techdocs-module-addons-contrib	
Permite a los usuarios de TechDocs expandir o contraer toda la navegación principal de TechDocs y mantiene el estado preferido del usuario entre los sitios de documentación.

<ReportIssue />	@backstage/plugin-techdocs-module-addons-contrib
Permite a los usuarios de TechDocs seleccionar una parte del texto en una página de TechDocs y abrir un problema en el repositorio que contiene la documentación, completando la descripción del problema con el texto seleccionado de acuerdo con una plantilla configurable.

<TextSize />	@backstage/plugin-techdocs-module-addons-contrib	
Este complemento de TechDocs permite a los usuarios personalizar el tamaño del texto en las páginas de documentación, pueden seleccionar cuánto desean aumentar o disminuir el tamaño de fuente mediante el control deslizante o los botones. El valor predeterminado para el tamaño de fuente es 100% y esta configuración se mantiene en el almacenamiento local del navegador cada vez que se cambia.

<LightBox />	@backstage/plugin-techdocs-module-addons-contrib	
Este complemento de TechDocs permite a los usuarios abrir imágenes en un cuadro de luz en páginas de documentación, pueden navegar entre imágenes si hay varias en una página. El tamaño de la imagen de la caja de luz es el mismo que el tamaño de la imagen en la página del documento. Al hacer clic en el icono de zoom, se amplía la imagen para que quepa en la pantalla (similar a background-size: contain).


-CREACIÓN DE UN COMPLEMENTO:
Los complementos más simples son componentes de react antiguos y simples que se representan en ubicaciones específicas dentro de un sitio TechDocs. Para empaquetar dicho componente de reacción como un complemento, siga estos pasos:

    -Crear archivo plugins/your-plugin/src/plugin.ts:

    ```typescript
    import {
        createTechDocsAddonExtension,
        TechDocsAddonLocations,
    } from '@backstage/plugin-techdocs-react';
    import { CatGifComponent, CatGifComponentProps } from './addons';

    // ...

    // You must "provide" your Addon, just like any extension, via your plugin.
    export const CatGif = yourPlugin.provide(
    // This function "creates" the Addon given a component and location. If your
    // component can be configured via props, pass the prop type here too.
    createTechDocsAddonExtension<CatGifComponentProps>({
        name: 'CatGif',
        location: TechDocsAddonLocations.Header,
        component: CatGifComponent,
    }),
    );
    ```