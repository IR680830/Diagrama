# Guardar PDF de pedimentos (Outlook → OneDrive)

Flujo automatizado en **Power Automate** (nube) para:

1. Detectar correos del dominio **`@intelideck.com`**.
2. Guardar **todos** los PDF adjuntos (el nombre es indistinto).
3. Crear (si no existe) la carpeta del mes, por ejemplo `10 OCTUBRE 2026`.
4. Guardarlos en OneDrive, en la ruta de Extracciones.

Ruta base (equivalente a tu carpeta local sincronizada):

```text
Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/EXTRACCIONES 2026
```

Ejemplo carpeta del mes:

```text
.../EXTRACCIONES 2026/10 OCTUBRE 2026
```

> Nota: en OneDrive/SharePoint no uses la ruta `C:\Users\...`. Usa la ruta relativa dentro de OneDrive (arriba). El cliente de OneDrive la sincroniza a tu PC.

---

## Antes de empezar

Necesitas:

- Cuenta Microsoft 365 con Outlook y OneDrive (Andujar).
- Acceso a [https://make.powerautomate.com](https://make.powerautomate.com)

Filtros ya definidos:

| Dato | Valor |
|------|--------|
| Remitente | Dominio **`@intelideck.com`** |
| Nombre PDF | Indistinto — se guardan **todos** los `.pdf` |
| Asunto | Sin filtro |
| Año de extracciones | `2026` |
| Carpeta base OneDrive | `Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/EXTRACCIONES 2026` |

---

## Crear el flujo (paso a paso)

### 1) Nuevo flujo automatizado

1. Entra a **Power Automate** → **Crear** → **Flujo de nube automatizado**.
2. Nombre sugerido: `Pedimentos - Guardar PDF en Extracciones`.
3. Descarta el asistente y arma el flujo a mano (más claro).

### 2) Desencadenador

**Conector:** Office 365 Outlook  
**Acción:** *Cuando llega un nuevo correo electrónico (V3)*

Configuración recomendada:

| Campo | Valor |
|-------|--------|
| Carpeta | `Bandeja de entrada` (o la carpeta donde llegan) |
| Incluir archivos adjuntos | **Sí** |
| Solo con archivos adjuntos | **Sí** |
| De | *(vacío)* — el filtro de dominio va en la condición de abajo |
| Asunto contiene | *(vacío)* |

Justo después del desencadenador, añade una **Condición** (solo dominio `intelideck.com`):

```powerfx
contains(toLower(string(triggerOutputs()?['body/from'])), '@intelideck.com')
```

Versión más estricta (toma lo que va después de `@`):

```powerfx
endsWith(
  toLower(
    trim(
      last(
        split(
          string(triggerOutputs()?['body/from']),
          '@'
        )
      )
    )
  ),
  'intelideck.com'
)
```

- **Si sí** → continúa el flujo (crear carpeta + guardar PDF).
- **Si no** → no hagas nada (termina).

### 3) Inicializar variables

Añade **Inicializar variable** (una por cada una):

#### A) `AnioExtracciones` (Cadena)

```text
2026
```

#### B) `RutaBase` (Cadena)

```text
Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/EXTRACCIONES 2026
```

#### C) `NombreCarpetaMes` (Cadena) — expresión

En el campo Valor → **Expresión**:

```powerfx
concat(
  formatDateTime(utcNow(), 'MM'),
  ' ',
  toUpper(
    replace(
      replace(
        replace(
          replace(
            replace(
              replace(
                replace(
                  replace(
                    replace(
                      replace(
                        replace(
                          replace(
                            formatDateTime(utcNow(), 'MMMM', 'es-MX'),
                            'á', 'A'
                          ),
                          'é', 'E'
                        ),
                        'í', 'I'
                      ),
                      'ó', 'O'
                    ),
                    'ú', 'U'
                  ),
                  'ñ', 'N'
                ),
                'january', 'ENERO'
              ),
              'february', 'FEBRERO'
            ),
            'march', 'MARZO'
          ),
          'april', 'ABRIL'
        ),
        'may', 'MAYO'
      ),
      'june', 'JUNIO'
    )
  ),
  ' ',
  formatDateTime(utcNow(), 'yyyy')
)
```

Si tu entorno ya devuelve el mes en español con `es-MX`, puedes usar esta versión más corta:

```powerfx
concat(
  formatDateTime(utcNow(), 'MM'),
  ' ',
  toUpper(formatDateTime(utcNow(), 'MMMM', 'es-MX')),
  ' ',
  formatDateTime(utcNow(), 'yyyy')
)
```

Resultado esperado hoy: `10 OCTUBRE 2026`.

> Si el mes sale en inglés (`OCTOBER`), usa la tabla de conversión de abajo en una **Compose** o cambia a la expresión larga / switch por número de mes.

#### Alternativa estable (recomendada): mes por número

**Compose** llamado `MesEsp` con expresión:

```powerfx
switch(
  int(formatDateTime(utcNow(), 'MM')),
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

Luego `NombreCarpetaMes`:

```powerfx
concat(
  formatDateTime(utcNow(), 'MM'),
  ' ',
  outputs('MesEsp'),
  ' ',
  formatDateTime(utcNow(), 'yyyy')
)
```

### 4) Crear carpeta del mes (si no existe)

**Conector:** OneDrive for Business  
**Acción:** *Crear nueva carpeta*

| Campo | Valor |
|-------|--------|
| Ruta de la carpeta | `@{variables('RutaBase')}` |
| Nombre | `@{variables('NombreCarpetaMes')}` |

Configura **Configurar ejecución posterior** en esta acción:

- También ejecutar si da error (porque si la carpeta ya existe, OneDrive a veces falla).

Opcional más robusto:

1. **Obtener metadatos de archivo o carpeta** con ruta  
   `@{variables('RutaBase')}/@{variables('NombreCarpetaMes')}`
2. Si falla → **Crear nueva carpeta**.
3. Si existe → continuar.

### 5) Recorrer adjuntos

Tras el desencadenador, usa el array de adjuntos:

**Aplicar a cada uno** → `Adjuntos` del correo.

Dentro del bucle:

#### Condición: cualquier PDF (nombre indistinto)

Da igual que se llame `1422CEX0126026494.pdf` u otro nombre. Solo filtra extensión:

```powerfx
endsWith(toLower(items('Apply_to_each')?['name']), '.pdf')
```

#### Si sí → Crear archivo

**Conector:** OneDrive for Business  
**Acción:** *Crear archivo*

| Campo | Valor |
|-------|--------|
| Ruta de la carpeta | `@{variables('RutaBase')}/@{variables('NombreCarpetaMes')}` |
| Nombre de archivo | `@{items('Apply_to_each')?['name']}` |
| Contenido del archivo | `@{items('Apply_to_each')?['contentBytes']}` |

Si el desencadenador V3 no trae `contentBytes` en el bucle, inserta antes:

**Obtener archivo adjunto (V2)**

- Id del mensaje: `Id` del correo
- Id del archivo adjunto: `Id` del adjunto actual

Y usa el contenido de esa acción en **Crear archivo**.

### 6) (Opcional) Marcar correo como leído / mover

Tras guardar:

- *Marcar como leído*, o
- *Mover correo* a una carpeta `Pedimentos procesados`.

Así evitas reprocesar a mano.

### 7) Guardar y probar

1. **Guardar** el flujo.
2. Reenvía a tu bandeja un correo real de `@intelideck.com` con algún PDF (o espera el siguiente).
3. Revisa en OneDrive:

```text
Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/EXTRACCIONES 2026/10 OCTUBRE 2026
```

4. En el PC debe aparecer por sincronización en:

```text
C:\Users\ISMAEL RAMIREZ\OneDrive - andujar\Documentos\01 IMPORTACIONES ANDUJAR\IMPORTACIONES\EXTRACCIONES 2026\10 OCTUBRE 2026
```

---

## Diagrama del flujo

```text
[Cuando llega correo con adjuntos]
        |
        v
[¿Dominio = @intelideck.com?] --No--> (fin)
        |
       Sí
        v
[Calcular nombre carpeta: MM MES AÑO]
        |
        v
[Crear carpeta en EXTRACCIONES 2026 si no existe]
        |
        v
[Para cada adjunto]
        |
        +--> ¿Es .pdf? (cualquier nombre) --No--> (omitir)
        |
       Sí
        v
[Crear archivo en .../10 OCTUBRE 2026]
        |
        v
[(Opcional) Marcar leído / mover correo]
```

---

## Cambio de año (2027)

Cuando llegue 2027:

1. Crea en OneDrive: `.../EXTRACCIONES 2027`
2. En el flujo, cambia:
   - variable `AnioExtracciones` → `2027`
   - `RutaBase` → `.../EXTRACCIONES 2027`

O haz que `RutaBase` sea dinámica:

```powerfx
concat(
  'Documentos/01 IMPORTACIONES ANDUJAR/IMPORTACIONES/EXTRACCIONES ',
  formatDateTime(utcNow(), 'yyyy')
)
```

Y crea también la carpeta del año al inicio del flujo (misma lógica que la del mes).

---

## SharePoint en lugar de OneDrive

Si la biblioteca está en un sitio SharePoint:

1. Cambia el conector a **SharePoint**.
2. Sitio: el de Andujar.
3. Biblioteca / ruta: la equivalente a `Documentos/.../EXTRACCIONES 2026`.
4. Acciones: *Crear nueva carpeta* y *Crear archivo* de SharePoint.

El resto del flujo es igual.

---

## Power Automate Desktop (solo si debes usar ruta C:\)

Solo necesario si **no** puedes usar el conector de OneDrive en la nube.

1. Flujo nube: *Cuando llega correo* → *Ejecutar flujo creado con Power Automate para escritorio*.
2. En Desktop:
   - Obtener adjuntos / descargar PDF.
   - `Folder.Create` en  
     `%UserProfile%\OneDrive - andujar\Documentos\01 IMPORTACIONES ANDUJAR\IMPORTACIONES\EXTRACCIONES 2026\%MesCarpeta%`
   - Copiar PDF a esa carpeta.

Para Andujar, la versión **solo nube (OneDrive)** suele ser suficiente y más estable.

---

## Checklist de puesta en marcha

- [ ] Condición de dominio `@intelideck.com`
- [ ] Condición de adjunto: cualquier `.pdf` (nombre indistinto)
- [ ] `RutaBase` coincide con OneDrive (sin `C:\Users\...`)
- [ ] Nombre de carpeta = `MM MES AÑO` (ej. `10 OCTUBRE 2026`)
- [ ] Prueba con un correo real de `@intelideck.com` y verificación en PC sincronizado
- [ ] Flujo activado (no en borrador)

---

## Fallos frecuentes

| Problema | Qué revisar |
|----------|-------------|
| No se crea la carpeta | Ruta base mal escrita; permisos OneDrive |
| Carpeta en inglés (`10 OCTOBER 2026`) | Usar el `switch` de mes en español |
| Archivo vacío o error al crear | Usar *Obtener archivo adjunto (V2)* y su contenido |
| No entra el correo al flujo | Que el *From* sea `@intelideck.com`; carpeta de Outlook incorrecta |
| No guarda el PDF | Que el adjunto termine en `.pdf` (el nombre no importa) |
| Duplicados | Activar *Marcar como leído* o mover a carpeta procesados |

---

## Filtros aplicados

| Criterio | Regla |
|----------|--------|
| Remitente | Dominio **`@intelideck.com`** |
| Adjunto | Cualquier archivo que termine en **`.pdf`** |
| Nombre del PDF | Indistinto |
| Asunto | Sin filtro |
