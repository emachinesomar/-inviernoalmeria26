# VII Campus de Invierno «Sembrando Futuro» (Edición Navidad)
**Organiza: Instituto Bahá'í de Capacitación para el Desarrollo Comunitario · Garrucha (Almería)**

Formulario web moderno, completamente adaptativo y sistema backend basado en **Google Apps Script** y **Google Sheets** para la gestión integral de inscripciones al VII Campus de Invierno «Sembrando Futuro».

---

## 🌟 Características Principales

### 1. Frontend Responsivo y Adaptativo ([index.html](index.html))
- **Identidad institucional y bienvenida pedagógica**:
  - Distintivo visual del **Instituto Bahá'í de Capacitación para el Desarrollo Comunitario (Garrucha)** con su logotipo oficial.
  - Sección introductoria completa sobre los objetivos de convivencia intercultural en Garrucha y la metodología pedagógica del **Instituto Ruhí**.
  - Desglose detallado del programa de contenidos para **Niños** (Mindfulness, valores, arte, talleres) y **Adolescentes** (Mindfulness, deporte, reflexión moral sobre relatos internacionales y servicio comunitario).
- **Diseño visual moderno**: Tipografía *Plus Jakarta Sans* y *Outfit*, Tailwind CSS, tarjetas glassmorphism y micro-interacciones.
- **Compatibilidad móvil rigurosa**: Cumple con el estándar de compatibilidad para **iOS y iPadOS (versión 12 hasta la más reciente)**, evitando el zoom automático en inputs de Safari (`font-size: 16px`), `viewport-fit=cover` y safe areas.
- **Gestión dinámica de participantes**:
  - Cálculo automático de edad según la fecha de nacimiento.
  - Asignación inteligente a grupos sin edades entre paréntesis:
    - *Etapa Infantil (3 a 5 años)*: Exige adulto acompañante durante las 5 horas completas del campus.
    - *1º a 3º de Primaria*: Asistencia autónoma.
    - *4º a 5º de Primaria*: Asistencia autónoma.
    - *6º de Primaria a 3º de la ESO*: Asistencia autónoma con dinámicas adaptadas.
    - *A partir de 4º de la ESO*: Grupo formativo y voluntariado.
- **Logística, talleres y autorizaciones**:
  - Campo para observaciones médicas y alergias alimentarias.
  - Propuestas de talleres familiares o de talentos locales.
  - Autorización explícita de participación con selector afirmativo/negativo (sin términos médicos innecesarios).
- **Área de voluntariado**:
  - Selector individual o completo de los días del evento (23, 24, 28, 29 y 30 de Diciembre).
- **Cálculo de aportaciones en tiempo real**:
  - Asistencia al campus 100% gratuita gracias a monitores voluntarios.
  - Coste real de material: 5€ por participante registrado (donación sugerida).
  - Selector de donación voluntaria extra (0€, 5€, 10€, 20€ u otro importe).
  - Opciones de abono: **Bizum** (con copia rápida del teléfono de contacto) o en efectivo el primer día.
- **Seguridad y Control Anti-Bot**:
  - Campo trampa (*Honeypot*) invisible para scripts automatizados.
  - Captcha visual dinámico de 4 caracteres alfanuméricos no ambiguos.
  - **Bloqueo del botón de envío**: Al enviar, el botón se deshabilita con spinner, y tras el éxito se mantiene bloqueado como "Inscripción Enviada" hasta que se limpie o reinicie el formulario.

---

### 2. Backend en Google Apps Script ([codigo.gs.txt](codigo.gs.txt))
- **Registro consolidado (1 fila por inscripción)**:
  - Evita duplicidad de importes económicos o información de contacto en Google Sheets.
  - Todos los participantes inscritos por un mismo tutor se agrupan ordenadamente en una única celda.
- **Confirmación automática por correo electrónico**:
  - **Dual delivery (HTML + Texto plano)**: Plantilla HTML moderna con tarjetas e información estructurada, acompañada de un cuerpo de texto plano limpio.
  - **100% libre de caracteres corruptos**: Sin emojis multi-byte que puedan generar errores de codificación (rombos con signos de interrogación) en clientes de correo antiguos o notificaciones móviles.
- **Funciones de prueba y diagnóstico integradas**:
  - `autorizarPermisos()`: Concesión limpia de permisos OAuth en Google Workspace.
  - `testDoPost()`: Simulación completa de registro sin necesidad de abrir el navegador.
  - `testVerificarEncabezados()`: Auditoría de columnas en la hoja de cálculo.

---

## 🛠️ Tecnologías Utilizadas

- **HTML5 & CSS3** (Vanilla + Tailwind CSS)
- **JavaScript Moderno (ES6+)** (Validaciones asíncronas, manipulación reactiva del DOM)
- **Google Apps Script** (API Web App, LockService, SpreadsheetApp, GmailApp)
- **Google Sheets** (Almacenamiento persistente de datos)

---

## 🚀 Despliegue y Configuración

1. **Configurar Google Sheet y Apps Script**:
   - Crea una hoja de cálculo en Google Sheets.
   - Abre **Extensiones > Apps Script**.
   - Pega el código de [`codigo.gs.txt`](codigo.gs.txt) en `Código.gs`.
   - Modifica la constante `SPREADSHEET_ID` con el ID de tu hoja.
   - Ejecuta primero `autorizarPermisos()` y acepta los consentimientos.
   - Realiza el despliegue como Aplicación Web (**Implementar > Nueva implementación**, acceso: *Cualquier usuario*).
2. **Conectar el formulario web**:
   - En [`index.html`](index.html), actualiza la constante `SCRIPT_URL` con la URL de tu aplicación web (`/exec`).
   - Sirve el archivo `index.html` mediante GitHub Pages, Vercel, Netlify o cualquier servidor web estático.
