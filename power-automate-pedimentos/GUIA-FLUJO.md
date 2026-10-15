# Pedimentos Intelideck → OneDrive + Excel de control

Flujo profesional en **Power Automate** para:

1. Detectar correos del dominio **`@intelideck.com`**.
2. Guardar **todos** los PDF adjuntos (nombre indistinto).
3. Crear carpetas de año y mes (`EXTRACCIONES 2026` / `10 OCTUBRE 2026`).
4. Evitar duplicados con un Excel de control (`MessageID` + `NombrePDF`).
5. Registrar tamaño del archivo y manejar errores sin tumbar el flujo.

---

## 1) Excel de control (crear una sola vez)

Crea el archivo:

```text
CONTROL PEDIMENTOS.xlsx
```

Ubicación en OneDrive:

```text
Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/CONTROL/CONTROL PEDIMENTOS.xlsx
```

Convierte el rango en tabla con nombre:

```text
tblPedimentos
```

Columnas (en este orden):

| FechaCorreo | Remitente | MessageID | NombrePDF | TamanoBytes | RutaDestino | FechaGuardado | Estado |
|-------------|-----------|-----------|-----------|-------------|-------------|---------------|--------|

> `Estado` sirve para auditoría (`Guardado` / `Error`).  
> `TamanoBytes` evita confusiones si llega el mismo nombre con otro contenido.

---

## 2) Crear el flujo

Nombre sugerido: `Pedimentos Intelideck - OneDrive + Control Excel`

### Paso 1 — Desencadenador

**Office 365 Outlook** → *Cuando llega un nuevo correo electrónico (V3)*

| Campo | Valor |
|-------|--------|
| Carpeta | Bandeja de entrada |
| Incluir archivos adjuntos | **Sí** |
| Solo con archivos adjuntos | **Sí** |
| De | *(vacío)* |
| Asunto contiene | *(vacío)* |

### Paso 2 — Condición dominio Intelideck

Expresión (recomendada):

```powerfx
endsWith(
  toLower(triggerBody()?['from']?['emailAddress']?['address']),
  '@intelideck.com'
)
```

Si en tu tenant `from` viene como texto plano:

```powerfx
contains(toLower(string(triggerBody()?['from'])), '@intelideck.com')
```

- **Sí** → continúa  
- **No** → termina (sin acciones)

### Paso 3 — Variable `Anio` (String)

```powerfx
formatDateTime(triggerBody()?['receivedDateTime'], 'yyyy')
```

### Paso 4 — Compose `MesEsp`

```powerfx
switch(
  int(formatDateTime(triggerBody()?['receivedDateTime'], 'MM')),
  1, 'ENERO',
  2, 'FEBRERO',
  3, 'MARZO',
  4, 'ABRIL',
  5, 'MAYO',
  6, 'JUNIO',
  7, 'JULIO',
  8, 'AGOSTO',
  9, 'SEPTIEMBRE',
  10, 'OCTUBRE',
  11, 'NOVIEMBRE',
  12, 'DICIEMBRE',
  'MES'
)
```

### Paso 5 — Variable `CarpetaMes` (String)

```powerfx
concat(
  formatDateTime(triggerBody()?['receivedDateTime'], 'MM'),
  ' ',
  outputs('MesEsp'),
  ' ',
  formatDateTime(triggerBody()?['receivedDateTime'], 'yyyy')
)
```

Ejemplo: `10 OCTUBRE 2026`

### Paso 6 — Variable `RutaAnio` (String)

```powerfx
concat(
  '/Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/EXTRACCIONES ',
  variables('Anio')
)
```

### Paso 7 — Variable `RutaMes` (String)

```powerfx
concat(
  variables('RutaAnio'),
  '/',
  variables('CarpetaMes')
)
```

### Paso 8 — Crear carpeta del año

**OneDrive for Business** → *Crear nueva carpeta*

| Campo | Valor |
|-------|--------|
| Ruta | `/Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES` |
| Nombre | `concat('EXTRACCIONES ', variables('Anio'))` |

**Configurar ejecución posterior:** Correcto + Error + Omitido  
(así no falla si la carpeta ya existe)

### Paso 9 — Crear carpeta del mes

| Campo | Valor |
|-------|--------|
| Ruta | `variables('RutaAnio')` |
| Nombre | `variables('CarpetaMes')` |

**Configurar ejecución posterior:** Correcto + Error + Omitido

### Paso 10 — Scope `ProcesarAdjuntos` (manejo de errores)

Envuelve los pasos 11–17 en un **Ámbito (Scope)** llamado `ProcesarAdjuntos`.

Luego añade otro Scope `SiFalloProcesar` con **ejecutar solo si `ProcesarAdjuntos` falla**, y dentro:

**Excel** → *Agregar una fila a una tabla* (registro de error):

| Columna | Valor |
|---------|--------|
| FechaCorreo | `triggerBody()?['receivedDateTime']` |
| Remitente | `triggerBody()?['from']?['emailAddress']?['address']` |
| MessageID | `triggerBody()?['id']` |
| NombrePDF | `(error global)` |
| TamanoBytes | `0` |
| RutaDestino | `variables('RutaMes')` |
| FechaGuardado | `utcNow()` |
| Estado | `Error` |

Opcional: acción *Enviar un correo* a ti mismo avisando el fallo.

### Paso 11 — Aplicar a cada uno

Origen: `Attachments` del correo.

Dentro del bucle:

### Paso 12 — Condición: ¿es PDF?

```powerfx
endsWith(toLower(items('Apply_to_each')?['name']), '.pdf')
```

- **No** → siguiente adjunto  
- **Sí** → continúa

### Paso 13 — Consultar Excel (¿ya existe?)

**Excel Online (Business)** → *Listar filas presentes en una tabla*

| Campo | Valor |
|-------|--------|
| Ubicación | OneDrive / Documentos… |
| Biblioteca | OneDrive |
| Archivo | `CONTROL PEDIMENTOS.xlsx` |
| Tabla | `tblPedimentos` |

#### Filtro OData corregido

En **Consulta de filtro** (Filter Query) pega **una sola línea** (sin saltos ni `and` en líneas separadas mal escapadas):

```text
MessageID eq '@{triggerBody()?['id']}' and NombrePDF eq '@{replace(items('Apply_to_each')?['name'], '''', '''''')}'
```

Notas OData importantes:

1. Usa `eq` (no `=`).
2. `and` en minúsculas, en la misma línea.
3. Si el nombre del PDF tuviera comilla simple `'`, el `replace(... '''', '''''')` la escapa.
4. Si el conector se queja del filtro dinámico, deja el filtro vacío y usa **Filtrar matriz** (paso 13b).

#### Paso 13b — Alternativa sin OData (más estable)

1. *Listar filas presentes en una tabla* **sin** Filter Query.  
2. Acción **Filtrar matriz**:

**Desde:**

```powerfx
body('List_rows_present_in_a_table')?['value']
```

**Condición avanzada:**

```powerfx
@and(
  equals(item()?['MessageID'], triggerBody()?['id']),
  equals(item()?['NombrePDF'], items('Apply_to_each')?['name'])
)
```

3. La condición de duplicado usa `body('Filter_array')` en lugar de `body('List_rows_present_in_a_table')`.

> Con muchos registros, OData es mejor. Al inicio, Filtrar matriz es más fácil de depurar.

### Paso 14 — Condición: ¿ya procesado?

```powerfx
greater(length(body('List_rows_present_in_a_table')?['value']), 0)
```

Si usaste Filtrar matriz:

```powerfx
greater(length(body('Filter_array')), 0)
```

- **Sí (TRUE)** → no hacer nada (duplicado; evita sobrescribir).  
- **No (FALSE)** → guardar PDF + registrar.

### Paso 15 — Obtener contenido del adjunto

**Outlook** → *Obtener archivo adjunto (V2)* / *Get attachment content*

| Campo | Valor |
|-------|--------|
| Id del mensaje | `triggerBody()?['id']` |
| Id del adjunto | `items('Apply_to_each')?['id']` |

### Paso 16 — Crear archivo en OneDrive

**OneDrive for Business** → *Crear archivo*

| Campo | Valor |
|-------|--------|
| Ruta de carpeta | `variables('RutaMes')` |
| Nombre | `items('Apply_to_each')?['name']` |
| Contenido | contenido del paso 15 |

**Configurar ejecución posterior:** Correcto + Error  
Si falla (conflicto de nombre raro), el Scope de error lo captura.

### Paso 17 — Registrar en Excel

**Excel Online (Business)** → *Agregar una fila a una tabla*

Tabla: `tblPedimentos`

| Columna | Valor |
|---------|--------|
| FechaCorreo | `triggerBody()?['receivedDateTime']` |
| Remitente | `triggerBody()?['from']?['emailAddress']?['address']` |
| MessageID | `triggerBody()?['id']` |
| NombrePDF | `items('Apply_to_each')?['name']` |
| TamanoBytes | `items('Apply_to_each')?['size']` |
| RutaDestino | `concat('EXTRACCIONES ', variables('Anio'), '/', variables('CarpetaMes'))` |
| FechaGuardado | `utcNow()` |
| Estado | `Guardado` |

---

## 3) Diagrama

```text
[Correo nuevo con adjuntos]
        |
        v
[¿@intelideck.com?] --No--> fin
        |
       Sí
        v
[Anio + CarpetaMes]
        |
        v
[Crear EXTRACCIONES AÑO] (ignorar si existe)
        |
        v
[Crear carpeta MES] (ignorar si existe)
        |
        v
[Scope ProcesarAdjuntos]
        |
        +--> Para cada adjunto
                |
                +--> ¿.pdf? --No--> siguiente
                |
               Sí
                v
         [¿MessageID+NombrePDF en Excel?]
                |
          Sí --> ignorar
          No --> obtener contenido
                    --> crear archivo OneDrive
                    --> fila Excel (TamanoBytes + Guardado)
        |
        v
[Si falla Scope] --> fila Excel Estado=Error (+ aviso opcional)
```

---

## 4) Resultado esperado

Correo de `ivonne.casillas@intelideck.com` con:

```text
1422CEX0126026494.pdf
1422CEX0126026495.pdf
```

Quedan en:

```text
.../EXTRACCIONES 2026/10 OCTUBRE 2026/
```

PC sincronizado:

```text
C:\Users\ISMAEL RAMIREZ\OneDrive - andujar\Documentos\01 IMPORTACIONES ANDUJAR\IMPORTACIONES\EXTRACCIONES 2026\10 OCTUBRE 2026\
```

Y en `CONTROL PEDIMENTOS.xlsx` una fila por PDF guardado (o `Error` si falló).

Si el mismo correo se reenvía o el flujo corre dos veces: **no se vuelve a guardar** porque `MessageID` + `NombrePDF` ya existen.

---

## 5) Manejo de errores (resumen)

| Situación | Qué hace el flujo |
|-----------|-------------------|
| Carpeta año/mes ya existe | Continúa (run after Error/Omitido) |
| PDF ya en Excel (mismo MessageID + nombre) | Ignora; no sobrescribe |
| Falla al crear archivo / Excel | Scope captura → fila `Estado=Error` |
| Adjunto no PDF | Se salta |

Opcional extra (doble red): además del Excel, puedes listar archivos en la carpeta del mes y omitir si el mismo **nombre + tamaño** ya está en OneDrive. El Excel con MessageID suele bastar.

---

## 6) Checklist

- [ ] Excel `CONTROL PEDIMENTOS.xlsx` creado con tabla `tblPedimentos` y columna `TamanoBytes`
- [ ] Condición `@intelideck.com`
- [ ] Carpetas año/mes con run-after en error
- [ ] Filtro OData en una sola línea (o Filtrar matriz)
- [ ] Solo `.pdf`
- [ ] Registro Excel tras guardar (o Error en Scope)
- [ ] Prueba con correo real de Intelideck
- [ ] Flujo activado
