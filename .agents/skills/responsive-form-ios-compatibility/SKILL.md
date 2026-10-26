---
name: responsive-form-ios-compatibility
description: >-
  Estándar de ingeniería frontend para la creación de formularios y páginas web perfectamente adaptables
  (responsive) a cualquier tamaño de pantalla (desde móviles de 320px hasta monitores 4K), garantizando
  compatibilidad técnica rigurosa con iOS y iPadOS desde la versión 12 hasta la más reciente.
---

# Diseño y Código Responsive Universal (Compatibilidad iOS 12+ a 4K)

Esta habilidad proporciona las directrices técnicas, patrones de CSS y buenas prácticas de JavaScript para construir formularios web que no fallen en ningún dispositivo, con especial énfasis en la compatibilidad con **Mobile Safari en iPhone e iPad (desde iOS 12 hasta la versión actual)** y pantallas ultra-pequeñas o ultra-grandes.

---

## 1. La Regla de Oro del Zoom en iOS (Evitar el Auto-Zoom Molesto)

En iOS Safari (desde iOS 12 hasta iOS 18+), si un campo de entrada tiene un tamaño de letra inferior a **16px**, el navegador **fuerza un zoom automático sobre la pantalla** cuando el usuario toca el campo para escribir. Esto descompone la maqueta, desplaza el formulario horizontalmente y obliga al usuario a alejar la pantalla manualmente.

### Regla Obligatoria en CSS:
```css
/* Todos los campos interactivos DEBEN tener mínimo 16px en pantallas móviles */
input[type="text"],
input[type="email"],
input[type="tel"],
input[type="number"],
input[type="date"],
input[type="password"],
select,
textarea {
    font-size: 16px !important;
}

@media (min-width: 641px) {
    /* En tabletas y escritorio se puede permitir un tamaño menor si el diseño lo pide */
    input, select, textarea {
        font-size: 0.95rem;
    }
}
```

---

## 2. Configuración Esencial del Viewport y Safe Area (Notch e Isla Dinámica)

Desde el iPhone X hasta los modelos más recientes con Dynamic Island, el diseño debe respetar las zonas seguras (*Safe Area Insets*).

### 1. Etiqueta Meta en `<head>`:
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=5.0, viewport-fit=cover">
```
*(Nota: No uses `user-scalable=no` ni `maximum-scale=1.0` porque perjudica la accesibilidad de personas con baja visión y puede ser penalizado).*

### 2. Variables de entorno Safe Area en CSS:
```css
body {
    padding-top: env(safe-area-inset-top, 0px);
    padding-bottom: env(safe-area-inset-bottom, 0px);
    padding-left: env(safe-area-inset-left, 0px);
    padding-right: env(safe-area-inset-right, 0px);
}
```

---

## 3. Solución al Error de Altura `100vh` en Safari Móvil

En Safari de iOS, `100vh` calcula la altura incluyendo el espacio detrás de las barras de herramientas dinámicas superior e inferior. Cuando la barra se expande, los botones situados al final de modales o pantallas quedan ocultos.

### Solución con Fallback Progresivo:
```css
.full-screen-container {
    min-height: 100vh; /* Fallback universal para navegadores antiguos (iOS 12-14) */
    min-height: -webkit-fill-available;
    min-height: 100dvh; /* Soporte moderno para Dynamic Viewport Height (iOS 15.4+) */
}
```

---

## 4. Compatibilidad Crítica de Flexbox `gap` en iOS 12 a 14.4

> ⚠️ **Peligro en WebKit:** Safari en versiones de iOS anteriores a la **14.5** **NO soporta la propiedad `gap` dentro de contenedores `display: flex`**. Si se usa `flex gap-4`, en un iPhone con iOS 12 o 13 los elementos se solaparán o quedarán pegados sin separación.

### Pautas para maquetar con seguridad:
1. **Para grillas y columnas:** Usar siempre **CSS Grid** (`display: grid; gap: 1rem;`). CSS Grid tiene soporte completo en WebKit desde iOS 10.3.
2. **Si se usa Flexbox:** Aplicar márgenes tradicionales o utilidades de espaciado en los elementos hijos:
   ```html
   <!-- En lugar de: <div class="flex gap-3"> -->
   <!-- Usar CSS Grid: -->
   <div class="grid grid-cols-1 sm:grid-cols-2 gap-4"> ... </div>

   <!-- O margen en elementos hijos: -->
   <div class="flex flex-wrap -mx-2">
       <div class="px-2 mb-3">...</div>
   </div>
   ```

---

## 5. Reseteo de Apariencia Nativa en iOS WebKit

iOS aplica estilos propietarios pesados (degradados grises, bordes redondeados forzados y sombras internas) a botones y campos de texto.

```css
/* Limpieza de estilos nativos de iOS */
input, textarea, select, button {
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    border-radius: 0; /* Permite aplicar el radio personalizado sin artefactos de iOS */
}

/* En iOS 12, los select requieren volver a declarar el icono de flecha si se quita el appearance */
select {
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 20 20'%3E%3Cpath stroke='%236b7280' stroke-linecap='round' stroke-linejoin='round' stroke-width='1.5' d='M6 8l4 4 4-4'/%3E%3C/svg%3E");
    background-position: right 0.75rem center;
    background-repeat: no-repeat;
    background-size: 1.25em 1.25em;
    padding-right: 2.5rem !important;
}

/* Eliminar el resplandor azul grisáceo al tocar en Safari */
* {
    -webkit-tap-highlight-color: transparent;
}
```

---

## 6. Teclados Virtuales y Atributos de Entrada Móvil

Mejoran radicalmente la experiencia del usuario abriendo el teclado adecuado en su teléfono:

```html
<!-- Para teléfonos: abre teclado numérico con signos telefónicos -->
<input type="tel" name="telefono" inputmode="tel" autocomplete="tel">

<!-- Para DNI/NIE o códigos numéricos: teclado numérico puro -->
<input type="text" name="codigo" inputmode="numeric" pattern="[0-9]*">

<!-- Para emails: abre teclado con @ y sin autocapitalización molesta -->
<input type="email" name="email" inputmode="email" autocomplete="email" autocapitalize="none" spellcheck="false">

<!-- Para nombres propios: autocapitaliza la primera letra de cada palabra -->
<input type="text" name="nombre" autocomplete="name" autocapitalize="words">
```

---

## 7. Áreas Táctiles Mínimas (Human Interface Guidelines de Apple)

- **Tamaño mínimo de toque:** Cualquier botón, casilla de verificación (*checkbox*), botón de opción (*radio*) o enlace debe tener un área táctil de al menos **44 × 44 píxeles**.
- **Solución para Checkboxes y Radios:** Envolver siempre el input en una etiqueta `<label>` amplia que cubra todo el texto:
  ```html
  <label class="flex items-start gap-3 p-3.5 rounded-xl border border-slate-200 bg-white cursor-pointer select-none">
      <input type="checkbox" class="w-5 h-5 mt-0.5 rounded text-blue-600 flex-shrink-0">
      <span class="text-sm text-slate-800 leading-snug">
          Texto descriptivo amplio donde pulsar activa la casilla sin esfuerzo táctil.
      </span>
  </label>
  ```

---

## 8. Escala de Pantallas: Desde 320px hasta Monitores 4K

| Dispositivo / Pantalla | Ancho Típico | Consideraciones Técnicas |
| :--- | :--- | :--- |
| **Móviles Ultra-Pequeños** | `320px - 360px` (iPhone SE 1ª gen, Galaxy Fold cerrado) | Prohibido usar anchos fijos mayores a `100%`. Utilizar `padding: 12px` en lugar de `24px`. Toda cuadrícula debe ser de 1 sola columna. |
| **Móviles Estándar** | `375px - 430px` (iPhone 12/13/14/15/16 Pro Max) | Espaciado cómodo, fuentes entre 14px y 16px, botones de ancho completo (`w-full`). |
| **iPad / Tabletas** | `768px - 1024px` (iPad Mini, iPad Air, iPad Pro) | Transición fluida a 2 columnas. Mantener formularios en tarjetas centradas de ancho máximo (`max-w-2xl` a `max-w-3xl`). |
| **Portátiles y Escritorio** | `1025px - 1920px` (MacBook, monitores Full HD) | Contenedor principal con `max-w-4xl` centrado mediante `mx-auto` para evitar campos exageradamente anchos. |
| **Pantallas 4K / Ultrawide** | `> 2560px` | Nunca permitir que el formulario ocupe el 100% del ancho del monitor. Fijar un tope (`max-width: 900px`) para evitar fatiga ocular. |

---

## 9. Desplazamiento Suave en iOS Antiguo

Para contenedores internos o modales con scroll en iOS 12:
```css
.scroll-container {
    overflow-y: auto;
    -webkit-overflow-scrolling: touch; /* Habilita el desplazamiento por inercia nativo en iOS */
}
```

---

---

## 10. Protocolo de Seguridad Anti-Bot y Ciclo de Vida del Botón de Envío

Para evitar que bots maliciosos envíen formularios masivos y prevenir envíos duplicados por parte de los usuarios, se establece la siguiente regla obligatoria de arquitectura:

### 1. Doble Capa Anti-Bot:
* **Capa Visible (Captcha Alfanumérico):** Código visual aleatorio de 4 caracteres alfanuméricos no ambiguos (excluyendo caracteres confusos como `0`/`O` y `1`/`I`), con botón de refresco y campo de entrada en mayúsculas forzadas.
* **Capa Oculta (Honeypot):** Campo invisible para humanos que si un bot rellena automáticamente, anula el envío silenciosamente.

### 2. Regla de Oro del Bloqueo y Reseteo del Botón Final:
* **Al pulsar enviar:** El botón se bloquea de inmediato (`disabled = true;`), muestra un spinner y añade clases visuales de desactivación (`opacity-50 cursor-not-allowed pointer-events-none`).
* **Tras envío exitoso:** El botón **NO se reactiva inmediatamente**. Permanece bloqueado (`Inscripción Enviada`) para impedir cualquier reenvío accidental.
* **Desbloqueo condicionado a la limpieza total:** El botón final **SÓLO se desbloquea cuando el formulario se limpia y reinicia por completo** (al cerrar el modal de éxito, al pulsar "Realizar otra inscripción" o mediante limpieza manual). En ese instante exacto se vacían campos, se restaura a 1 participante, se regenera el Captcha y se vuelve a habilitar el botón.

---

## 11. Checklist de Verificación Responsive, iOS y Seguridad

Antes de publicar cualquier formulario:

- [ ] ¿Todos los inputs, selects y textareas tienen `font-size: 16px` en móvil para evitar zoom en Safari?
- [ ] ¿Se ha comprobado que no hay elementos que causen scroll horizontal en pantallas de `320px` de ancho?
- [ ] ¿Los botones y casillas tienen un área táctil mínima de 44px?
- [ ] ¿Se utiliza CSS Grid o márgenes seguros en lugar de `flex gap` si se soporta iOS < 14.5?
- [ ] ¿El modal de confirmación o pie de página no queda oculto bajo la barra dinámica de Safari (`min-height: 100dvh` / fallback `100vh`)?
- [ ] ¿Se han incluido los atributos `inputmode`, `autocapitalize` y `autocomplete` en los campos de tutor y contacto?
- [ ] ¿El contenedor principal tiene un `max-w-*` centrado para verse elegante en iPad y monitores grandes?
- [ ] ¿Está presente la verificación de seguridad anti-bot (Captcha visible + Honeypot invisible)?
- [ ] ¿El botón de envío se bloquea durante el proceso y se mantiene bloqueado hasta que el formulario se limpia por completo?

