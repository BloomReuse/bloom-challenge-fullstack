# Anita Marti Campos
## Proceso de Desarrollo y Visión del Proyecto

El objetivo de esta mini aplicación de entrevista  es construir una plataforma para visualizar Preguntas Frecuentes (FAQs) personalizadas por marca. El desarrollo se ha enfocado en crear una solución simple, separando claramente las responsabilidades entre el frontend y el backend.

El flujo de usuario principal es simple pero efectivo:
1.  El usuario llega a una página principal donde se listan todas las marcas disponibles.
2.  Al seleccionar una marca, es redirigido a una página dedicada a esa marca.
3.  En esta página, se muestra el header, logo y una sección de FAQs generadas dinámicamente según la configuración específica de la marca, como sus métodos de pago, opciones de envío y tarifas.

Este enfoque narrativo guía la arquitectura y las decisiones técnicas tomadas a lo largo del proyecto.

### Ejemplo estático del Flujo
![alt text](image.png)


[screen-capture.webm](https://github.com/user-attachments/assets/eedf0587-f365-43bd-ac54-26021fde05de)

## Como levantar el proyecto
Para ejecutar el proyecto en modo producción es necesario instalar las dependencias, generar el build optimizado y levantar el servidor que servirá los archivos construidos. Este flujo permite validar el comportamiento real de la aplicación tal como funcionará en un entorno productivo.

### Pasos de instalación

1. Clonar el repositorio
```bash
git clone <url-del-repositorio>
cd <nombre-del-proyecto>
```

2. Instalar dependencias

```bash
yarn install
```

Esto descarga todas las librerías necesarias definidas en package.json.

3. Configurar variables de entorno
Crear un archivo .env (o .env.production si el proyecto lo requiere) con las variables necesarias para producción.

4. Levantar el proyecto en modo producción

* Generar el build
```bash
yarn build
```
Este comando compila y optimiza el código fuente.

* Ejecutar el build
```bash
yarn start
```

Esto inicia el servidor que carga los archivos generados.
La aplicación correrá en un entorno equivalente a producción, permitiendo validar rendimiento, rutas, carga de assets y comportamiento general.

## Arquitectura y Separación de Responsabilidades

La arquitectura del proyecto se divide en dos componentes principales: un **frontend** construido con Next.js (App Router) y un **backend** con Express y TypeScript. (base otorgada por bloom reuse)

### Backend (Express + TypeScript)

El backend es responsable de la lógica de negocio y la exposición de datos a través de una API REST. Su estructura sigue un patrón de capas para una clara separación de responsabilidades.

```
backend/
└── src/
    ├── app.ts                # Configuración del servidor Express y registro de rutas.
    ├── data/                 # Simulación de la base de datos con datos estáticos.
    │   ├── brands.ts
    │   └── brands-settings.ts
    ├── models/               # Definiciones de tipos y modelos de datos.
    │   ├── brand.ts
    │   └── brand-setting.ts
    ├── services/             # Lógica de negocio centralizada.
    │   ├── brand.ts
    │   └── faq.ts
    ├── controllers/          # Manejadores de solicitudes HTTP.
    │   └── brand.ts
    └── routes/               # Definición de las rutas de la API.
        ├── brands.ts
        └── index.ts
```

-   **`routes/`**: Define los endpoints de la API (`/brands`, `/brands/:id/faqs`). Actúa como el punto de entrada para las solicitudes HTTP y las delega a los controladores correspondientes.

-   **`controllers/`**: Orquesta el flujo de una solicitud. Recibe la petición, llama a los servicios necesarios para obtener o procesar datos y formula la respuesta HTTP. Por ejemplo, `getBrandById` en `brand.ts` coordina llamadas a `BrandService` y `FAQService`. Notar que esta separación no existía en el repositorio base, la razón de agregarla es debido a que los controladores pueden servir para orquestar diferentes servicios y que los servicios puedan funcionar de manera más desacoplada entre ellos, permitiendo enfocarse en lógicas más minimalistas.

-   **`services/`**: Encapsula la lógica de negocio. `BrandService` se encarga de acceder a los datos de las marcas, mientras que `FAQService` contiene la lógica clave para generar dinámicamente las FAQs basándose en la configuración de cada marca (`BrandSettings`). Esta centralización es crucial para mantener la consistencia.

    Si bien en este proyecto se decidió mantener la generación de FAQs en el backend, es técnicamente posible trasladar esta lógica al frontend. Hacerlo tendría implicaciones directas en la arquitectura: el frontend pasaría a interpretar y procesar configuraciones provenientes del servidor, lo que aumenta su complejidad, exige mayor coordinación entre ambos lados y puede generar duplicación de reglas si no se administra correctamente.

    En este ejemplo, mantener la construcción de las FAQs en el backend es suficiente porque reduce la carga del frontend, evita inconsistencias y asegura que cualquier modificación en la lógica de negocio se realice en un solo punto. Sin embargo, si se buscara mayor flexibilidad en la capa de presentación, personalización dinámica por parte del cliente o experimentación A/B en tiempo real, podría considerarse mover parte de la lógica al frontend. La decisión final depende del nivel de desacoplamiento, extensibilidad y control que se desee lograr.

-   **`models/`**: Define las estructuras de datos (`Brand`, `BrandSettings`).
-   **`data/`**: Simula una base de datos con arrays estáticos, permitiendo un desarrollo rápido sin dependencias externas.

### Frontend (Next.js + App Router)

El frontend se encarga de la presentación y la interacción con el usuario. Utiliza el App Router de Next.js para aprovechar los Server Components, mejorando el rendimiento y el SEO.

```
frontend/
└── src/
    ├── app/
    │   ├── [slug]/            # Ruta dinámica para cada marca.
    │   │   ├── layout.tsx    # Layout específico de la marca, aplica el tema.
    │   │   └── page.tsx      # Página que muestra los detalles y FAQs de la marca.
    │   ├── globals.css       # Estilos globales y variables CSS para los temas.
    │   ├── layout.tsx        # Layout raíz de la aplicación.
    │   └── page.tsx          # Página de inicio que lista todas las marcas.
    ├── components/           # Componentes de UI reutilizables.
    │   ├── brand-card.tsx
    │   ├── brand-theme-provider.tsx
    │   ├── button.tsx
    │   ├── faq-item.tsx
    │   ├── faq-section.tsx
    │   └── header.tsx
    └── lib/
        ├── api/              # Cliente para consumir la API del backend.
        │   ├── brands.ts
        │   ├── config.ts
        │   ├── index.ts
        │   └── types.ts
        ├── models/           # Modelos de datos del frontend.
        │   ├── brand.ts
        │   └── brands-settings.ts
        └── utils.ts          # Funciones de utilidad.
```

-   **`app/`**: Contiene las rutas y páginas. El uso de Server Components por defecto (`page.tsx`, `[slug]/page.tsx`) permite obtener datos directamente en el servidor, renderizando el HTML inicial y enviándolo al cliente. Esto reduce el tiempo de carga percibido y es ideal para el SEO.

-   **`components/`**: Alberga los componentes de la interfaz de usuario. Se ha utilizado **shadcn/ui** para construir una UI consistente y fácilmente personalizable. Componentes como `Button` son reutilizables y se adaptan al tema de cada marca gracias a la integración con Tailwind CSS y variables CSS.

-   **`lib/api/`**: Define el cliente HTTP para comunicarse con el backend. Esta capa de abstracción es fundamental para un código limpio y mantenible.
-   **`lib/utils.ts`**: Incluye utilidades como la función `cn`, un wrapper que combina `clsx` y `tailwind-merge`. Esta función permite aplicar clases de Tailwind CSS de forma condicional sin conflictos, especialmente útil en componentes con variantes de estilo. Aunque existen alternativas más ligeras que solo manejan la concatenación de clases (como `clsx` por sí solo), `tailwind-merge` añade la ventaja de resolver conflictos de clases de Tailwind (ej. `p-2` y `p-4`), lo cual justifica su uso en este proyecto para garantizar la previsibilidad de los estilos.

## Modelos de Datos Principales

Los modelos de datos definen la estructura de la información que fluye a través de la aplicación.

#### `Brand`
Representa una marca individual.
```
+-------------+
|    Brand    |
+-------------+
| id: string  |
| name: string|
| url: string |
| logo_url?:  |
|   string    |
| settings?:  |
| BrandSettings|
+-------------+
```

#### `BrandSettings`

El modelo `BrandSettings` contiene la configuración que determina el comportamiento operativo y el contenido específico de cada marca, incluyendo métodos de pago, métodos de envío, tarifas de servicio.
```
+------------------+
|  BrandSettings   |
+------------------+
| payment:         |
|   PaymentConfig  |
| shipping:        |
|   ShippingConfig |
| service_fees:    |
|   ServiceFees    |
| only_santiago:   |
|   boolean        |
| brandId: string  |
+------------------+

```
> **🔍 Nota importante** 
> Este modelo es más amplio de lo que normalmente se utilizaría en un entorno real, ya que se diseñó para ilustrar cómo podrían representarse configuraciones complejas y combinables. En este planteamiento, tanto `PaymentConfig` como `ShippingConfig` permiten definir múltiples opciones, porcentajes, cargos y etiquetas, de manera similar a lo que sería una relación en base de datos entre tablas como `PaymentOptions`, `ShippingOptions` o `ServiceFees`.

> En una aplicación de producción, es posible que este nivel de flexibilidad no siempre sea necesario. Dependiendo de las reglas de negocio, la cantidad de opciones reales y el grado de dinamismo requerido, el modelo puede simplificarse con el uso de flags o variables directas que representen casos más acotados. Por ejemplo, si una marca solo opera con dos métodos de pago, como transferencias bancarias y gift cards, la estructura podría reducirse a propiedades simples como `acceptTransfer`, `transferAmount`, `acceptGiftCard` o `giftCardPercentage`. Estos flags permiten reducir complejidad, disminuir uso de memoria y facilitar las operaciones de lectura y escritura, especialmente cuando la lógica de negocio se mantiene estable.

> La elección entre un modelo relacional más detallado o uno basado en flags depende de los objetivos del sistema: si se busca una aplicación altamente extensible, marcas con configuraciones muy diferentes o condiciones de negocio que cambian con frecuencia, un modelo como el presentado aquí es más adecuado. En cambio, si el dominio es acotado y las reglas son relativamente estáticas, una representación basada en flags puede resultar más eficiente, sencilla de administrar y suficiente para las necesidades del proyecto.



## Flujos de Datos y Lógica de Negocio

1.  **Listado de Marcas**:
    -   El Server Component en `frontend/src/app/page.tsx` llama a la función `listBrands()` del cliente API.
    -   Esta función realiza una petición `GET /brands` al backend.
    -   El controlador correspondiente en el backend invoca a `BrandService.list()`, que recupera los datos de la simulación en `/data/brands.ts` y los devuelve como JSON.

2.  **Visualización de FAQs por Marca**:
    -   La página dinámica `frontend/src/app/[slug]/page.tsx` (Server Component) obtiene el `slug` de la marca desde la URL.
    -   Llama a `getBrandById(slug)` y `getFAQsByBrandId(slug)` para obtener los datos de la marca y sus FAQs.
    -   En el backend, el controlador de FAQs invoca a `FAQService.generateAllFAQs(brandId)`. Este servicio es el núcleo de la lógica de negocio: lee la configuración de la marca (`BrandSettings`) y genera dinámicamente el texto y la disponibilidad de cada FAQ.
    -   El frontend recibe las FAQs ya procesadas y listas para renderizar, manteniendo la lógica de presentación separada de la lógica de negocio.

## Arquitectura del Cliente API: Flexibilidad y Tipado

El cliente API en `frontend/src/lib/api/` está diseñado para ser robusto y flexible.
-   **Abstracción del Cliente HTTP**: En `config.ts`, se ha creado un wrapper simple sobre la API `fetch` nativa. Las funciones `api.get`, `api.post`, etc., centralizan la configuración de las peticiones (como headers y la URL base).

    Esta abstracción es importante porque hace que el cliente HTTP sea intercambiable. Si en el futuro se decidiera usar una librería como **Axios** por sus características avanzadas (ej. interceptores, cancelación de peticiones), solo sería necesario modificar la implementación dentro de `config.ts`. Ningún otro archivo que consume los servicios (`listBrands`, etc.) necesitaría cambios.

## Estrategia de Renderizado: Server vs. Client Components

-   **Server-Side Rendering (SSR)**: La mayoría de las páginas y layouts son Server Components por defecto. Esto significa que la obtención de datos y el renderizado inicial ocurren en el servidor.
    -   **Por qué**: Mejora el **rendimiento inicial** (el usuario recibe HTML significativo más rápido) y es fundamental para el **SEO**, ya que los motores de búsqueda pueden indexar el contenido sin necesidad de ejecutar JavaScript. (esto sirve bastante para las paginas de retail, aunque últimamente con los motores impulsados por ia y no por búsqueda no es un objetivo principal)

-   **Client-Side Rendering (CSR)**: Solo los componentes que requieren interactividad usan el hook `"use client"`.
    -   `faq-item.tsx`: Gestiona el estado de "abierto/cerrado" de cada pregunta con `useState`. La interactividad del usuario (clics) solo puede manejarse en el cliente.
    -   `brand-theme-provider.tsx`: Manipula el DOM para aplicar el atributo `data-brand`, lo que activa el tema CSS correspondiente. La manipulación del DOM es una tarea exclusiva del cliente.

## Puntos Fuertes y Posibles Mejoras

#### Puntos Fuertes

-   **Arquitectura Limpia**: La separación entre frontend y backend, y la estructura de capas en cada uno, facilita la mantenibilidad y el escalado.
-   **Lógica de Negocio Centralizada**: La generación de FAQs está encapsulada en el `FAQService` del backend, lo que significa que el frontend solo se preocupa de mostrar datos, no de cómo se crean.
-   **Rendimiento Optimizado**: El uso de Server Components en Next.js para la carga inicial de datos asegura una experiencia de usuario rápida y amigable para el SEO.
-   **UI Tematizable y Reutilizable**: Gracias a **shadcn/ui** y las variables CSS, la interfaz es consistente y se adapta visualmente a cada marca sin recargar la página.

#### Puntos a Mejorar

-   **Duplicación de Tipos**: Los modelos de datos (`Brand`, `BrandSettings`) están duplicados en el frontend y el backend. Esto es un riesgo de desincronización.
    -   **Mejora**: Crear un **paquete de tipos compartido** (por ejemplo, en un monorepo con `pnpm workspaces`) para que ambos proyectos consuman una única fuente de verdad.
-   **Simulación de Base de Datos**: El backend utiliza datos estáticos, lo que no es viable para producción.
    -   **Mejora**: Reemplazar la simulación con una **base de datos real** (ej. PostgreSQL con Prisma) y una capa de repositorio para abstraer las consultas.
-   **Manejo de Errores y Códigos HTTP**: El backend usa códigos de estado inconsistentes (ej. `201` para una lista) y carece de un formato de error estructurado.
    -   **Mejora**: Implementar un **middleware de manejo de errores** centralizado y usar códigos HTTP semánticos (ej. `200` para `GET`, `404` si no se encuentra un recurso). La `ApiException` del frontend ya está preparada para recibir errores estructurados.
-   **Falta de Pruebas**: El proyecto carece de tests automatizados.
    -   **Mejora**: Añadir **tests unitarios** para los servicios del backend (especialmente `FAQService`), y **pruebas de integración** para los endpoints de la API. En el frontend, se podrían añadir tests para los componentes interactivos. (en mis proyectos pasados me siento más cómoda con playwright y jest)

## Visión a Futuro: Escalando la Solución

Para llevar este proyecto a una escala de producción, se podrían tomar los siguientes pasos:

1.  **Monorepo y Tipos Compartidos**: Migrar a un monorepo para compartir tipos y utilidades entre frontend y backend. Además, personalmente me gusta checkear esquemas entre solicitudes, en particular en este proyecto me gustaría la implementación de `Zod` para poder trabajar con esquemas en peticiones y tambien generar middlewares de validaciones de request de manera más sencilla y aislada en la responsabilidad de los validadores
2.  **Capa de Persistencia Real**: Implementar una base de datos con un ORM como Prisma para gestionar los datos de marcas y configuraciones.
3.  **Autenticación y Autorización**: Añadir un sistema para que los administradores de marca puedan gestionar sus solicitudes a Bloom reuse.
4.  **Caching y Optimización**: Implementar estrategias de caching en Next.js (revalidate) para reducir las llamadas a la API para datos que no cambian con frecuencia (que deberían ser la mayoría, considerando que en Bloom Reuse la mayor parte del contenido es estático o semiestático). Esto permite mejorar el rendimiento, disminuir costos de infraestructura y entregar una experiencia más rápida al usuario.

    En mi caso, trabajé directamente con los enlaces utilizados por las marcas dentro de su aplicación, pero no apliqué una optimización específica para cada tipo de recurso. Este tipo de sitio —orientado a la publicación y visualización de productos— debería incorporar técnicas de optimización más avanzadas, tales como:

    -  Optimización de imágenes con next/image, utilizando tamaños responsivos, fill, quality, y dominios configurados en next.config.js.  (utilizo algunos pero no de una manera que genere diferencia en esta prueba técnica)

    - Uso de Image CDN o servicios externos como Cloudflare Images o Imgix para reducción de peso sin perder calidad. (en este caso uso directamente su link [lo inspeccioné dentro de su página, obviamente en producción y un ambiente real se debe tener más cuidado con las políticas y copy rights])

    - Static Rendering (SSG) para páginas de productos y listados que no cambian con frecuencia, disminuyendo el tiempo de respuesta. ( para las paginas partilares de los productos de cada pagina, si el cliente debe solicitar la publicación entonces es posible trabajar de antemano con los productos y cargarlos por parte del servidor)

    - Prefetching automático de rutas mediante el router de Next.js para mejorar tiempos de navegación.

En conjunto, estas prácticas permiten que un proyecto como el de Bloom Reuse —centrado en catálogos de marcas, productos reutilizados y contenido mayormente estático— funcione con un rendimiento más optimo en el build y con menor carga sobre las APIs o servidores de backend, el cual asumo se comparte entre marcas y solo se diferencian en frontend.

5.  **Observabilidad**: Integrar logging estructurado, métricas y tracing para monitorear la salud de la aplicación en producción.

6.  **CI/CD**: Configurar un pipeline de integración y despliegue continuo para automatizar las pruebas y los despliegues.


Cambio en el código para testear rabbit pr

console.log("hola")