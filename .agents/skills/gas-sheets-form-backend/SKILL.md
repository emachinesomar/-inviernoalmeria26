---
name: gas-sheets-form-backend
description: >-
  Estándar de ingeniería backend para formularios web conectados a Google Apps Script y Google Sheets.
  Cubre prevención de duplicación de datos financieros, gestión de concurrencia con LockService,
  normalización de arrays de participantes, correos transaccionales duales (HTML + texto plano) 100% libres
  de errores de codificación (mojibake/emojis rotos), resolución de permisos OAuth y arnés de pruebas integrado.
---

# Backend de Formularios con Google Apps Script y Google Sheets

Esta habilidad proporciona el estándar de arquitectura y patrones de código para conectar formularios web HTML/JavaScript con **Google Apps Script (GAS)** y **Google Sheets** de forma robusta, segura y escalable.

---

## 1. La Regla de Oro de los Datos Financieros: Fila Única por Inscripción

### El Problema Crítico de los Bucles Multi-Participante:
En formularios donde un tutor inscribe a varios participantes (por ejemplo, 2 o 3 hijos):
- Si el backend ejecuta un bucle `for` e inserta una fila por cada hijo, los campos globales (datos del tutor, teléfono, donaciones voluntarias, total estimado y método de pago) **se duplican en cada fila**.
- Esto **corrompe la contabilidad**: cualquier fórmula `=SUMA(...)` o recuento en Google Sheets multiplicará erróneamente los importes reales recibidos.

### Patrón Obligatorio: 1 Fila por Inscripción / Transacción
Cada envío de formulario debe generar **una única fila consolidada** en la hoja principal (`Inscripciones`):
1. Los participantes se formatean en un único bloque de texto legible dentro de una sola celda:
   ```javascript
   var listaParticipantesFormateada = [];
   for (var i = 0; i < nombres.length; i++) {
     listaParticipantesFormateada.push((i + 1) + '. ' + nombres[i] + ' (Nac: ' + nacimientos[i] + ' - ' + grupos[i] + ')');
   }
   var textoParticipantes = listaParticipantesFormateada.join('\n');
   ```
2. Los importes económicos (`coste_material`, `donacion_voluntaria`, `total_estimado`) se registran **exactamente una vez**.
3. **Pestaña secundaria opcional (`Participantes`):** Si los coordinadores o monitores de grupo necesitan pasar lista individualmente por niño, se puede alimentar una segunda hoja con una fila por participante vinculada por el mismo `Localizador`, pero manteniendo la hoja principal como la única fuente de verdad contable y de contacto.

---

## 2. Control de Concurrencia con `LockService`

Google Apps Script ejecuta peticiones web de forma concurrente. Si dos usuarios envían el formulario en el mismo segundo, `sheet.appendRow()` puede colisionar o generar inconsistencias.

### Implementación Obligatoria:
```javascript
function doPost(e) {
  var lock = LockService.getScriptLock();
  // Esperar hasta 30 segundos a que finalice cualquier escritura en curso
  lock.tryLock(30000);

  try {
    // Lógica de validación, inserción y correo...
    
    return ContentService.createTextOutput(JSON.stringify({
      success: true,
      message: 'Inscripción procesada correctamente'
    })).setMimeType(ContentService.MimeType.JSON);

  } catch (error) {
    Logger.log('Error en doPost: ' + error.toString());
    return ContentService.createTextOutput(JSON.stringify({
      success: false,
      message: 'Error en el servidor: ' + error.toString()
    })).setMimeType(ContentService.MimeType.JSON);

  } finally {
    // Liberar siempre el bloqueo, incluso en caso de error
    lock.releaseLock();
  }
}
```

---

## 3. Normalización Universal de Parámetros (`e.parameter` vs `e.parameters`)

Cuando un formulario envía campos múltiples (como `participante_nombre[]` o casillas múltiples de días), Google Apps Script los expone de manera distinta según el método de empaquetado:
- `e.parameters['nombre[]']`: Array de valores.
- `e.parameter['nombre']`: Cadena simple si sólo hay un valor.

### Helper de Normalización Seguro:
```javascript
function normalizeArray(name) {
  if (e.parameters && e.parameters[name] && e.parameters[name].length) {
    return e.parameters[name];
  }
  var value = e.parameter ? e.parameter[name] : null;
  if (!value) return [];
  return Array.isArray(value) ? value : [value];
}

// Uso:
var nombres = normalizeArray('participante_nombre[]');
var nacimientos = normalizeArray('participante_nacimiento[]');
var diasVoluntario = normalizeArray('voluntario_dias[]');
```

---

## 4. Prevención de Mojibake y Correo Transaccional Limpio (`GmailApp`)

### El Peligro de los Emojis de 4 Bytes:
En Google Apps Script y servidores de correo:
- El uso de emojis crudos (`📅`, `🎉`, `👥`, `💳`, `🤝`, `✅`) dentro de cadenas de texto suele decodificarse como secuencias inválidas (`` o rombos con signo de interrogación) en clientes de correo antiguos, en webmail sin UTF-8 estricto o en las previsualizaciones de notificaciones móviles (pantalla de bloqueo de iOS/Android).

### Reglas Obligatorias para Correos Transaccionales:
1. **Cero emojis crudos:** Usar siempre tipografía limpia, viñetas estándar (`•`, `-`), badges CSS o entidades HTML seguras (`&bull;`, `&mdash;`, `&euro;`).
2. **Entrega Dual (Dual Delivery):** Pasar siempre una versión estructurada en texto plano como tercer parámetro de `GmailApp.sendEmail` y el HTML en las opciones:
   ```javascript
   GmailApp.sendEmail(destinatarioEmail, asunto, cuerpoTextoPlano, {
     htmlBody: cuerpoHTML,
     bcc: 'coordinacion@asociacion.org'
   });
   ```
3. **Encabezado UTF-8 en HTML:** El cuerpo HTML debe declarar siempre `<meta charset="UTF-8">` y `<meta name="viewport" content="width=device-width, initial-scale=1.0">`.

---

## 5. Trampa de Autorización OAuth y Solución Rápida

### El Problema de la Supresión de Consentimiento:
Si un script intenta abrir una hoja externa con `SpreadsheetApp.openById(...)` dentro de `doPost` envuelto en un bloque `try/catch`, Google **no puede mostrar la ventana emergente de consentimiento** al usuario externo. El script simplemente fallará con:
`Exception: No tienes permiso para llamar a SpreadsheetApp.openById.`

### Solución en 2 Pasos:
1. **Fallback a Hoja Activa:**
   ```javascript
   var ss;
   try {
     ss = SpreadsheetApp.openById(SPREADSHEET_ID);
   } catch (err) {
     ss = SpreadsheetApp.getActiveSpreadsheet();
     if (!ss) throw err;
   }
   ```
2. **Función de Autorización Directa:** Incluir siempre una función sin `try/catch` para que el desarrollador o administrador la ejecute una única vez desde el editor y apruebe los permisos:
   ```javascript
   function autorizarPermisos() {
     var ss = SpreadsheetApp.openById(SPREADSHEET_ID);
     Logger.log('[OK] Permiso concedido a: ' + ss.getName());
     var cuota = GmailApp.getRemainingDailyQuota();
     Logger.log('[OK] Permiso concedido a Gmail. Cuota diaria: ' + cuota);
   }
   ```

---

## 6. Arnés de Pruebas Integrado (`testDoPost`)

No requieras que el usuario abra el navegador y rellene 20 campos para probar cambios en el backend. Incorpora siempre un arnés de prueba con datos simulados (*mock event*):

```javascript
function testDoPost() {
  Logger.log('[INICIO] Ejecutando testDoPost con hoja: ' + SPREADSHEET_ID);

  var mockEvent = {
    parameter: {
      tutor_nombre: 'María García López (Tutor de Prueba)',
      tutor_dni: '12345678Z',
      tutor_telefono: '612345678',
      tutor_email: Session.getActiveUser().getEmail() || 'omarmonasterio19@gmail.com', 
      coste_material: '10€',
      donacion_voluntaria: '10€',
      total_estimado: '20€',
      metodo_pago: 'Bizum',
      captcha_verificado: 'Sí'
    },
    parameters: {
      'participante_nombre[]': ['Lucía Martínez', 'Mateo Martínez'],
      'participante_nacimiento[]': ['2021-04-12', '2016-09-25'],
      'participante_grupo[]': ['Etapa Infantil', 'Primaria']
    }
  };

  var res = doPost(mockEvent);
  Logger.log('[RESULTADO TEST]: ' + res.getContent());
}
```

---

## 7. Ciclo de Despliegue de la Aplicación Web

Cada vez que se modifica el archivo `codigo.gs` en Google Apps Script:
1. **Guardar** (`Ctrl + S` / `Cmd + S`).
2. Ir a **Implementar > Administrar implementaciones**.
3. Seleccionar la implementación activa y hacer clic en el lápiz (**Editar**).
4. En el desplegable *Versión*, seleccionar obligatoriamente **Nueva versión** (si no se selecciona nueva versión, los clientes web seguirán ejecutando la versión compilada anterior en los servidores de Google).
5. Configurar los permisos:
   - *Ejecutar como:* **Yo (tu cuenta de correo)**.
   - *Quién tiene acceso:* **Cualquier usuario (incluso anónimos)**.
6. Copiar la URL final con terminación `/exec` y actualizar la constante `SCRIPT_URL` en el frontend.

---

## 8. Checklist de Validación para Nuevos Backends

- [ ] ¿Los datos de múltiples participantes se guardan consolidados en 1 fila por registro?
- [ ] ¿Las fórmulas de suma y contabilidad no duplican importes por número de niños?
- [ ] ¿Está presente el bloqueo de concurrencia `LockService` con su bloque `finally`?
- [ ] ¿El correo de confirmación usa entrega dual (HTML + texto plano) sin emojis crudos que puedan generar mojibake?
- [ ] ¿Existe una función `autorizarPermisos()` para la concesión rápida de OAuth?
- [ ] ¿Existe la función `testDoPost()` para comprobar la inserción sin depender del navegador?
- [ ] ¿Se ha publicado una **Nueva versión** en Administrar implementaciones tras el último cambio de código?
