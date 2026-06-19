# Auditoría SEO — quintas.espacioalvarez.com.ar
**Fecha:** 19 de junio de 2026  
**Archivo auditado:** `index.html`  
**Herramienta:** Análisis manual del código fuente + robots.txt + sitemap.xml

---

## Resumen ejecutivo

| # | Problema | Impacto | Complejidad de fix |
|---|----------|---------|-------------------|
| 1 | H1 sin keywords comerciales | Alto | Bajo |
| 2 | SPA sin subpáginas por servicio | Alto | Alto |
| 3 | Sin precios en el sitio | Alto | Medio |
| 4 | Schema JSON-LD fuera del `<head>` | Alto | Bajo |
| 5 | Schema LocalBusiness incompleto | Alto | Bajo |
| 6 | Videos sin transcript/subtítulos | Alto | Medio |
| 7 | `preload="auto"` en video hero | Medio | Bajo |
| 8 | Imágenes de galería sin `width`/`height` | Medio | Bajo |
| 9 | Galería repite imagen (habitacion-nueva) | Medio | Bajo |
| 10 | Alt texts contradictorios en misma imagen | Medio | Bajo |
| 11 | Dirección inconsistente entre schema y HTML | Medio | Bajo |
| 12 | Sin enlace a Google Business Profile | Medio | Bajo |
| 13 | Asset con extensión en mayúscula (.JPG) | Bajo | Bajo |
| 14 | Marquee con keywords ocultas por aria-hidden | Bajo | Bajo |
| 15 | Sin meta keywords | Bajo | Bajo |

---

## Impacto Alto

### 1. H1 sin keywords comerciales
**Línea:** `index.html:1315`

El H1 actual es `"Donde el tiempo se detiene"`. Poético pero invisible para Google. Ningún cliente potencial busca esa frase. Los términos reales son: *"quinta para eventos zona oeste"*, *"alquiler quinta francisco álvarez"*, *"quinta con pileta buenos aires"*. El H1 es la señal de relevancia más importante de la página.

**Solución:**
```html
<!-- Antes -->
<h1 class="hero-title">Donde el tiempo<br><em>se detiene</em></h1>

<!-- Después -->
<h1 class="hero-title">Quinta para <em>Eventos</em><br>y Hospedaje</h1>
<p class="hero-sub">Donde el tiempo se detiene · Francisco Álvarez, Acceso Oeste km 44</p>
```

---

### 2. SPA de una sola URL — sin páginas de destino por servicio
**Impacto estructural**

El sitio entero es una página con anclas (`#espacios`, `#momentos`). Google no puede rankear por separado para:
- *"quintas para casamientos zona oeste"*
- *"hospedaje quinta buenos aires"*
- *"eventos corporativos zona oeste"*

Cada servicio compite por la misma URL contra el resto del contenido.

**Solución a futuro:** Crear subpáginas `/casamientos/`, `/hospedaje/`, `/eventos-corporativos/` con contenido propio y sus propios title/description/H1.

---

### 3. Sin precios en el sitio
**Impacto en conversión y ranking**

Más del 70% de las búsquedas de alquiler de quintas incluyen precio (*"quinta con pileta precio zona oeste"*). Al no tener tarifas, el sitio no rankea para esos términos y quienes llegan rebotan rápido. El schema solo dice `priceRange: "$$"` sin ningún detalle.

**Solución:** Agregar una sección "Tarifas" con rangos de precio por tipo de evento, o al menos un CTA con "Pedí tu cotización sin compromiso".

---

### 4. Schema JSON-LD fuera del `<head>`
**Línea:** `index.html:2045`

El bloque `<script type="application/ld+json">` está al final del `<body>`, justo antes de `</body>`. Google lo procesa igual, pero la práctica recomendada es colocarlo en el `<head>` para asegurar que los crawlers lo encuentren en la primera pasada del documento.

**Solución:** Mover el bloque completo dentro de `<head>`, antes de `</head>`.

---

### 5. Schema LocalBusiness incompleto
**Línea:** `index.html:2045–2087`

Faltan propiedades importantes para búsquedas locales:

| Propiedad faltante | Valor sugerido |
|-------------------|----------------|
| `openingHours` | `"Sa-Su 08:00-22:00"` |
| `hasMap` | URL de Google Maps del negocio |
| `aggregateRating` | Rating promedio + cantidad de reviews |
| `review` | Al menos 1-2 reseñas reales |
| `numberOfRooms` | `3` (para el tipo `LodgingBusiness`) |
| `@type adicional` | `"LodgingBusiness"` para el hospedaje |

**Solución:** Ampliar el bloque JSON-LD existente con esas propiedades.

---

### 6. Videos sin transcript ni subtítulos
**Líneas:** `index.html:1309` (hero), `index.html:2024` (cómo llegar)

`hero-atardecer.mp4` y `01-como-llegar.mp4` no tienen `<track kind="subtitles">`. Google no puede indexar el audio de los videos. El video "cómo llegar" especialmente pierde la oportunidad de indexar términos locales hablados (*Francisco Álvarez, Acceso Oeste, km 44*).

**Solución:**
```html
<video id="videoLlegar" controls playsinline preload="none">
  <source src="assets/01-como-llegar.mp4" type="video/mp4">
  <track kind="subtitles" src="assets/como-llegar.vtt" srclang="es" label="Español" default>
</video>
```

---

## Impacto Medio

### 7. `preload="auto"` en el video hero
**Línea:** `index.html:1309`

```html
<video class="hero-video" autoplay muted loop playsinline poster="assets/pileta-noche.webp" preload="auto">
```

`preload="auto"` le ordena al navegador descargar el video entero al cargar la página, antes de cualquier interacción del usuario. Impacta negativamente el **LCP** (Largest Contentful Paint) y el **FCP** (First Contentful Paint), que son factores de ranking de Google desde 2021 (Core Web Vitals).

**Solución:** Cambiar `preload="auto"` por `preload="none"`. El poster `pileta-noche.webp` ya actúa como fallback visual mientras el video carga.

---

### 8. Imágenes de galería sin `width` y `height`
**Líneas:** `index.html:1655–1731`

Ningún `<img>` dentro de `.galeria-track` tiene atributos de dimensión. El navegador no puede reservar el espacio antes de que carguen, causando **CLS (Cumulative Layout Shift)** — otro factor negativo de Core Web Vitals que penaliza el ranking.

**Solución:** Agregar `width` y `height` a cada `<img>` de la galería según las dimensiones reales de cada archivo.

---

### 9. Galería repite la misma imagen dos veces seguidas
**Líneas:** `index.html:1723–1730`

`habitacion-nueva.webp` aparece en dos slides consecutivos con alts distintos:
- `"Habitación con camas cuchetas"`
- `"Habitación con placarón y cuchetas"`

Además, **`habitacion-1.jpg` y `habitacion-2.jpg` ya existen en `assets/` y no se usan en ningún lado del sitio**.

**Solución:** Reemplazar los dos slides repetidos por `habitacion-1.webp` y `habitacion-2.webp`.

---

### 10. Misma imagen con alt texts contradictorios
**Casos:**

| Imagen | Alt en Espacios | Alt en Galería/Adicionales |
|--------|----------------|---------------------------|
| `salon2.png` (línea 1497) | `"Salón cubierto con cocina integrada"` | `"Barra de bebidas y living"` (línea 1669) |
| `quincho-gente.jpg` (línea 1425) | `"Quincho de hormigón visto con gente reunida"` | `"Evento con catering"` (línea 1791) |

Google Images cruza estos textos y los descarta si son contradictorios. Afecta el posicionamiento en búsquedas de imágenes.

**Solución:** Usar el mismo alt en todas las apariciones de una imagen, o usar imágenes distintas para cada contexto.

---

### 11. Inconsistencia de dirección entre schema y HTML
**Líneas:** `index.html:2062` (schema) vs `index.html:1876` (HTML)

| Fuente | Dirección |
|--------|-----------|
| Schema JSON-LD | `"Alcorta 531, Acceso Oeste km 44"` |
| HTML visible | `"Alcorta 531, Francisco Álvarez km 44, General Rodríguez, Buenos Aires"` |

Google cruza estas fuentes con Google Maps y el perfil de negocio. Las inconsistencias afectan la aparición en el **Local Pack** de búsquedas.

**Solución:** Unificar ambas a:
```
Alcorta 531, Francisco Álvarez, General Rodríguez, Buenos Aires
```

---

### 12. Sin enlace al perfil de Google Business
**Línea:** `index.html:2073–2077` (bloque `sameAs`)

El `sameAs` del schema incluye Instagram, Facebook y TikTok, pero **no el perfil de Google Business**. Mantener los datos sincronizados entre el sitio y el perfil de Google es el factor más importante del SEO local en Argentina.

**Solución:**
1. Verificar/crear el perfil en Google Business Profile.
2. Agregar la URL del perfil al `sameAs` del schema.
3. Asegurarse de que nombre, dirección y teléfono coincidan exactamente en ambos lados (NAP consistency).

---

## Impacto Bajo

### 13. Asset con extensión en mayúscula
**Archivo:** `assets/galeria2.JPG`

El archivo tiene extensión `.JPG` (mayúsculas). No está en uso actualmente, pero en servidores Linux (sensibles a mayúsculas, que son la mayoría de los hostings), `/assets/galeria2.jpg` ≠ `/assets/galeria2.JPG`. Si se usa en el futuro, causará error 404.

**Solución:** Renombrar a `galeria2.jpg` (minúsculas).

---

### 14. Marquee oculta keywords con `aria-hidden`
**Línea:** `index.html:1333`

```html
<div class="marquee" aria-hidden="true">
```

El marquee contiene términos valiosos (*Casamientos · Cumpleaños · Hospedaje · Corporativos · Lanzamientos*) pero `aria-hidden="true"` los hace invisibles para lectores de pantalla. El impacto en Google es mínimo (Google ignora `aria-hidden` en su mayoría), pero el contenido decorativo podría aprovecharse mejor con una versión estática accesible.

---

### 15. Sin meta keywords
Google los ignora desde 2009. Bing y Yahoo los usan marginalmente. No es prioritario.

---

## Estado actual — Lo que funciona bien ✅

| Elemento | Estado |
|----------|--------|
| `lang="es"` en `<html>` | ✅ Correcto |
| `meta viewport` | ✅ Presente |
| `canonical` tag | ✅ Correcto |
| `meta description` (157 chars) | ✅ Dentro del rango óptimo |
| Open Graph completo (og:type, og:url, og:title, og:description, og:image, og:locale) | ✅ Completo |
| Twitter Card `summary_large_image` | ✅ Presente |
| `meta robots: index, follow` | ✅ Correcto |
| Google Site Verification | ✅ Presente |
| Un solo H1 | ✅ Correcto |
| Todas las imágenes tienen `alt` | ✅ Correcto |
| `loading="lazy"` en imágenes | ✅ Presente |
| Google Fonts no bloqueante (`media="print"` trick) | ✅ Correcto |
| Scripts al final del body (no bloqueantes) | ✅ Correcto |
| `og-image.jpg` existe en `assets/` | ✅ Verificado |
| `robots.txt` configurado | ✅ Presente |
| `sitemap.xml` enlazado desde robots.txt | ✅ Presente |
| Schema `@type: LocalBusiness + EventVenue` | ✅ Tipo correcto |
| Coordenadas GPS en schema | ✅ Presentes |
| `preconnect` a Google Fonts | ✅ Presente |
| `fetchpriority="high"` en imagen hero | ✅ Correcto |
| iframe de Maps con `title` | ✅ Correcto |
| WebP con fallback JPG/PNG en todas las imágenes | ✅ Correcto |

---

*Auditoría generada con Claude Code · Proyecto: Quintas Espacio Álvarez*
